---
title: "I. Derandomization: Kth Smallest Element"
description: Turning QuickSelect to DeterministicSelect
slug: derandomization-kth-smallest-element
date: 2023-06-04 00:00:00+0000
image: waterfall_fire.jpg
categories:
    - 15-451
    - algorithms
tags:
    - search
    - las vegas
    - derandomization
---

## Problem
> Find the $k\text{th}$ smallest element in an unsorted array $A$ of size $n$.

Sorting has $\mathcal{O}(n\log n)$ time complexity and is overkill for this specific problem (solves for all $k$).

---
### QuickSelect
#### Algorithm
> $\texttt{QuickSelect}(A, k)$
> * Arbitrarily pick a pivot element $p$ from $A$
> * Split $A$ into $L = \lbrace a | \thickspace a \in A \text{ and } a < p \rbrace$ and $G = \lbrace a | \thickspace a \in A \text{ and } a > p \rbrace$
> * Recurse
>   * If $|L| = k$ then return $p$
>   * If $|L| > k - 1$ then $\texttt{QuickSelect}(L, k)$
>   * If $|L| < k - 1$ then $\texttt{QuickSelect}(G, k - (|L| + 1))$

{{< box info >}}The $A_k\in G$ case has to readjust the recursive $k$ value since we are essentially throwing away the first $|L| + 1$ elements ($L$ and $p$).{{< /box >}}

---
#### Example

_Suppose we always pick the first element $A_0$ to be $p$ (this is arbitrary for arbitrary $A$)._

{{< box warning >}}$A$ can adversarially be monotonically ordered and the time complexity becomes $\mathcal{O}(n^2)${{< /box >}}

```goat
             +---+---+---+---+---+---+----+---+----+---+----+---+
k = 8,   A = | 7 | 5 | 1 | 4 | 6 | 3 | 11 | 8 | 12 | 2 | 10 | 9 |
             +---+-.-+-.-+-.-+-.-+-.-+--.-+-.-+--.-+-.-+--.-+-.-+
                  /   /   /   /   /      \  |     \  |    |   |
                 /   /   /   /   /        \  '----.\ |     \   \
                /   /   /   /   / .--------+-------++       \   \
               /   /   /   /   / |          \      | \       |   |
              v   v   v   v   v  v           v     v  v      v   v
           +---+---+---+---+---+---+        +----+---+----+----+---+
p = 7, L = | 5 | 1 | 4 | 6 | 3 | 2 |,   G = | 11 | 8 | 12 | 10 | 9 |
           +---+---+---+---+---+---+        +----+---+----+----+---+
                                                                                                              

                    ----------------------------


             +----+---+----+----+---+
k = 1,   A = | 11 | 8 | 12 | 10 | 9 |
             +----+-.-+-.--+--.-+-.-+
                   /     \___/___/ 
                  /    _____/   /\ 
                 /    /   _____/  ' 
                /    /   /        |
               v    v   v         v
            +---+----+---+      +----+
p = 11, L = | 8 | 10 | 9 |, G = | 12 |
            +---+----+---+      +----+


                    ----------------------------


           +---+----+---+
k = 1, A = | 8 | 10 | 9 |
           +---+-.--+-.-+
                  \    \
                   \    \
                    v    v
                   +----+---+
p = 8, L = {}, G = | 10 | 9 |
                   +----+---+


                    ----------------------------


return 8
```
---
#### Analysis

> **Theorem**
>
> : The expected number of comparisons is upper bounded by $4n$
>
> **Proof**
>
> : Let $C(A, n, k)$ be the cost (number of comparisons) to find the $k\text{th}$ smallest element in an array $A$ of $n$ elements. Let $C(n) = \max_{A,k} C(A,n,k)$.
> {{< box info >}}Convince yourself that $C(A,n,k)$ doesn't depend on $A${{< /box >}}
> : It takes $n - 1$ comparisons to split $A$ into $L$ and $G$. The sizes of the pieces take the same distribution as $p$, which is uniformly random. By our definition of $C$, we will always consider the larger of the two pieces in cost calculation.
>
> $$\mathbb{E}[C(n)] \leq (n-1) + \text{avg}[C(\frac{n}{2}),...,C(n-1)]$$
>
> : Inductively, assume $\mathbb{E}[C(i)] \leq 4i$ for $i < n$ ($0$ comparisons when $n=1$ so the base case holds).
> $$\begin{align*}\mathbb{E}[C(n)] &\leq (n-1) + \text{avg}[4(\frac{n}{2}),...,4(n-1)] \newline &\leq (n-1) + 4(\frac{3n}{4}) \newline &\leq 4n\end{align*}$$
> {{<box important>}}Technically we assumed $n$ is even, but the proof is basically the same in the odd case{{</box>}}

Thus, QuickSelect has $\mathcal{O}(n)$ time complexity.

---
### DeterministicSelect
#### Algorithm
{{<box info>}}Pick a parameter constant $\alpha$. It matters what $\alpha$ is, but for now suppose $\alpha=5${{</box>}}
> $\texttt{ApproxMedian}(A, \alpha=5)$
> * If $|A| = 1$ then return $A_0$
> * Partition $A$ into groups $G_1, G_2, ... G_{\frac{n}{\alpha}}$ each of size $\alpha$
> * Let $M = \lbrace \text{median}(G_i) \forall i \rbrace$
{{<box info>}}It doesn't matter how median is calculated here since $\alpha$ is constant implies each calculation has $\mathcal{O}(1)$ time complexity ($\mathcal{O}(n)$ total) {{</box>}}
> * return $\texttt{DeterministicSelect}(M, \lceil\frac{|M|}{2}\rceil)$
{{<box info>}}In other words, find the true median of the group medians{{</box>}}
>
> {{<box important>}}There are some integrality issues (that we will ignore) which complicate the analysis, though the time complexity should be unaffected{{</box>}}
> $\texttt{DeterministicSelect}(A, k)$
> * If $|A| \leq \alpha$ then return $k\text{th}$ smallest value in $A$ by brute force
> * Let the pivot $p$ be $\texttt{ApproxMedian}(A)$
> * Split $A$ into $L = \lbrace a | \thickspace a \in A \text{ and } a < p \rbrace$ and $G = \lbrace a | \thickspace a \in A \text{ and } a > p \rbrace$
> * Recurse
>   * If $|L| = k$ then return $p$
>   * If $|L| > k - 1$ then $\texttt{DeterministicSelect}(L, k)$
>   * If $|L| < k - 1$ then $\texttt{DeterministicSelect}(G, k - (|L| + 1))$

{{<box info>}}Notice that the only difference between $\texttt{QuickSelect}$ and $\texttt{DeterministicSelect}$ is how the pivot $p$ is chosen{{</box>}}

---
#### ApproxMedian Example
$\alpha=3$
```goat
    +---+---+---+---+---+---+---+---+---+----+----+----+----+----+----+
A = | 7 | 5 | 1 | 4 | 6 | 3 | 8 | 2 | 9 | 11 | 14 | 15 | 10 | 12 | 13 |
    +---+---+---+---+---+---+---+---+---+----+----+----+----+----+----+

+---+---+---+  +---+---+---+  +---+---+---+  +----+----+----+  +----+----+----+
| 7 | 5 | 1 |  | 4 | 6 | 3 |  | 8 | 2 | 9 |  | 11 | 14 | 15 |  | 10 | 12 | 13 |
+---+-.-+---+  +-.-+---+---+  +-.-+---+---+  +----+-.--+----+  +----+-.--+----+
      |          |              |                   |                 |
      |    .----'   .-----------+------------------'                  |
      |   |    .---+-----------'                                      |
      |   |   |    |                                                  |
      v   v   v    v                                                  |
    +---+---+---+----+----+                                           |
M = | 5 | 4 | 8 | 14 | 12 |<-----------------------------------------'
    +---+---+---+----+----+

                                                                                                              
        DeterministicSelect


                   +---+---+---+----+----+
        k = 2, A = | 5 | 4 | 8 | 14 | 12 |
                   +---+---+---+----+----+


        return 5


return 5
```
---
#### Analysis
> **ApproxMedian Bound Lemma**
>
> : Let $p = \texttt{ApproxMedian}(A)$. Let $n = \lceil\frac{n}{2\alpha}\rceil\lceil\frac{\alpha}{2}\rceil$. At least $n$ elements in $A$ are greater than $p$ and at least $n$ elements in $A$ are less than $p$.
>
> **Proof**
>
> : WLOG look at the $\leq$ case. By definition, $p$ is the true median of the group medians, so $\lceil\frac{n}{2\alpha}\rceil$ groups (including the group $p$ is in) have medians $\leq p$.
>
> : For each of those groups, there are $\lceil\frac{\alpha}{2}\rceil$ elements (including that group's median) $\leq$ that group's median and therefore $p$.
>
> **Visual**
> ```goat
> +---+---+---+---+---+---+---+---+---+----+----+----+----+----+----+
> | 7 | 5 | 1 | 4 | 6 | 3 | 8 | 2 | 9 | 11 | 14 | 15 | 10 | 12 | 13 |
> +---+---+---+---+---+---+---+---+---+----+----+----+----+----+----+
>                                                                                                             
>
> +---+---+---+  +---+---+---+  +---+---+---+  +----+----+----+  +----+----+----+
> | 7 | 5 | 1 |  | 4 | 6 | 3 |  | 8 | 2 | 9 |  | 11 | 14 | 15 |  | 10 | 12 | 13 |
> +---+-.-+---+  +---+-.-+---+  +---+-.-+---+  +----+-.--+----+  +----+-.--+----+
>       |              |              |               |                 |
>        '---.         |            .-+---------------+----------------'
>    .--------+-------'  .---------+-'         .-----'
>   |         |         |          |          |
>   v         v         v          v          v
> +---+     +---+     +---+     +----+     +----+
> | 3 |     | 1 |     | 2 |     | 10 |     | 11 |
> +---+     +---+     +---+     +----+     +----+
> | 4 |     | 5 |     | 8 |     | 12 |     | 14 |
> +---+     +---+     +---+     +----+     +----+
> | 6 |     | 7 |     | 9 |     | 13 |     | 15 |
> +---+     +---+     +---+     +----+     +----+
>
>
> +---+---+---+
> | 3 | 1 | 2 |
> +---+---+---+   +----+----+
> | 4 | 5 | 8 |   | 10 | 11 |
> +---+---+---+   +----+----+
>                 | 12 | 14 |
>     +---+---+---+----+----+
>     | 6 | 7 | 9 | 13 | 15 |
>     +---+---+---+----+----+
> ```
>{{<box tip>}}$$\texttt{ApproxMedian}(A)=8$$ $$\begin{align}8 &\geq 5\geq 4 \tag*{medians} \newline 8&\geq 2 \tag*{group 3} \newline 5&\geq 1 \tag*{group 2} \newline 4&\geq 3 \tag*{group 1} \newline 8&\geq i \thickspace \forall i\in\lbrace 3, 4, 1, 5, 2, 8 \rbrace \tag*{$\therefore$} \end{align}$${{</box>}}
>
> ---
> **Theorem**
>
> : If $\alpha$ is chosen intelligently, $\texttt{DeterministicSelect}$ makes $\mathcal{O}(n)$ comparisons
>
> **Proof**
>
> : Let $C(A, n, k)$ be the worst-case cost to find the $k\text{th}$ smallest element in an array $A$ of $n$ elements and let $C(n) = \max_{A,k} C(A,n,k)$.
>
> : * Finding the median of a single group has constant cost (since its size is bounded by $\alpha$). Since there are a linear number of groups, this step has linear time complexity.
>
> : * As in $\texttt{QuickSelect}$ it also take linear time to split $A$ into $L$ and $G$.
>
> : * The first recursive call to find the true median of medians has time complexity $C(\frac{n}{\alpha})$.
{{<box info>}}Since there are that many number of medians{{</box>}}
> : * The second recursive call to search from a sub-piece has time complexity $C(n - \lceil\frac{n}{2\alpha}\rceil\lceil\frac{\alpha}{2}\rceil)$ by the **ApproxMedian Bound Lemma**.
{{<box info>}}Since in worst case we recurse on the larger sub-piece{{</box>}}
>
> $$C(n) \leq \gamma n + C(\frac{n}{\alpha}) + C(n - \lceil\frac{n}{2\alpha}\rceil\lceil\frac{\alpha}{2}\rceil)$$
{{<box info>}}$\gamma$ is a constant where its subexpression represents linear time complexity from the first two bullet points{{</box>}}
> : Using $\alpha=5$
> $$\begin{align*}C(n)&\leq \gamma n + C(\frac{n}{5}) + C(n - 3\lceil\frac{n}{10}\rceil)\newline&\leq \gamma n + C(\frac{n}{5}) + C(\frac{7n}{10})\newline&\leq \gamma n[1 + (\frac{1}{5} + \frac{7}{10}) + (\frac{1}{5}(\frac{1}{5} + \frac{7}{10}) + \frac{7}{10}(\frac{1}{5} + \frac{7}{10})) + ... ] \newline &\leq \gamma n[1 + (\frac{9}{10}) + (\frac{1}{5}\cdot\frac{9}{10} + \frac{7}{10}\cdot\frac{9}{10}) + ...] \newline &\leq \gamma n[1 + (\frac{9}{10}) + (\frac{9}{10})^2 + (\frac{9}{10})^3 + ...] \newline &\leq 10\gamma n \newline &\in \mathcal{O}(n)\end{align*}$$
>
> **Corollary**
> : $\texttt{DeterministicSelect}$ has linear time complexity if $$\begin{align*}\frac{1}{\alpha} + 1 - \frac{1}{2\alpha}\lceil\frac{\alpha}{2}\rceil &< 1\newline\frac{1}{\alpha} - \frac{1}{2\alpha}\lceil\frac{\alpha}{2}\rceil &< 0\end{align*}$$
