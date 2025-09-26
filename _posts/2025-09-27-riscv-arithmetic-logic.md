---
title: "[ComputerArch] RISC-V 연산 명령어 정리"
date: 2025-09-27 04:30:00 +0900
categories: [Computer Architecture, RISC-V]
tags: [RISC-V, Assembly, Computer Architecture]
description: "RISC-V 연산 명령어"
toc: true
---

# RISC-V 연산 명령어 정리

RISC-V의 기본 연산(Arithmetic / Logic) 명령어들을 정리한다.  
각 명령어는 어떤 포맷(R, I)에 속하는지 함께 표기한다.

---

## 1. 덧셈 / 뺄셈

- **add rd, rs1, rs2** (R-format)  
  `rd = rs1 + rs2`

- **addi rd, rs1, imm** (I-format)  
  `rd = rs1 + imm`

- **sub rd, rs1, rs2** (R-format)  
  `rd = rs1 - rs2`

예시:
```asm
addi x5, x0, 10   # x5 = 10
addi x6, x0, 20   # x6 = 20
add  x7, x5, x6   # x7 = 30
sub  x8, x6, x5   # x8 = 10
```

---

## 2. 논리 연산

- **and rd, rs1, rs2** (R-format)  
  `rd = rs1 & rs2`

- **andi rd, rs1, imm** (I-format)  
  `rd = rs1 & imm`

- **or rd, rs1, rs2** (R-format)  
  `rd = rs1 | rs2`

- **ori rd, rs1, imm** (I-format)  
  `rd = rs1 | imm`

- **xor rd, rs1, rs2** (R-format)  
  `rd = rs1 ^ rs2`

- **xori rd, rs1, imm** (I-format)  
  `rd = rs1 ^ imm`

- **not rd, rs1**  
  RISC-V에는 별도 명령어가 없으며,  
  `xori rd, rs1, -1` (I-format)으로 구현한다.

예시:
```asm
addi x5, x0, 0b1010   # x5 = 0b1010
addi x6, x0, 0b1100   # x6 = 0b1100
and  x7, x5, x6       # x7 = 0b1000
or   x8, x5, x6       # x8 = 0b1110
xor  x9, x5, x6       # x9 = 0b0110
xori x10, x5, -1      # x10 = ~x5
```

---

## 3. 시프트 연산

- **sll rd, rs1, rs2** (R-format)  
  `rd = rs1 << rs2` (왼쪽으로 레지스터 값만큼 이동)

- **slli rd, rs1, imm** (I-format)  
  `rd = rs1 << imm` (왼쪽으로 정수값만큼 이동)

- **srl rd, rs1, rs2** (R-format)  
  `rd = rs1 >> rs2` (오른쪽으로 레지스터 값만큼 이동, 상위는 0)

- **srli rd, rs1, imm** (I-format)  
  `rd = rs1 >> imm` (오른쪽으로 정수값만큼 이동, 상위는 0)

- **sra rd, rs1, rs2** (R-format)  
  `rd = rs1 >>> rs2` (산술적 오른쪽 이동, 상위는 부호비트)

- **srai rd, rs1, imm** (I-format)  
  `rd = rs1 >>> imm` (산술적 오른쪽 이동, 상위는 부호비트)

예시:
```asm
addi x5, x0, 0b0001   # x5 = 1
slli x6, x5, 3        # x6 = 0b1000 (1 << 3)

addi x7, x0, -16      # x7 = 0xFFFFFFF0
srai x8, x7, 2        # x8 = 0xFFFFFFFC (부호 확장 유지)
srli x9, x7, 2        # x9 = 0x3FFFFFFC (0으로 채움)
```

---

## ✅ 정리

- **R-format**: 레지스터끼리 연산 (add, sub, and, or, xor, sll, srl, sra)  
- **I-format**: 레지스터와 즉시값 연산 (addi, andi, ori, xori, slli, srli, srai)  
- **not**: 독립된 명령어 없음, `xori rd, rs1, -1`로 대체

---