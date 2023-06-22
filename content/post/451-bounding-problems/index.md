---
title: II. Bounding Problems
description: Determining algorithmic upper/lower bounds
slug: bounding-problems
date: 2023-06-09 00:00:00+0000
image: waterfall_rock.jpg
categories:
    - 15-451
    - algorithms
tags:
    - theoretical bound
    - sorting
    - merge sort
    - kth priority element
    - graph connectivity
    - information theory
    - adversarial argument
---

## Sorting

### Lower Bound (Info Theory Argument)

> **Theorem**
> : ***Any*** deterministic sorting algorithm on an array of length $n$ must make at least $\log_2(n!)$ comparisons
>
> **Proof**
> : Observe that sorting is just a permutation on the input array. In total, there are $n!$ permutations. In the worst case, only one permutation corresponds to a sorted array.
{{<box info>}}This is when all elements in the array are unique{{</box>}}
> : Observe that each comparison ($a {_?\atop >} b$) partitions the solution space into two: permutations that are still possibly correct and permutations that are not.
{{<box info>}}Suppose we have $[3, 1, 2]$. At the start of the algorithm (without having made any comparisons), all permutations could potentially be correct (sorted). However, once we make the comparison between $3 {?\atop >} 1 \Rightarrow 1 < 3$, every permutation where $3$ comes before $1$ is known to be incorrect.

$$\cancel{[3,1,2]}\newline\cancel{[3,2,1]}\newline[2,1,3]\newline\cancel{[2,3,1]}\newline[1,3,2]\newline[1,2,3]$$
{{</box>}}
> : Since in the worst case, the algorithm will mark the smaller partition as incorrect, it must be the case that each comparison shrinks the solution space by at most half. So in the worst case, it takes $\log_2(n!)$ comparisons.

This is an [information theoretic](https://en.wikipedia.org/wiki/Information_theory) argument since the proof is explaining that any algorithm requires $\log_2 n$ bits of information.

## Upper Bound

> $\texttt{MergeSort}(A)$
> * Split $A$ into two contiguous subarrays $L,R$ with (approximately) equal number of elements
> * $L'=\texttt{MergeSort}(L)$, $R'=\texttt{MergeSort}(R)$
> * Merge sorted $L'$ and $R'$ to create sorted $A'$
{{<box info>}}You can inductively reason that $L'$ and $R'$ are sorted.{{</box>}}
{{<box info>}}The merging is done linearly using two pointers on $L'$ and $R'$ and appending the lesser of the two values to $A'$ while incrementing that pointer (this is possible because $L'$ and $R'$ are both sorted){{</box>}}

The only comparisons done in $\texttt{MergeSort}$ are in the last step. Every comparison increments at least one pointer, which can happen at most $n-1$ times (See [this](https://www.geeksforgeeks.org/merge-two-sorted-arrays/) for a more thorough explanation on the last step).

Unrolling the recurrence:
$$\begin{align*}&\thickspace\underbrace{(n-1)+2(\frac{n}{2}-1)+4(\frac{n}{4}-1)+...}_{\log_2(n)}\newline=&\thickspace (n-1) + (n-2) + (n-4) + ... \newline=&\thickspace n\log_2 n - (n - 1) \newline <& \thickspace n\log_2 n\newline\in&\thickspace \mathcal{O}(n\log n)\end{align*}$$

Since $\log_2(n!)\in\Omega(n \log n)$, the bound is tight.
{{<box info>}}$$\begin{align*}\log_2 (n!) &= \log_2(n) + \log_2(n - 1) + \log_2(n - 2) + ... + \log_2(1) \newline &> \frac{n}{2}\log_2 (\frac{n}{2}) \newline &= \Omega(n \log n) \end{align*}$${{</box>}}

## Maximum Element

### Upper Bound

Scanning the array from left to right while keeping track of the largest seen element requires $n-1$ comparisons.

### Lower Bound

> **Theorem**
> : Any deterministic maximum element algorithm must make at least $n - 1$ comparisons 
>
> **Proof**
> : AFSOC there exists an algorithm $\mathcal{A}$ which makes less than $n-1$ comparisons. Construct a graph with $n$ vertices for each array element. For every comparison $\mathcal{A}$ makes between two elements, append an edge to the graph between the corresponding vertices. Since there are less than $n-1$ edges, there must be at least two islands.
>
> : If $\mathcal{A}$ selects an element $e$ in an island, consider the same input array except all corresponding elements on every other island are incremented by $e$. The observations $\mathcal{A}$ makes are the same since the comparisons known are only within an island, yet clearly $e$ cannot be the maximum element.
{{<box info>}}
Suppose our array is $[1, 2, 3, 4, 5, 6, 7]$ labelled $[a, b, c, d, e, f, g]$ and $\mathcal{A}$ makes the following ($n-2=5$) comparisons:
$$
a_{(1)}<c_{(3)}\newline
c_{(3)}<g_{(7)}\newline
g_{(7)}>d_{(4)}\newline
\text{}\newline
b_{(2)}<f_{(6)}\newline
f_{(6)}>e_{(5)}\newline
$$

```goat
    3                                 3
    o                                 o
   / \  +-+                          / \  +-+
1 o   o |7|                       1 o   o |7|
   \ /  +-+                          \ /  +-+
    o                                 o
    4            =========>           4


    5                                12        /
    o                                 o       /
   / \                               / \     v
2 o---o 6                         9 o---o 13
```
Now look at the comparisons after the incremental modification to $[1, 9, 3, 4, 5, 12, 13]$ labelled $[a, b, c, d, e, f, g]$:
$$
a_{(1)}<c_{(3)}\newline
c_{(3)}<g_{(7)}\newline
g_{(7)}>d_{(4)}\newline
\text{}\newline
b_{(9)}<f_{(13)}\newline
f_{(13)}>e_{(12)}\newline
$$
The comparisons are still true yet $\mathcal{A}$'s original output is incorrect!
{{</box>}}

## Second Largest Element

### Upper Bound

Find the maximum element with a "playoffs" structure ($n-1$ comparisons).

{{<box info>}}
```goat
6       4       2       1       8       7       3       5
|       |       |       |       |       |       |       |
 '-. .-'         '-. .-'         '-. .-'         '-. .-'
    |               |               |               |
    6               2               8               5
    |               |               |               |
     '-----. .-----'                 '-----. .-----'
            |                               |
            6                               8  
            |                               |
             '-------------. .-------------'
                            |
                            8
```
{{</box>}}

Observe that the second largest element is only less than the maximum element. Therefore, it must be the case that the second largest element had a comparison with the maximum element.

Since there are $\log_2 n$ "rounds", it takes $\log_2 n - 1$ comparisons to find the maximum element among those previously compared with the true maximum element.

Thus, this algorithm provides an upper bound of $n + \log_2 n - 2$ comparisons.

### Lower Bound (Adversarial Argument)

> **Theorem**
> : Any deterministic second largest element algorithm must make at least $n + \log_2 n - 2$ comparisons 
>
> **Proof**
> : Consider the set of all comparisons made that do not involve the maximum element. Since any algorithm $A$ finds the second largest element, the same argument from the previous problem can be made to prove that this set has at least $n-2$ elements.
>
> : Let $M$ be the set of all comparisons made that involve the maximum element. The lower bound on the number of comparisons made by $A$ is
> : $$n-2+|M|$$
>
> : We will now show that $|M|$ is at least $\log_2 n$.
> : Let each element have a corresponding weight associated with it. The weight $w$ represents the number of elements known to be less than it. Observe that with each comparison query $\mathcal{A}$ makes between any two elements $e_1,e_2$, one of $w(e_1) := w(e_1) + w(e_2)$ or $w(e_2) := w(e_2) + w(e_1)$ will occur.
>
> : Since either is possible before the comparison occurs, we can choose which is true upon $\mathcal{A}$'s query. Specifically, we will minimize the weight increment (if $w(e_1) > w(e_2)$ then assign $e_1>e_2$ else assign $e_1<e_2$).
>
> : Observe that with this dynamic response, any weight can at most double with each comparison. Since the algorithm only knows the maximum element once such an element obtains a weight of $n-1$ (there are $n-1$ elements less than the maximum one), $|M|\geq \log_2 n$.
>
> : $$n - 2 + \log_2 n$$

This is an ***adversarial*** argument since the proof constructs an adversary that dynamically responds to any algorithm's queries such that it minimizes the algorithm's effectiveness.

## Graph Connectivity

### Upper Bound

Query every pair to know the entire graph. In an $n$-node graph, this is $n\choose 2$ queries.

### Lower Bound (Adversarial Argument)

> **Theorem**
> : Any deterministic graph connectivity algorithm must make $n\choose 2$ queries
>
> **Proof**
> : We will again construct an adversary $\mathcal{E}$ to maximize the number of required queries an algorithm $\mathcal{A}$ must make.
>
> : Observe that the edges declared by $\mathcal{A}$ form a forest of trees (where $\mathcal{A}$ aims to determine if the number of trees in the forest is exactly $1$). $\mathcal{E}$ will maintain the following invariant.
> : * For each tree $T$, all possible edges among its vertices have been queried
> : * For each pair of trees $T_1, T_2$, $\exists (e_1,e_2)$ such that $e_1\in T_1, e_2\in T_2$. 
>
> : In other words, only dynamically create an edge if it is the last edge to connect two forests.
{{<box info>}}
Intuitively $\mathcal{E}$'s idea is that if it creates an edge between two trees $T_1, T_2$, but it is not the last possible edge query between $T_1, T_2$, then $\mathcal{A}$ would not need to 
query any more edges between $T_1, T_2$.
```goat

     T1             T2

                    d
  a o---o c   ?     o
     \      <--->   |                            
      o             o
      b             e
```
In the above example, there currently exists $0$ queries between trees $T_1$ and $T_2$. If $\mathcal{A}$ queries $(b, e)$ and $\mathcal{E}$ responds with ***yes***, then $\mathcal{A}$ has no need to query any more edges between $T_1$ and $T_2$ because it already knows that there is going to be some path in $T_1$ to $b$ which connects to some path in $T_2$ to $e$.
{{</box>}}
> : AFSOC $\mathcal{A}$ outputs without querying $n\choose 2$ edges from $\mathcal{E}$. By definition of $\mathcal{E}$, $\exists T_1, T_2, ..., T_t$ such that there is no queried connecting edge between $T_i, T_j\thickspace \forall i, j$.
>
> : * If $\mathcal{A}$ outputs ***yes***, then suppose all un-queried edges are not in the graph.
> : * If $\mathcal{A}$ outputs ***no***, then suppose all un-queried edges are indeed in the graph.
