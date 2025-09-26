---
title: "[ComputerArch] RISC-V 명령어 포맷 (R, I, U, S)"
date: 2025-09-27 04:00:00 +0900
categories: [Computer Architecture, RISC-V]
tags: [RISC-V, Instruction Format, Computer Architecture]
description: "RISC-V 명령어 포맷 정리"
toc: true
---

# RISC-V 명령어 포맷 상세 정리

RISC-V 명령어는 고정 길이 32비트로 구성되며, 다양한 **포맷(Format)**으로 구분된다.  
여기서는 **R, I, U, S 포맷**을 중점적으로 설명한다.

---

## 1. R-format (Register Format)

- **주요 용도**: 레지스터 간 산술/논리 연산 (ADD, SUB, AND, OR, XOR, SLL, SRL, SRA 등)
- **구조**
```
| funct7 (7) | rs2 (5) | rs1 (5) | funct3 (3) | rd (5) | opcode (7) |
```
- **필드 설명**
  - `opcode`: 어떤 명령어 그룹인지 결정 (예: 산술/논리 연산은 0110011)
  - `rd`: 결과가 저장될 목적지 레지스터
  - `rs1`, `rs2`: 연산에 사용될 소스 레지스터
  - `funct3`: 세부 연산 구분 (예: 000=ADD, 100=XOR 등)
  - `funct7`: 추가적인 연산 구분 (예: 0000000=ADD, 0100000=SUB)

- **예시**
```asm
add x5, x6, x7   # x5 = x6 + x7
```
비트 구조 (ADD):  
```
funct7=0000000 | rs2=00111 | rs1=00110 | funct3=000 | rd=00101 | opcode=0110011
```

---

## 2. I-format (Immediate Format)

- **주요 용도**: 즉시값(Immediate)과 연산, Load 명령어, JALR 점프
- **구조**
```
| imm[11:0] (12) | rs1 (5) | funct3 (3) | rd (5) | opcode (7) |
```
- **필드 설명**
  - `imm[11:0]`: 12비트 즉시값, 부호 확장(sign-extend)됨
  - `rs1`: 기준이 되는 소스 레지스터
  - `rd`: 결과 저장 레지스터
  - `funct3`: 연산 종류 구분
  - `opcode`: 명령어 그룹 (예: addi=0010011, lw=0000011)

- **예시**
```asm
addi x5, x6, 10   # x5 = x6 + 10
```
비트 구조 (ADDI):  
```
imm=000000001010 | rs1=00110 | funct3=000 | rd=00101 | opcode=0010011
```

---

## 3. U-format (Upper Immediate Format)

- **주요 용도**: 상위 20비트를 즉시값으로 채움 (LUI, AUIPC)
- **구조**
```
| imm[31:12] (20) | rd (5) | opcode (7) |
```
- **필드 설명**
  - `imm[31:12]`: 20비트 즉시값 (하위 12비트는 0으로 채워짐)
  - `rd`: 결과 저장 레지스터
  - `opcode`: 명령어 그룹 (예: LUI=0110111, AUIPC=0010111)

- **예시**
```asm
lui x5, 0x12345   # x5 = 0x12345 << 12
```

---

## 4. S-format (Store Format)

- **주요 용도**: 메모리에 값 저장 (SB, SH, SW 등)
- **구조**
```
| imm[11:5] (7) | rs2 (5) | rs1 (5) | funct3 (3) | imm[4:0] (5) | opcode (7) |
```
- **필드 설명**
  - `imm[11:5]` + `imm[4:0]`: 합쳐서 12비트 즉시값 (부호 확장됨)
  - `rs1`: 기준 주소를 가진 레지스터 (base register)
  - `rs2`: 저장할 값을 가진 소스 레지스터
  - `funct3`: 저장 크기 구분 (000=SB, 001=SH, 010=SW)
  - `opcode`: 명령어 그룹 (Store=0100011)

- **예시**
```asm
sw x5, 8(x6)   # 메모리[R[x6] + 8] = R[x5]
```
비트 구조 (SW):  
```
imm[11:5]=0000000 | rs2=00101 | rs1=00110 | funct3=010 | imm[4:0]=01000 | opcode=0100011
```

---

## ✅ 정리
- **R-format**: 레지스터 ↔ 레지스터 연산  
- **I-format**: 레지스터 ↔ 즉시값 연산, Load 계열  
- **U-format**: 상위 20비트 즉시값 (LUI, AUIPC)  
- **S-format**: 메모리에 값 저장 (Store 계열)  

---
