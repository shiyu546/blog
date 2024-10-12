---
title: 矩阵乘法结合律的证明
date: 2024-10-11 20:14:56
mathjax: true
categories:
- [数学,线性代数]
tags:
- 矩阵
- 结合律
description: "矩阵乘法结合律的证明"
---

## 题目

矩阵A,B,C满足A(BC)和(AB)C都成立，则A(BC)=(AB)C.

## 过程

证明过程分为两步：先比较等式两边的矩阵大小是否相等，再比较对应的元素是否相等。

令A的大小为$m \times n$ ,B的大小为$n \times r$ ,C的大小为$r \times s$.根据矩阵乘法的定义,先计算BC的大小为 $n \times s$,得
$$A(BC) = A_{m\times n}(BC)_{n \times s}$$
=>
$$ A(BC)=(A(BC))_{m \times s} \tag{1}$$
同理，计算(AB)C,得
$$(AB)C = (AB)_{m \times r}C_{r \times s}$$
=>
$$(AB)C=((AB)C)_{m \times s}    \tag{2}$$
由(1),(2)得A(BC)和(AB)C大小相等。

再计算任意对应位置元素的值$(A(BC))_{ij}$,
$$(A(BC))_{ij}=a_{i1}(BC)_{1j}+a_{i2}(BC)_{2j}+...+a_{in}(BC)_{nj}  \tag{3}$$
根据乘法定义，
$$(BC)_{kj}=b_{k1}c_{1j}+b_{k2}c_{2j}+...+b_{kr}c_{rj} \quad\quad k=1,2,...,n$$
将所有的$(BC)_{kj}$带入(3)式，得到展开式：
$$(A(BC))_{ij}=\\
a_{i1}(b_{11}c_{1j}+b_{12}c_{2j}+...+b_{1r}c_{rj}) +\\
a_{i2}(b_{21}c_{1j}+b_{22}c_{2j}+...+b_{2r}c_{rj}) +\\
.\\
.\\
.+\\
a_{in}(b_{n1}c_{1j}+b_{n2}c_{2j}+...+b_{nr}c_{rj})
$$
观察式子每一列都有$c_{kj}$,共r列，将整个加法按$c_{kj}$结合(即每一列结合)，得到
$$(A(BC))_{ij}=\\
c_{1j}(a_{i1}b_{11}+a_{i2}b_{21}+...+a_{in}b_{n1})+\\
c_{2j}(a_{i1}b_{12}+a_{i2}b_{22}+...+a_{in}b_{n2})+\\
.\\
.\\
.+\\
c_{rj}(a_{i1}b_{1r}+a_{i2}b_{2r}+...+a_{in}b_{nr})
$$
=>
$$
(A(BC))_{ij}=\\
c_{1j}(AB)_{i1}+\\
c_{2j}(AB)_{i2}+\\
.\\
.\\
.+\\
c_{rj}(AB)_{ir}
$$
=>
$$(A(BC))_{ij}=(AB)_{i1}c_{1j}+(AB)_{i2}c_{2j}+...+(AB)_{ir}c_{rj}$$
=>
$$ (A(BC))_{ij}=((AB)C)_{ij}$$
得证。
