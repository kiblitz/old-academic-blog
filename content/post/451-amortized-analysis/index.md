---
title: III. Amortized Analysis
description: Tightening algorithmic time bounds based on execution sequences
slug: 451-amortized-analysis
date: 2023-06-15 00:00:00+0000
image: waterfall_forest_pool.jpg
categories:
    - 15-451
    - algorithms
tags:
    - binary counter
    - unbounded array
    - amortized analysis
---

## Introduction

### Motivation

Analyzing an algorithm's worst-case time complexity can be too pessimistic. Perhaps an algorithm $\mathcal{A}$ is usually cheap, but every $n$ executions, it is expensive. Should $\mathcal{A}$ be categorized as an expensive algorithm?

### Definition

The amortized time cost for $\mathcal{A}$ is the average time cost per execution across **any** sequence of executions of length $n$ for some determined $n$.

{{<box info>}}$n=1$ is worst-case analysis, but larger $n$ can offer more optimistic analysis{{</box>}}
## Examples

### Binary Counter

#### Problem

> We want to store a big binary counter in an array $A$ initialized to $0$.
> The cost model is every bit flip.

{{<box info>}}
$$
\begin {array}{ccccc|c}
\underline{A_4}\quad & \underline{A_3}\quad & \underline{A_2}\quad & \underline{A_1}\quad & \underline{A_0}\quad & \quad\underline{\textbf{cost}} \newline
0\quad & 0\quad & 0\quad & 0\quad & 0\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 0\quad & 0\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad2\newline
0\quad & 0\quad & 0\quad & \boxed{1}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 0\quad & 1\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad3\newline
0\quad & 0\quad & \boxed{1}\quad & \boxed{0}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 1\quad & 0\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad2\newline
0\quad & 0\quad & 1\quad & \boxed{1}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 1\quad & 1\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad4\newline
0\quad & \boxed{1}\quad & \boxed{0}\quad & \boxed{0}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 1\quad & 0\quad & 0\quad & \boxed{1}\quad &
\end {array}$$
{{</box>}}

The worst-case time complexity is $\mathcal{O}(\log n)$ since at worst $n$ becomes a power of $2$, and we have to flip all of its significant bits.

#### Amortized Analysis

> **Theorem**
> : The amortized time cost is at most $2$.
>
> **Proof**
> : Consider $n$ (for large $n$) executions beginning at any state. $A_0$ flips every execution. $A_1$ flips every other execution. $A_2$ flips every 4 executions. $A_k$ flips every $2^k$ executions.
>
> : The total cost is the sum of these flips.
> : $$n + \frac{n}{2} + \frac{n}{4} + ... \leq 2n$$
> : Thus, the average per execution is at most $2\in\mathcal{O}(1)$.
{{<box important>}}The "beginning at any state" is important. Otherwise, our analysis is not generalizable.{{</box>}}

### Expensive Binary Counter

#### Problem

> We want to store a big binary counter in an array $A$ initialized to $0$.
> It costs $2^k$ to flip bit $A_k$.

{{<box info>}}Same problem but with exponential costs{{</box>}}

{{<box info>}}
$$
\begin {array}{ccccc|c}
\underline{A_4}\quad & \underline{A_3}\quad & \underline{A_2}\quad & \underline{A_1}\quad & \underline{A_0}\quad & \quad\underline{\textbf{cost}} \newline
0\quad & 0\quad & 0\quad & 0\quad & 0\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 0\quad & 0\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad3\newline
0\quad & 0\quad & 0\quad & \boxed{1}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 0\quad & 1\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad7\newline
0\quad & 0\quad & \boxed{1}\quad & \boxed{0}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 1\quad & 0\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad3\newline
0\quad & 0\quad & 1\quad & \boxed{1}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 0\quad & 1\quad & 1\quad & \boxed{1}\quad & \newline
  &   &   &   &   & \quad15\newline
0\quad & \boxed{1}\quad & \boxed{0}\quad & \boxed{0}\quad & \boxed{0}\quad & \newline
  &   &   &   &   & \quad1\newline
0\quad & 1\quad & 0\quad & 0\quad & \boxed{1}\quad &
\end {array}$$
{{</box>}}

The worst-case time complexity is $\mathcal{O}(n)$ since at worst $n$ becomes a power of $2$, and we have to flip all of its significant bits ($1+2+4+...+\frac{n}{2}+n\leq2n$).

#### Amortized Analysis

> **Theorem**
> : The amortized time cost is at most $\log_2 (n) + 1$.
>
> **Proof**
> : Consider $n$ (for large $n$) executions beginning at any state. $A_0$ flips every execution for a cost of $1$. $A_1$ flips every other execution for a cost of $2$. $A_2$ flips every 4 executions for a cost of $4$. $A_k$ flips every $2^k$ executions for a cost of $2^k$.
>
> : $n$ has $\lfloor\log_2 n + 1\rfloor$ significant bits.
>
> : The total cost is the sum of these flips.
> : $$\underbrace{n + 2\cdot\frac{n}{2} + 4\cdot\frac{n}{4} + ...}_{\lfloor\log_2 n + 1\rfloor} = n\lfloor\log_2 n + 1\rfloor\$$
> : Thus, the average per execution is $\lfloor\log_2 n + 1\rfloor\in\mathcal{O}(\log n)$.

### Unbounded Array
#### Problem
> We want to store a linear stream of data. We start with an Array $\mathcal{A}$ of memory space of size $1$. Every new append inserts into $\mathcal{A}$. If an append is attempted when $\mathcal{A}$ is completely filled, we must reallocate memory at double the size and re-insert previous data before inserting the new data.
>
> It costs $1$ to insert one slot of data.

{{<box info>}}
$$
\begin {array}{c|c}
\underline{A} & \underline{\textbf{cost}} \newline
\Box&\newline
&1\newline
\blacksquare&\newline
&2\newline
\blacksquare\blacksquare&\newline
&3\newline
\Box\blacksquare\blacksquare\blacksquare&\newline
&1\newline
\blacksquare\blacksquare\blacksquare\blacksquare&\newline
&5\newline
\Box\Box\Box\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare&\newline
&1\newline
\Box\Box\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare&\newline
&1\newline
\Box\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare&\newline
&1\newline
\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare&\newline
&9\newline
\Box\Box\Box\Box\Box\Box\Box\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare\blacksquare&\newline
\end {array}$$
{{</box>}}

The worst-case time complexity is $\mathcal{O}(n)$ with respect to the data size since at worst, we are in a reallocation step where we must re-insert all old data.

#### Amortized Analysis

> **Theorem**
> : The amortized time cost is at most $3$.
>
> **Proof**
> : Consider $n$ (for large $n$) executions beginning at any state. Without including the cost of inserting old data, every append costs exactly $1$. In total, this has cost $n$.
>
> : Inserting old data costs the size of the old array. In total, this costs at most $n + \frac{n}{2} + ... \leq 2n$.
>
> : Thus, the total cost is at most $n+2n=3n$ and the average cost per execution is $3$. 

## Potentials

Potentials offer a more rigorous approach to amortized analysis. What we've been doing is still entirely correct, but might be problematically difficult with more complex algorithms.

### Intuition: Banker's Method

The intuitive way to think about potentials is with a bank. Every time $\mathcal{A}$ is executed, we earn some amount of coins $k(S)$ (function on data-structure state). If the current $\mathcal{A}$ operation is cheap, we can split our earnings between this operation and saving it in the bank. If the current $\mathcal{A}$ operation is expensive, we can use our funds from the bank to support it.

$k(S)$ is an amoritized time bound on $\mathcal{A}$ if we will always have enough coins to pay for an operation at any given time.

### Definition

The potential function $\Phi$ is a function mapping data-structure states to $\reals$. It represents the "potential" of that state (the coin cost).

Let $ac_i$ represent the amortized cost over $i$ algorithm executions. Let $c_i$ be the cost of the $i$th execution, and let $S_i$ being the resulting state.

$$ac_i = c_i + \Phi(S_i)-\Phi(S_{i-1})$$
Analogically, $\text{amortized cost}=\text{actual cost}+\text{change in potential}$.

$$\begin{align*}\sum_iac_i&=\sum_i(c_i+\Phi(S_i)-\Phi(S_{i-1}))\newline&=\sum_ic_i+\Phi(S_n)-\Phi(S_0)\end{align*}$$

{{<box important>}}Observe that if $\Phi(S_n)\geq\Phi(S_0)$ (which is frequently the case): $$\sum_ic_i\leq\sum_iac_i$$

Note that $\Phi(S)$ can be defined in any way, but unless $\Phi(S_0)-\Phi(S_n)$ is appropriately bounded (as in above), our determined $ac$ has no meaning.
{{</box>}}

## Revisiting Binary Counter
> **Theorem**
> : The amortized time cost is at most $2$.
>
> **Proof**
> : Let $\Phi(S)=1\text{s in }S$.
>
> : Consider the $i$th execution $i-1\rightarrow i$. Let $k$ be the number of carries that occur.
{{<box info>}}
In $10100\rightarrow 10101$, $\thickspace k=0$

In $10111\rightarrow 11000$, $\thickspace k=3$
{{</box>}}
> : $\Phi(S_i)-\Phi(S_{i-1})=-k+1$ since $k$ $1$s become $0$ and one $0$ becomes a $1$.
>
> : For the same reason, $c_i=k+1$.
{{<box info>}}
In $10100\rightarrow 10101$
* costs $1$
* changes potential from $2\rightarrow 3$

In $10111\rightarrow 11000$, $\thickspace k=3$
* costs $4$
* changes potential from $4\rightarrow 2$
{{</box>}}
> : Thus, $ac_i=(k+1)+(-k+1)=2$.
>
> : Since $\Phi(S_n)\geq\Phi(S_0)\thickspace\forall n$,
> : $$\sum_ic_i\leq\sum_iac_i$$
