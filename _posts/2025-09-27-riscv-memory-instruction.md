---
title: "[ComputerArch] RISC-V 메모리 접근 명령어 정리"
date: 2025-09-27 05:10:00 +0900
categories: [Computer Architecture, RISC-V]
tags: [RISC-V, Assembly, Computer Architecture]
description: "RISC-V 메모리 접근 명령어"
toc: true
---

# RISC-V 메모리 접근 명령어 정리 (Byte/Halfword/Word)

RISC-V는 **Load/Store 구조**를 사용한다.  
즉, 연산은 반드시 레지스터에서만 수행되며, 메모리에 접근할 때는 load/store 명령어를 이용한다.

이번 정리에서는 **Byte(1B), Halfword(2B), Word(4B)** 단위 명령어들을 모두 다룬다.

---

## 1. Load 계열 (I-format)

### 1) `lw rd, offset(rs1)` (Load Word)
- 메모리에서 4바이트(32비트)를 읽어와 `rd`에 저장
- I-format 사용
- `addr = R[rs1] + offset`
- `R[rd] = Mem[addr .. addr+3]`

예시:
```asm
lw x5, 0(x10)   # x5 = Mem[x10 + 0] (4바이트 읽기)
```

---

### 2) `lh rd, offset(rs1)` (Load Halfword)
- 메모리에서 2바이트(16비트)를 읽어와 `rd`에 저장
- sign-extend: 16비트의 최상위 비트를 복사해 상위 16비트를 채움
- I-format 사용

예시:
```asm
lh x5, 2(x10)   # x5 = sign-extend(Mem[x10+2 .. x10+3])
```

---

### 3) `lhu rd, offset(rs1)` (Load Halfword Unsigned)
- 메모리에서 2바이트(16비트)를 읽어와 `rd`에 저장
- zero-extend: 상위 16비트를 0으로 채움
- I-format 사용

예시:
```asm
lhu x5, 2(x10)  # x5 = zero-extend(Mem[x10+2 .. x10+3])
```

---

### 4) `lb rd, offset(rs1)` (Load Byte)
- 메모리에서 1바이트(8비트)를 읽어와 `rd`에 저장
- sign-extend: 7번째 비트를 복사해 상위 24비트를 채움
- I-format 사용

예시:
```asm
lb x5, 0(x10)   # x5 = sign-extend(Mem[x10])
```

---

### 5) `lbu rd, offset(rs1)` (Load Byte Unsigned)
- 메모리에서 1바이트(8비트)를 읽어와 `rd`에 저장
- zero-extend: 상위 24비트를 0으로 채움
- I-format 사용

예시:
```asm
lbu x5, 0(x10)  # x5 = zero-extend(Mem[x10])
```

---

## 2. Store 계열 (S-format)

### 1) `sw rs2, offset(rs1)` (Store Word)
- 레지스터의 값(32비트)을 메모리에 저장
- S-format 사용
- `addr = R[rs1] + offset`
- `Mem[addr .. addr+3] = R[rs2]`

예시:
```asm
sw x5, 8(x10)   # Mem[x10 + 8] = x5 (4바이트 저장)
```

---

### 2) `sh rs2, offset(rs1)` (Store Halfword)
- 레지스터의 값 중 하위 16비트만 메모리에 저장
- S-format 사용
- `addr = R[rs1] + offset`
- `Mem[addr .. addr+1] = R[rs2][15:0]`

예시:
```asm
sh x5, 4(x10)   # Mem[x10+4 .. x10+5] = x5의 하위 16비트
```

---

### 3) `sb rs2, offset(rs1)` (Store Byte)
- 레지스터의 값 중 하위 8비트만 메모리에 저장
- S-format 사용
- `addr = R[rs1] + offset`
- `Mem[addr] = R[rs2][7:0]`

예시:
```asm
sb x5, 0(x10)   # Mem[x10] = x5의 하위 1바이트
```

---

## ✅ 정리

- **Load 명령어 (I-format)**  
  - `lw`: 4바이트 읽기  
  - `lh`: 2바이트 읽기 + 부호 확장  
  - `lhu`: 2바이트 읽기 + 0 확장  
  - `lb`: 1바이트 읽기 + 부호 확장  
  - `lbu`: 1바이트 읽기 + 0 확장  

- **Store 명령어 (S-format)**  
  - `sw`: 4바이트 저장  
  - `sh`: 2바이트 저장  
  - `sb`: 1바이트 저장  

---