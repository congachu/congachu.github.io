---
title: "[ComputerArch] RISC-V 점프 명령어 정리"
date: 2025-10-01 17:00:00 +0900
categories: [Computer Architecture, RISC-V]
tags: [RISC-V, Assembly, Computer Architecture]
description: "RISC-V 점프 명령어 (jal, jalr) 정리"
toc: true
---

# RISC-V 점프 명령어

조건문(branch)와 달리, **점프(jump)** 명령어는 무조건적으로 프로그램 실행 흐름을 바꾼다.  
특히 `jal`과 `jalr`은 함수 호출과 리턴을 구현하는 데 핵심적인 명령어다.

---

## jal (Jump and Link)

- 형식: `jal rd, offset`
- 동작:
  - `rd ← PC + 4` (리턴 주소 저장)
  - `PC ← PC + offset` (점프 대상 주소로 이동)
- 보통 `rd = ra(x1)`로 사용한다.

### UJ-format
`jal`은 **UJ-format**을 사용한다.  
```
 imm[20] | imm[10:1] | imm[11] | imm[19:12] | rd | opcode
   1bit      10bit       1bit       8bit     5bit   7bit
```
- opcode = `1101111` (0x6F)
- offset = sign-extend(imm[20:1] << 1)
- 범위: 약 ±1MB

### 예시
```asm
jal x1, func   # func으로 점프, 복귀 주소는 x1(ra)에 저장
```

---

## jalr (Jump and Link Register)

- 형식: `jalr rd, offset(rs1)`
- 동작:
  - `rd ← PC + 4`
  - `PC ← R[rs1] + offset`
- 보통 함수 리턴에는 `jalr x0, 0(ra)`를 사용한다.

### I-format
`jalr`은 **I-format**을 사용한다.  
```
 imm[11:0] | rs1 | funct3 | rd | opcode
   12bit    5bit   3bit   5bit   7bit
```
- opcode = `1100111` (0x67)
- funct3 = `000`
- offset 범위: ±2048

### 예시
```asm
jalr x0, 0(x1)   # x1(ra)에 저장된 주소로 점프 (리턴)
```

---

## jal과 jalr의 활용

1. **함수 호출**
```asm
jal ra, foo   # foo 함수로 점프, 복귀 주소 저장
```
2. **함수 리턴**
```asm
jalr x0, 0(ra)   # ra에 저장된 주소로 복귀
```

---

## 요약

- `jal` = PC-relative 점프 + 리턴 주소 저장 (UJ-format, ±1MB)
- `jalr` = 레지스터 기반 점프 + 리턴 주소 저장 (I-format, ±2048)
- 함수 호출/리턴은 `jal`과 `jalr` 조합으로 구현된다.
