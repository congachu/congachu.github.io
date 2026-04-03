---
title: "[Math] 변환과 공간"
date: 2026-04-03 21:00:00 +0900
categories: [Math, Linear Algebra]
tags: [Math, Linear Algebra, Mapping, Space]
description: "기저 변환 이미지 공간 영 공간 아핀 공간"
toc: true
---

---

# 변환

## 기저 변환

- 벡터 공간 $V$의 기저와 $W$의 기저가 변할 때, 선형 변환 $\Phi : V \to W$의 표현이 어떻게 변하는가?
- $V$의 서로 다른 두 기저 $B = (b_1, ..., b_n)$와 $\tilde{B} = (\tilde{b}_1, ..., \tilde{b}_n)$,  
  $W$의 서로 다른 두 기저 $C = (c_1, ..., c_m)$와 $\tilde{C} = (\tilde{c}_1, ..., \tilde{c}_m)$가 있을 때,

- 기존 기저에서의 변환 행렬 $A_\Phi \in \mathbb{R}^{m \times n}$  
- 새로운 기저에서의 변환 행렬 $\tilde{A}_\Phi \in \mathbb{R}^{m \times n}$

이 둘의 관계를 구하는 것이 핵심이다.

---

## 기저 변환 정리

$$
\tilde{A}_\Phi = T^{-1} A_\Phi S
$$

- $S \in \mathbb{R}^{n \times n}$: $\tilde{B}$ 좌표 → $B$ 좌표 변환
- $T \in \mathbb{R}^{m \times m}$: $\tilde{C}$ 좌표 → $C$ 좌표 변환

---

## 증명

### 1. 새로운 기저 표현

$$
\tilde{b}_j = \sum_{i=1}^{n} s_{ij} b_i
$$

여기서 $S = (s_{ij})$는 기저 변환 행렬

---

$$
\tilde{c}_k = \sum_{l=1}^{m} t_{lk} c_l
$$

여기서 $T = (t_{lk})$

---

### 2. $\Phi(\tilde{b}_j)$ 두 가지 표현

#### (1) 새로운 기저 $\tilde{C}$ 기준

$$
\Phi(\tilde{b}_j)
= \sum_{k=1}^{m} \tilde{a}_{kj} \tilde{c}_k
= \sum_{k=1}^{m} \tilde{a}_{kj} \left( \sum_{l=1}^{m} t_{lk} c_l \right)
= \sum_{l=1}^{m} \left( \sum_{k=1}^{m} t_{lk} \tilde{a}_{kj} \right) c_l
$$

---

#### (2) 기존 기저 $B$ 기준

$$
\Phi(\tilde{b}_j)
= \Phi\left( \sum_{i=1}^{n} s_{ij} b_i \right)
= \sum_{i=1}^{n} s_{ij} \Phi(b_i)
= \sum_{i=1}^{n} s_{ij} \left( \sum_{l=1}^{m} a_{li} c_l \right)
= \sum_{l=1}^{m} \left( \sum_{i=1}^{n} a_{li} s_{ij} \right) c_l
$$

---

### 3. 계수 비교

$$
\sum_{k=1}^{m} t_{lk} \tilde{a}_{kj}
=
\sum_{i=1}^{n} a_{li} s_{ij}
$$

---

### 4. 행렬 형태

$$
T \tilde{A}_\Phi = A_\Phi S
$$

---

### 5. 정리

$$
\tilde{A}_\Phi = T^{-1} A_\Phi S
$$

---

## 동치와 닮음

- **동치 (Equivalent)**

$$
\tilde{A} = T^{-1} A S
$$

→ 서로 다른 기저

---

- **닮음 (Similar)**

$$
\tilde{A} = S^{-1} A S
$$

→ 같은 공간, 기저만 변경

---

## 기저 변환 예시

선형 변환 $\Phi : \mathbb{R}^3 \to \mathbb{R}^4$

$$
A_\Phi =
\begin{pmatrix}
1 & 2 & 0 \\
-1 & 1 & 3 \\
3 & 7 & 1 \\
-1 & 2 & 4
\end{pmatrix}
$$

---

$$
S =
\begin{pmatrix}
1 & 0 & 1 \\
1 & 1 & 0 \\
0 & 1 & 1
\end{pmatrix}
$$

---

$$
T =
\begin{pmatrix}
1 & 1 & 0 & 1 \\
1 & 0 & 1 & 0 \\
0 & 1 & 1 & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$

---

$$
T^{-1} =
\frac{1}{2}
\begin{pmatrix}
1 & 1 & -1 & -1 \\
1 & -1 & 1 & -1 \\
-1 & 1 & 1 & 1 \\
0 & 0 & 0 & 2
\end{pmatrix}
$$

---

$$
\tilde{A}_\Phi = T^{-1} A_\Phi S
$$

---

$$
\tilde{A}_\Phi =
\begin{pmatrix}
-4 & -4 & -2 \\
6 & 0 & 0 \\
4 & 8 & 4 \\
1 & 6 & 3
\end{pmatrix}
$$

---

## 랭크 성질

- $rk(A) = rk(A^T)$
- $dim(Im(A)) = rk(A)$
- $dim(Row(A)) = rk(A)$
- $A$ 가 가역 $\Leftrightarrow rk(A) = n$
- $Ax = b$ 해 존재 $\Leftrightarrow rk(A) = rk(A|b)$
- $dim(Null(A)) = n - rk(A)$
- $rk(A) = \min(m,n)$ → full rank

---

## 이미지 공간 / 영 공간

- **Kernel**

$$
ker(\Phi) = \{ v \in V \mid \Phi(v) = 0 \}
$$

---

- **Image**

$$
Im(\Phi) = \{ w \in W \mid \exists v \in V, \Phi(v) = w \}
$$

---

## 성질

- $\Phi(0) = 0$
- $ker(\Phi) \subseteq V$
- $Im(\Phi) \subseteq W$
- $\Phi$ 단사 $\Leftrightarrow ker(\Phi) = \{0\}$

---

## Rank-Nullity

$$
dim(ker(\Phi)) + dim(Im(\Phi)) = dim(V)
$$

---

## 아핀 공간

$$
L = x_0 + U = \{ x_0 + u \mid u \in U \}
$$

- $U$: 방향 공간
- $x_0$: 기준점

---

## 성질

$$
L \subseteq \tilde{L}
\Leftrightarrow
U \subseteq \tilde{U}, \; x_0 - \tilde{x}_0 \in \tilde{U}
$$

---

## 표현

$$
x = x_0 + \lambda_1 b_1 + ... + \lambda_k b_k
$$

---

## 아핀 변환

$$
\phi(x) = a + \Phi(x)
$$

---

## 성질

- $\phi(0) = a$
- 합성 → 아핀 변환 유지

---