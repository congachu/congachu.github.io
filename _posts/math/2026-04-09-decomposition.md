---
title: "[Math] 행렬 분해"
date: 2026-04-09 20:00:00 +0900
categories: [Math, Linear Algebra]
tags: [Math, Linear Algebra, Decomposition]
description: "행렬 분해 종류"
toc: true
---

## Cholesky 분해

**SPD 행렬 $A \in \mathbb{R}^{n \times n}$**에 대하여, 양의 대각원소를 가지는 하삼각행렬 $L$이 존재하여 다음을 만족한다.

- $A = LL^T$

**유용성**

- 역행렬 계산 : $A^{-1} = (LL^T)^{-1} = (L^T)^{-1}L^{-1}$
- $L^{-1}$는 RREF 계산을 통해 쉽게 계산 가능하다.
- 행렬식 계산: $\det(A) = \det(L)^2$

---

## 고유분해와 대각화

- **목표:** 정방행렬 $A \in \mathbb{R}^{n \times n}$와 닮은(similar) 대각행렬 $D$를 찾는 것.
- **목적:** 행렬식, 역행렬 등을 계산하기 쉬워진다.
- **정의:** 만약 행렬 $A \in \mathbb{R}^{n \times n}$가 대각행렬 $D = P^{-1}AP$와 닮음이면, $A$를 **대각화 가능(diagonalizable)**하다고 한다.  
여기서 $P$는 $A$의 고유벡터들을 열벡터로 가지는 행렬이고, $D$는 고유값들을 대각 원소로 가지는 행렬이다.

- **대각화 가능성 확인:**
    - $A$의 고유벡터들을 모았을 때 새로운 기저를 형성하면 대각화 가능하다.
    - $P = [p_1,...,p_n]$이고 $(\lambda_1,...,\lambda_n)$이 고유값들, $(p_1,...,p_n)$이 그에 해당하는 고유벡터들이라면
        - $AP = PD$
        - 여기서 $D$는 고유값들을 대각 원소로 가지는 대각행렬이다.
    - 따라서 $P$가 역행렬을 가져야 하므로, $(p_1,...,p_n)$이 **선형 독립**이어야 한다.

---

## 고유분해와 대각화 특성

- $A$가 대각화 가능 $\Leftrightarrow$ $A$가 $n$개의 선형 독립인 고유벡터들을 가지고 있음.
- 대칭행렬 $S$는 항상 대각화 가능하며, 다음과 같이 표현된다.
    - $S = PDP^{-1}$
    - 여기서 $P$는 직교행렬 $(P^{-1}=P^T)$이며, $D$는 고유값으로 구성된 대각행렬이다.
- 행렬의 거듭제곱 계산
    - $A^k = (PDP^{-1})^k = PD^kP^{-1}$

---

## 특이값 분해 (Singular Value Decomposition, SVD)

**SVD**는 $m \times n$ 형태의 임의의 직사각행렬 $A \in \mathbb{R}^{m \times n}$를 세 개의 행렬의 곱으로 분해하는 방법이다.  
기존의 행렬 분해는 정방행렬에 한정 되었지만, **SVD**는 모든 행렬에 적용 가능하다.

- $A = U \Sigma V^T$
    - $U \in \mathbb{R}^{m \times m}$: $AA^T$의 고유벡터들로 이루어진 직교행렬. 열벡터 $u_i$들은 정규직교
    - $\Sigma \in \mathbb{R}^{m \times n}$: 특이값 $\sigma_i$들을 대각 원소로 가지는 직사각 대각 행렬이다.  
      특이값들은 항상 양수이며 내림차순으로 정렬된다.  
      $\sigma_1 \ge \sigma_2 \ge \dots \ge \sigma_r \ge 0$, 여기서 $r = \min(m,n)$은 행렬의 랭크
    - $V \in \mathbb{R}^{n \times n}$: $A^TA$의 고유벡터들로 이루어진 직교행렬. 열벡터 $v_i$들은 정규직교

---

## 대칭행렬의 SVD

- 대칭행렬 $S = S^T$의 고유분해는 $S = PDP^T$ 형태로 나타난다.
- 대칭행렬의 SVD는 $S = U \Sigma V^T$ 형태를 가지므로, $U = P, V = P, D = \Sigma$와 같은 관계를 가진다.  
고유값의 부호에 따라 차이가 있을 수 있다.
- 따라서, 대칭행렬의 SVD는 고유분해와 매우 유사하다

---

## SVD를 계산하는 법

- **Right singular vector($v_1, ... , v_n \in \mathbb{R}^n$) 구성:**
    - $A^TA$의 고유벡터와 고유값을 이용한다.
    - 
    $$
    A^TA = PDP^T = (U\Sigma V^T)^T(U\Sigma V^T) = V\Sigma^T U^T U \Sigma V^T = V\Sigma^2 V^T
    $$
    - 여기서 $P = V$이고 $D = \Sigma^2$이다.  
    즉, $A^TA$의 고유값 $\lambda_i$는 특이값의 제곱 $\sigma_i^2$이 된다:  
    $\sigma_i^2 = \lambda_i$

- **Left singular vector($u_1,...,u_m \in \mathbb{R}^m$) 구성:**
    - $AA^T$의 고유벡터와 고유값을 이용한다.
    - 
    $$
    AA^T = PDP^T = (U\Sigma V^T)(U\Sigma V^T)^T = U\Sigma V^T V \Sigma^T U^T = U\Sigma^2 U^T
    $$
    - 여기서 $P = U$이고 $D = \Sigma^2$이다.  
    즉, $AA^T$의 고유값 $\lambda_i$는 특이값의 제곱 $\sigma_i^2$이 된다:  
    $\sigma_i^2 = \lambda_i$

- 결론: $A^TA$와 $AA^T$의 고유값이 같다.

---