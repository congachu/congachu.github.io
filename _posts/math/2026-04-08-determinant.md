---
title: "[Math] 행렬식"
date: 2026-04-08 20:00:00 +0900
categories: [Math, Linear Algebra]
tags: [Math, Linear Algebra, Determinant]
description: "행렬식과 고유값, 고유공간"
toc: true
---

---

## 행렬식 (Determinant)

행렬식은 $n \times n$ 정방행렬에 대해 정의되는 함수이다.

$$
\det(A) =
\begin{vmatrix}
a_{11} & a_{12} & \cdots & a_{1n} \\
a_{21} & a_{22} & \cdots & a_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
a_{n1} & a_{n2} & \cdots & a_{nn}
\end{vmatrix}
$$

행렬식은 행렬에 하나의 수를 대응시키며, 선형변환의 성질을 나타낸다.

---

## 가역성과 관계

행렬 $A$ 에 대해

$$
\det(A) \neq 0 \Leftrightarrow A \text{ is invertible}
$$

즉, 행렬식이 0이 아니면 역행렬이 존재하고, 0이면 존재하지 않는다.

---

## 삼각행렬의 행렬식

삼각행렬(상삼각, 하삼각)의 경우

$$
\det(A) = \prod_{i=1}^{n} a_{ii}
$$

즉, 대각 성분의 곱으로 계산된다.

---

## 행렬식의 의미

행렬식은 선형변환이 공간을 얼마나 "늘리거나 줄이는지"를 나타낸다.

- $\det(A) > 0$ : 방향 유지
- $\det(A) < 0$ : 방향 반전
- $\det(A) = 0$ : 차원 축소 (선형 종속)

즉, 행렬식은 부피(혹은 면적)의 스케일 변화량이다.

---

## 코팩터 전개 (Cofactor Expansion)

행렬식은 다음과 같이 전개할 수 있다.

$$
\det(A) = \sum_{k=1}^{n} (-1)^{k+j} a_{kj} \det(A_{k,j})
$$

여기서 $A_{k,j}$ 는 $k$행 $j$열을 제거한 부분행렬이다.

---

## 행렬식의 성질

- $\det(AB) = \det(A)\det(B)$
- $\det(A^T) = \det(A)$
- $\det(A^{-1}) = \frac{1}{\det(A)}$

---

## Trace (트레이스)

행렬 $A$ 의 대각 원소의 합을 trace 라고 한다.

$$
\operatorname{tr}(A) = \sum_{i=1}^{n} a_{ii}
$$

---

## 특성 방정식 (Characteristic Equation)

고유값을 구하기 위해 다음 방정식을 사용한다.

$$
\det(A - \lambda I) = 0
$$

---

## 고유값과 고유벡터

행렬 $A$ 와 벡터 $x$ 에 대해

$$
Ax = \lambda x
$$

를 만족하면,

- $\lambda$ : 고유값 (eigenvalue)
- $x$ : 고유벡터 (eigenvector)

---

## 고유공간

고유값 $\lambda$ 에 대해

$$
(A - \lambda I)x = 0
$$

의 해 공간을 고유공간이라고 한다.

---

## 고유값의 성질

- $\det(A) = \prod \lambda_i$
- $\operatorname{tr}(A) = \sum \lambda_i$

---

## 고유값 계산 예시

$$
A =
\begin{pmatrix}
4 & 2 \\
1 & 3
\end{pmatrix}
$$

특성 방정식:

$$
\det(A - \lambda I)
=
\begin{vmatrix}
4-\lambda & 2 \\
1 & 3-\lambda
\end{vmatrix}
$$

$$
= (4-\lambda)(3-\lambda) - 2
$$

$$
= \lambda^2 - 7\lambda + 10
$$

$$
= (\lambda - 2)(\lambda - 5)
$$

따라서

$$
\lambda = 2, 5
$$

---

## 고유벡터 계산

### $\lambda = 2$

$$
A - 2I =
\begin{pmatrix}
2 & 2 \\
1 & 1
\end{pmatrix}
$$

$$
\begin{pmatrix}
2 & 2 \\
1 & 1
\end{pmatrix}
\begin{pmatrix}
x_1 \\
x_2
\end{pmatrix}
= 0
$$

$$
x_1 = -x_2
$$

---

### $\lambda = 5$

$$
A - 5I =
\begin{pmatrix}
-1 & 2 \\
1 & -2
\end{pmatrix}
$$

$$
\begin{pmatrix}
-1 & 2 \\
1 & -2
\end{pmatrix}
\begin{pmatrix}
x_1 \\
x_2
\end{pmatrix}
= 0
$$

$$
x_1 = 2x_2
$$