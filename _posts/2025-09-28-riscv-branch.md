---
title: "[ComputerArch] RISC-V Branch 명령어 정리"
date: 2025-09-28 01:30:00 +0900
categories: [Computer Architecture, RISC-V]
tags: [RISC-V, Assembly, Computer Architecture]
description: "RISC-V 분기 명령어와 SB-format 정리"
toc: true
---

# RISC-V Branch 명령어

조건문(`if`, `while`, `for`) 같은 고수준 언어의 분기 구조는 RISC-V에서는 **branch 명령어**로 표현된다.  
Branch 명령어는 **PC-relative** 방식으로 동작하며, 현재 명령어의 주소(PC)에 **offset**을 더해서 분기 대상 주소(BTA, Branch Target Address)를 결정한다.

---

## Branch 명령어 종류

| 명령어 | funct3 | 설명 |
|--------|--------|------|
| `beq rs1, rs2, offset`  | 000 | rs1 == rs2 이면 분기 |
| `bne rs1, rs2, offset`  | 001 | rs1 != rs2 이면 분기 |
| `blt rs1, rs2, offset`  | 100 | rs1 < rs2 (signed) |
| `bge rs1, rs2, offset`  | 101 | rs1 >= rs2 (signed) |
| `bltu rs1, rs2, offset` | 110 | rs1 < rs2 (unsigned) |
| `bgeu rs1, rs2, offset` | 111 | rs1 >= rs2 (unsigned) |

---

## SB-format (B-format)

Branch 명령어는 **SB-format**을 사용한다.

```
 imm[12] | imm[10:5] | rs2 | rs1 | funct3 | imm[4:1] | imm[11] | opcode
   1bit       6bit     5bit  5bit    3bit      4bit      1bit     7bit
```

- **opcode** : `1100011` (0x63, branch 공통)  
- **rs1, rs2** : 비교할 두 레지스터  
- **funct3** : 분기 조건 종류 결정  
- **imm** : 12비트 signed offset (항상 2의 배수)  
  - imm[12]   → bit 31  
  - imm[10:5] → bits 30..25  
  - imm[4:1]  → bits 11..8  
  - imm[11]   → bit 7  
- 분기 대상 주소 계산:  
  ```
  BTA = PC + sign_extend(imm[12:1] << 1)
  ```

---

## 예시 1: 단순 분기

```asm
beq x5, x6, Target
```
- 조건: x5 == x6 일 때 Target으로 점프  
- funct3 = 000 (beq), opcode = 1100011  
- offset은 Target - PC 로 계산된다.

---

## 예시 2: 루프 구현

```asm
Loop:
  addi x22, x22, 1     # i++
  blt  x22, x23, Loop  # if (i < x23) → Loop
```
- `blt` 명령어로 i < x23 조건이 참일 때 반복문 실행  
- PC-relative offset을 사용하므로, Loop 라벨과 blt 사이 거리에 따라 imm 값이 달라진다.

---

## 예시 3: BTA 계산

```asm
bne x9, x24, Exit
```
- 현재 bne 주소 = 0x0000000c  
- offset = 0x000c (12)  
- BTA = 0x0000000c + 0x000c = 0x00000018  
- 따라서 PC는 0x18으로 점프

---

## 요약

- Branch 명령어는 조건문 구현에 사용된다.  
- **SB-format**을 사용하며, imm이 분산 저장된다.  
- 최종 분기 대상 주소(BTA)는 항상 `PC + offset`으로 계산된다.  
- 조건 종류는 funct3 값으로 구분된다.

```
조건문 (C 언어) → Branch 명령어 (RISC-V) → SB-format 기계어
```
