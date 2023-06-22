---
title: V. Universal and Perfect Hashing
description: Minimizing hash collisions
slug: 451-universal-and-perfect-hashing
date: 2023-06-22 00:00:00+0000
image: waterfall_bridge_cave.jpg
categories:
    - 15-451
    - algorithms
tags:
    - hashing
    - universal hashing
    - perfect hashing
    - randomness
    - matrix
---
## Motivation

A prevalent usecase of hashing is in storing sets or mappings for a subset of the input space --- hash tables. An optimal hash table uniformly distributes elements among its buckets.

## Universal Hashing

### Definition

> A randomized algorithm $H$ for constructing hash functions $h:U\rightarrow\lbrace 0,1,...,M-1\rbrace$ is ***universal*** if $\forall x \neq y \text{ s.t. } x, y\in U$, we have $$\mathbb{P} [h(x)=h(y)|\thinspace h \leftarrow H]\leq\frac{1}{M}$$

### Construction

#### Random Matrix
Suppose keys are $u$-bits long and $M=2^m$. Define $A$ to be a $m$-by-$u$ matrix filled with $0$ and $1$ randomly.
```goat
   <--------- u --------->
  
^  +---------------------+  +-+   +-+
|  |                     |  | |   | |
|  |                     |  | |   | |
m  |          A          |  | |   | | h(x) = Ax
|  |                     |  | | = | |
|  |                     |  |x|   | |
v  +---------------------+  | |   +-+
                            | |
                            | |
                            | |
                            +-+
```
> **Claim**
> : $H=\lbrace h\rbrace$ is universal
>
> **Proof**
> : Consider an arbitrary pair of distinct keys $x, y$. Suppose they differ in the $i$th bit. WLOG, $x_i=0$ and $y_i=1$.
>
> : Observe that regardless of the elements in the $i$th column of $A$, $h(x)=Ax$ since $x_i=0$.
>
> : However, each of the $2^m$ possibilities for the $i$th column of $A$ yield distinct $h(y)=Ay$.
{{<box info>}}A bit flip in the $i$th column of $A$ at row $j$ flips $Ay$ at the $j$th bit{{</box>}}
> : $$\mathbb{P} [Ax=Ay]=\frac{1}{2^m}$$

This is unfortunately quite space inefficient.

#### Random Vector
View the key $x$ as a vector of integers $\langle x_1, x_2, ..., x_k \rangle$ where $0\leq x_i < M$ and $M$ is prime.

Define a $k$-length vector $r_1, r_2, ..., r_k$ filled with random values where $0\leq r_i < M$.

$$h(x)=r\cdot x\mod M$$

> **Claim**
> : $H=\lbrace h\rbrace$ is universal
>
> **Proof**
> : Consider an arbitrary pair of distinct keys $x, y$. Suppose they differ in the $i$th number $x_i \neq y_i$.
>
> : Consider the dot product defined by $h$ excluding the $i$th expression. Specifically,
> : $$h'(x)=\sum_{j\neq i}r_jx_j$$
> Thus,
> : $$h(x)=h'(x)+r_ix_i$$
> Collision between $x, y$ occurs precisely when $h'(x) + r_ix_i = h'(y) + r_iy_i\mod M$.
> : $$r_i(x_i-y_i)=h'(y)-h'(x)\mod M$$
> : Note that because of $M$'s primality, every integer has a multiplicative inverse. Thus, $r_i$ is unique.
> : $$\mathbb{P} [h(x)=h(y)]=\frac{1}{M}$$

## Perfect Hashing
### Definition
> A hash function is ***perfect*** for a set $S, |S|=N$ if all lookups involve $\mathcal{O}(1)$ work.

### Construction
#### Try 1 --- Quadratic Space
Let $H$ be universal and $M=N^2$.
> **Claim**
> : $\mathbb{P}[\exists\text{ collision in $S$}]< \frac{1}{2}$
>
> **Proof**
> : There are $N\choose 2$ pairs $(x, y)$ in $S$. Each pair has at most $\frac{1}{M}=\frac{1}{N^2}$ collision probability by definition of universality.
> : $\mathbb{P}[\exists\text{ collision in $S$}]\leq \frac{N \choose 2}{N^2}<\frac{1}{2}$

#### Try 2 --- Linear Space
Let $H$ be universal and $M=N$. Hash into the first layer with $N$ buckets. Each bucket maps to a secondary layer each with $C_i^2$ slots, where $C_i$ represents the number of elements that collide in the $i$th bucket of the first layer.
```goat
<------------------------------ N ------------------------------>

+-------+-------+-------+-------+-------+-------+-------+-------+
|       |       |       |       |       |       |       |       |
|   o   |   o   |   o   |   o   |   o   |   o   |   o   |   o   |
|   |   |   |   |   |   |   |   |       |   |   |   |   |   |   |
+---+---+---+---+---+---+---+---+-------+---+---+---+---+---+---+
    |    .-'        |       |    .---------'        |       |
    |   |      .---'         '--+----------------.  |       v
 .-'    v     |                 |                 | |   +-------+
|   +-------+ |                 v                 | |   |       |
|   |       | | +-------+-------+-------+-------+ | |   |       |
|   |       | | |       |       |       |       | | |   |       |
|   |       | | |       |       |       |       | | |   +-------+
|   +-------+ | |       |       |       |       | | |
|             | +-------+-------+-------+-------+ | |
|             v                                 .-+'    +-------+
|   +-------+-------+-------+-------+          |  |     |       |
|   |       |       |       |       |          v   '--->|       |
 '->|       |       |       |       |      +-------+    |       |
    |       |       |       |       |      |       |    +-------+
    +-------+-------+-------+-------+      |       |
                                           |       |
                                           +-------+
```
> **Theorem**
> : $\mathbb{P}[\sum_iC_i^2 > 4N]<\frac{1}{2}$
>
> **Proof**
> : Let $I_{xy}$ be an indicator that $x,y$ collide. Observe that within any secondary layer with $C_i$ elements ($C_i^2$ slots), for any two elements $x, y$, $I_{xy}=1$ (including $I_{xx}$, this amounts to $C_i^2$).
> : $$\begin{align*}\mathbb{E}[\sum_iC_i^2]&=\mathbb{E}[\sum_x\sum_yI_{xy}]\newline&=N+\sum_x\sum_{y\neq x}\mathbb{E}[C_{xy}]\newline&\leq N+\frac{N(N-1)}{M}\newline&=N+\frac{N(N-1)}{N}\newline&<2N\end{align*}$$
> : By [Markov's Inequality](https://en.wikipedia.org/wiki/Markov%27s_inequality), the problem statement is proven.
> {{<box info>}}$$\mathbb{P}[X\geq a]\leq\frac{\mathbb{E}[X]}{a}$${{</box>}}
