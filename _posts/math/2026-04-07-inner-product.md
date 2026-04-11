---
title: "[Math] 내적"
date: 2026-04-07 20:00:00 +0900
categories: [Math, Linear Algebra]
tags: [Math, Linear Algebra, Inner Product, Norm]
description: "노름, 직교, 각도 등 내적 관련"
toc: true
---

## 노름 (Norm)

벡터 공간 $V$에서 **노름(norm)**은 각 벡터 $v \in V$를 그 길이 $\|v\|$에 매핑하는 함수이다.

- **양의 정부호성**: $\|v\| \ge 0$, $\|v\| = 0 \Leftrightarrow v = 0$
- **동차성**: $\|\alpha v\| = |\alpha| \|v\|$
- **삼각 부등식**: $\|v+w\| \le \|v\| + \|w\|$

---

## 노름의 예시

- **L1 norm**
  $$
  \|x\|_1 = \sum_{i=1}^{n} |x_i|
  $$

- **L2 norm**
  $$
  \|x\|_2 = \sqrt{\sum_{i=1}^{n} x_i^2}
  $$

---

## 내적 (Inner Product)

내적은 두 벡터를 스칼라로 보내는 함수 $\langle v, w \rangle$

- **대칭성**: $\langle v, w \rangle = \langle w, v \rangle$
- **선형성**: $\langle \alpha u + \beta v, w \rangle = \alpha \langle u, w \rangle + \beta \langle v, w \rangle$
- **양의 정부호성**: $\langle v, v \rangle \ge 0$, $=0 \Leftrightarrow v=0$

---

## 쌍선형 사상

$B: V \times V \to \mathbb{R}$

- $B(\alpha u + \beta v, w) = \alpha B(u,w) + \beta B(v,w)$
- $B(u, \alpha v + \beta w) = \alpha B(u,v) + \beta B(u,w)$

---

## 내적의 정의

대칭 + 양의정부호 + 쌍선형 ⇒ 내적

$$
(V, \langle \cdot, \cdot \rangle)
$$

---

## 내적 예시

$$
\langle x, y \rangle = x_1y_1 - (x_1y_2 + x_2y_1) + 2x_2y_2
$$

$$
\langle x,x \rangle = (x_1 - x_2)^2 + x_2^2
$$

---

## 내적의 특성

- $\|v\| = \sqrt{\langle v,v \rangle}$
- **코시-슈바르츠**
  $$
  |\langle v,w \rangle| \le \|v\| \|w\|
  $$

---

## 행렬과 내적

$$
\langle x, y \rangle = [x]_B^T A [y]_B
$$

- $A_{ij} = \langle b_i, b_j \rangle$

---

## SPD 행렬

- $A^T = A$
- $z^T A z > 0$

---

## 거리

$$
d(v,w) = \|v-w\| = \sqrt{\langle v-w, v-w \rangle}
$$

---

## 각도

$$
\cos \theta = \frac{\langle v,w \rangle}{\|v\|\|w\|}
$$

---

## 직교

$$
\langle x,y \rangle = 0
$$

---

## 직교 행렬

$$
A^T A = I
$$

---

## 사영 (Projection)

$$
P(P(v)) = P(v)
$$

---

## 1차원 사영

$$
p = \frac{\langle x,u \rangle}{\|u\|^2} u
$$

단위벡터이면

$$
p = \langle x,u \rangle u
$$

---

## 행렬 형태

$$
p = (u^T x) u = uu^T x
$$

---

## m차원 사영

$$
p = B(B^T B)^{-1} B^T x
$$

---

## 회전 (2D)

$$
R(\theta) =
\begin{pmatrix}
\cos \theta & -\sin \theta \\
\sin \theta & \cos \theta
\end{pmatrix}
$$

---

## 3D 회전

$$
R_x(\theta) =
\begin{pmatrix}
1 & 0 & 0 \\
0 & \cos\theta & -\sin\theta \\
0 & \sin\theta & \cos\theta
\end{pmatrix}
$$

---

## 핵심 정리

- 노름 → 길이
- 내적 → 각도/거리
- SPD → 내적의 행렬 표현
- 사영 → 최단 거리
- 직교 → 각도 보존