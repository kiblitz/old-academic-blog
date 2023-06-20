---
title: IV. Splay Trees
description: Self-balancing binary tree with persistent access memory
slug: 451-splay-trees
date: 2023-06-20 00:00:00+0000
image: waterfall_forest.jpg
categories:
    - 15-451
    - algorithms
    - data structures
tags:
    - splay tree
    - amortized analysis
---

## Introduction

### Motivation

Suppose you want to keep an ordered mapping of keys to values. If the contents are dynamic, a self-balancing tree (i.e. AVL, red-black, etc.) is desirable to allow for logarithmic modification and access operations.

What if repeated access are expected? Specifically, if a value is queried several times and is located at a leaf, then each operation has $\mathcal{O}(\log n)$ time complexity. But we already know the value after the first query. We should be caching accesses.

### Splay Tree

The splay tree is a self-balancing binary tree with the property that accesses are cached through its internal structure.

Operations on a splay tree are approximately the same as those of other self-balancing trees: $\mathcal{O}(\log n)$ amortized.

At a high level, any operation on a node $N$ restructures the tree so that $N$ becomes the new root. Thus, repeated queries have $\mathcal{O}(1)$ time complexity.

## Definition

### Rotations
There are three splay steps for moving $N$ to the root (in these cases, $x$ upwards). Each is constructed using rotations. Each of these three have a mirror version. **Zig** is specifically for when $x$ is the child of the root node.

#### Zig
When $y$ is the root.
```goat
    y      x
   o        o
  /   ===>   \
 o            o
x              y                               
```
#### Zig-zag
```goat
     z            z
    o            o            x
   /            /             o
y o    ===>    o x   ===>    / \
   \          /             o   o 
    o        o             y     z             
     x      y
```
#### Zig-zig
```goat
      z                     x
     o           y           o
    /            o            \
 y o    ===>    / \    ===>    o y
  /            o   o            \
 o            x     z            o
x                                 z            
```
{{<box info>}}
```goat
 ===>                                           
```
The arrows above signify ***rotations***.

---
```goat
---->                                          
```
The arrows below signify ***splay steps***.

---
Rotations make up splay steps, but for analysis purposes we only care about splay steps.
{{</box>}}
### Access
Whenever we want to access (i.e. find the mapped value of) $N$, we traverse to $N$ and then splay it to the root.
### Examples
```goat
            6 o                                
             /
          5 o
           /
        4 o
         /
      3 o
       /
    2 o
     /
  1 o
   /
0 o
```
$\texttt{splay}(0)$
```goat
               6 o                    
                /         6 o
             5 o           /          0 o
              /         5 o              \
           4 o           /              5 o
            /         0 o                / \
---->    3 o   ---->     \   ---->    3 o   o 6
          /             3 o            / \
       0 o               / \        1 o   o 4
          \           1 o   o 4        \
         1 o             \              o 2
            \             o 2
           2 o
```
$\texttt{splay}(3)$
```goat
              3 o                              
               / \
              /   \
             /     \
            /       \
---->    0 o       5 o
            \       / \
           1 o   4 o   o 6
              \
             2 o 
```

## Analysis
### Setup
#### Weights
Purely for the purposes of analysis, assign each node $x$ with weight $w(x)>0$.
#### Sizes
Let $T(x)$ denote the subtree rooted at node $x$.
$$s(x)=\sum_{y\in T(x)} w(y)$$
#### Rank
$$r(x)=\lfloor\log_2(s(x))\rfloor$$
{{<box info>}}Observe that in a rotation between nodes $x,y$, the sizes (and thus ranks) of ***only*** $x,y$ change. Thus, splay steps only alter the ranks of the nodes they involve.{{</box>}}
#### Potential
Let $T$ denote the entire tree state.
$$\Phi(T)=\sum_{x\in T}r(x)$$
#### Example
Let $w(\cdot)=1$. Sizes $s(x)$ on left and ranks $r(x)$ on right.
```goat
           9 o                         3 o
            /                           /
         8 o                         3 o
          / \                         / \
       6 o   o 1                   2 o   o 0   
        / \                         / \
     2 o   o 3                   1 o   o 1
      /   /                       /   /
   1 o   o 2                   0 o   o 1
          \                           \
           o 1                         o 0
```
$$\Phi(T)=11$$
{{<box info>}}Observe that when $w(\cdot)=1$, $\Phi(T)=\mathcal{O}(n)$ when $T$ is balanced, and $\Phi(T)=\mathcal{O}(n\log n)$ when $T$ is most unbalanced (long chain){{</box>}}
### Access Lemma
> **Lemma**
> : The amortized time cost for splaying node $x$ on tree $T$ with root $t$ is at most the following (regardless of $w$):
> : $$3(r(t)-r(x))+1$$
>
>{{<box info>}}
Recall:
$$ac_i = c_i + \Phi(S_i)-\Phi(S_{i-1})$$
{{</box>}}
>{{<box important>}}
This is a very tedious and dense proof. You'll be fine if you take my word for it that the lemma is true.
{{</box>}}
>
> **Proof**
> : Observe that if two siblings have the same rank $r$, then their sizes are at least $2^r$. Thus, their parent has size at least $2\cdot 2^r=2^{r+1}$. Thus, the parent node has rank at least $r+1$.
>
> : Conversely, observe that if a node $x$ and its parent have the same rank $r$, then the sibling of $x$ must have rank less than $r$.
>
> : Call this observation the **Rank Rule**.
> : ---
> : Call $T_i$ the tree state after splay step $i$, where $T_0=T$. Let $r_i(n)$ be the rank of node $n$ in $T_i$.
>
> : We define $ac_i'=c_i'+\Phi(T_i)-\Phi(T_{i-1})$, $c_i'$ is the cost of the $i$th splay step and $ac_i'$ is the amortized cost of the $i$th splay step. Observe that a splay with $k$ splay steps is consistent with global $ac$:
$$\begin{align*}ac&=\sum_iac_i'\newline&=\sum_ic_i'+(\Phi(T_k)-\Phi(T_{k-1}))+...+(\Phi(T_1)-\Phi(T))\newline&=c+\Phi(T_k)-\Phi(T)\end{align*}$$
>
> : For readibility purposes, if $i$ is fixed at a proof step, we will refer to $i$ as $\text{curr}$ and $i-1$ as $\text{prev}$.
>
> : We will show that if splay step $i$ is a zig step moving node $a$ upwards to $b$: $$ac_\text{curr}'\leq 3(r_\text{prev}(b)-r_\text{prev}(a))+1$$ We will also show that if splay step $i$ is either a zig-zag or zig-zig step: $$ac_\text{curr}'\leq 3(r_\text{prev}(b)-r_\text{prev}(a))$$
> : Since the zig step can occur at most once, we would be able to prove the problem statement.
> : $$ac=\sum_iac_i'\leq 1 + \underbrace{3(r(t) -r(x))}_{\text{telescopes}}$$
>
> : **Zig**
{{<box info>}}
```goat
    b      a
   o        o
  /   ===>   \
 o            o
a              b                               
```
{{</box>}}
> : This can only occur if $b$ is the root. Since the rank on the root is constant (sum of all weights is constant): $$r_\text{curr}(a)=r_\text{prev}(b)$$
>
> : Since the rank of a child is at most that of its parent: $$r_\text{curr}(b)\leq r_\text{curr}(a)=r_\text{prev}(b)$$
>
> : The root node still has rank $r_\text{prev}(b)$ (no change) and its child at most increases from $r_\text{prev}(a)$ to $r_\text{prev}(b)$.
> : $$\Phi(T_\text{curr})-\Phi(T_\text{prev})\leq r_\text{prev}(b)-r_\text{prev}(a)$$ 
> : $$ac \leq 1 + r_\text{prev}(b)-r_\text{prev}(a)\leq 3(r_\text{prev}(b)-r_\text{prev}(a))+1$$
>
> : **Zig-zag**
{{<box info>}}
```goat
     b            b
    o            o            a
   /            /             o
c o    ===>    o a   ===>    / \
   \          /             o   o 
    o        o             c     b             
     a      c
```
{{</box>}}
> : **Case $r_\text{prev}(a)=r_\text{prev}(b)$**
>
> : Since the rank of a node is lower bounded by the rank of its child, $r_\text{prev}(c)=r_\text{prev}(a)=r_\text{prev}(b)$.
>
> : Recall that since the sum of the weights of the subtree are the same, $r_\text{prev}(b)=r_\text{curr}(a)$. By the **Rank Rule**: $$r_\text{curr}(c)<r_\text{curr}(a) \text{ OR } r_\text{curr}(b)<r_\text{curr}(a)$$ Since $r(\cdot)$ is integral, $$r_\text{curr}(c)\leq r_\text{curr}(a) - 1 \text{ OR } r_\text{curr}(b)\leq r_\text{curr}(a) - 1$$
> : Thus,
> : $$\begin{align*}\Phi(T_\text{curr})-\Phi(T_\text{prev})&= (r_\text{curr}(a) + r_\text{curr}(b) + r_\text{curr}(c))-(r_\text{prev}(a) + r_\text{prev}(b) + r_\text{prev}(c))\newline &\leq r_\text{curr}(a) + (r_\text{curr}(a) + r_\text{curr}(a) - 1)-(r_\text{curr}(a) + r_\text{curr}(a) + r_\text{curr}(a))\newline &= -1\end{align*}$$ 
> : $$ac \leq 1 - 1 =0 \leq 3(r_\text{prev}(b)-r_\text{prev}(a))$$
> : ---
> : **Case $r_\text{prev}(a)<r_\text{prev}(b)$**
>
> : Since $r(\cdot)$ is integral and $r_\text{prev}(a)<r_\text{prev}(b)$,
>
> : $$r_\text{prev}(a)+1\leq r_\text{prev}(b)$$
>
> : As before, $r_\text{prev}(b)=r_\text{curr}(a)$, and all node ranks are upper bounded by those of their parents.
>
> : $$\begin{align*}\Phi(T_\text{curr})-\Phi(T_\text{prev})&= (r_\text{curr}(a) + r_\text{curr}(b) + r_\text{curr}(c))-(r_\text{prev}(a) + r_\text{prev}(b) + r_\text{prev}(c))\newline &\leq r_\text{prev}(b) + r_\text{prev}(b) + r_\text{prev}(b)-(r_\text{prev}(a) + (r_\text{prev}(a)+1) + r_\text{prev}(a))\newline &= 3(r_\text{prev}(b)-r_\text{prev}(a))-1\end{align*}$$ 
> : $$ac \leq 1 + 3(r_\text{prev}(b)-r_\text{prev}(a))- 1 = 3(r_\text{prev}(b)-r_\text{prev}(a))$$
> : **Zig-zig**
{{<box info>}}
```goat
      b                     a
     o           c           o
    /            o            \
 c o    ===>    / \    ===>    o c
  /            o   o            \
 o            a     b            o
a                                 b            
```
{{</box>}}
> : **Case $r_\text{prev}(a)=r_\text{prev}(b)$**
>
> : Since the rank of a node is lower bounded by the rank of its child, $r_\text{prev}(c)=r_\text{prev}(a)=r_\text{prev}(b)$.
>
> : As before, $r_\text{prev}(b)=r_\text{curr}(a)$, and all node ranks are upper bounded by those of their parents.
>
> : Let $r_\text{inter}(\cdot)$ indicate node ranks in the intermediate rotation (rooted at $c$). Observe that $r_\text{prev}(a)=r_\text{inter}(a)$ and $r_\text{inter}(b)=r_\text{curr}(b)$ since their children remain the same. By the **Rank Rule**, $r_\text{inter}(b) < r_\text{inter}(c)=r_\text{inter}(a)$. Because $r(\cdot)$ is integral, $$r_\text{curr}(b)\leq r_\text{curr}(a)-1$$
> : Thus,
>
> : $$\begin{align*}\Phi(T_\text{curr})-\Phi(T_\text{prev})&= (r_\text{curr}(a) + r_\text{curr}(b) + r_\text{curr}(c))-(r_\text{prev}(a) + r_\text{prev}(b) + r_\text{prev}(c))\newline &\leq r_\text{curr}(a) + (r_\text{curr}(a)-1) + r_\text{curr}(a) - (r_\text{curr}(a) + r_\text{curr}(a) + r_\text{curr}(a))\newline &= -1\end{align*}$$ 
> : $$ac \leq 1 - 1 =0 \leq 3(r_\text{prev}(b)-r_\text{prev}(a))$$
> : ---
> : **Case $r_\text{prev}(a)<r_\text{prev}(b)$**
>
> : Since $r(\cdot)$ is integral and $r_\text{prev}(a)<r_\text{prev}(b)$,
>
> $$r_\text{prev}(a)+1\leq r_\text{prev}(b)$$
>
> : As before, $r_\text{prev}(b)=r_\text{curr}(a)$, and all node ranks are upper bounded by those of their parents.
>
> : $$\begin{align*}\Phi(T_\text{curr})-\Phi(T_\text{prev})&= (r_\text{curr}(a) + r_\text{curr}(b) + r_\text{curr}(c))-(r_\text{prev}(a) + r_\text{prev}(b) + r_\text{prev}(c))\newline &\leq r_\text{prev}(b) + r_\text{prev}(b) + r_\text{prev}(b)-(r_\text{prev}(a) + (r_\text{prev}(a)+1) + r_\text{prev}(a))\newline &= 3(r_\text{prev}(b)-r_\text{prev}(a))-1\end{align*}$$ 
> : $$ac \leq 1 + 3(r_\text{prev}(b)-r_\text{prev}(a))- 1 = 3(r_\text{prev}(b)-r_\text{prev}(a))$$

### Balance Theorem
> **Theorem**
> : A sequence of $k$ splays in a tree of $n$ nodes has time complexity $$\mathcal{O}(k\log n + n\log n)$$
>
> **Proof**
> : Set $w(\cdot)=1$. By the **Access Lemma**,
> : $$\begin{align*}ac_i &= c_i + \Phi(T_i)-\Phi(T_{i-1}) \newline&\leq 3(r(t)-r(x) + 1\newline&\leq 3(\log_2(n)-0)+1\newline&\leq 3\log_2(n)+1\end{align*}$$
> : Thus,
> : $$\sum_i ac_i = \sum_i c_i + \Phi(T_m)-\Phi(T_{0}) \leq k (3\log_2 n + 1)$$
> : Since $\log(\cdot)>0$ and $r(t)\leq\log_2 n$, $$0\leq \Phi(T)\leq n\log_2 n$$
> : Thus,
> : $$\begin{align*}\sum_i c_i -n \log_2 n &\leq k (3\log_2 n + 1)\newline\sum_i c_i &\leq n \log_2 n + k (3\log_2 n + 1)\newline&\in\mathcal{O}(k\log n + n\log n)\end{align*}$$

## Operations
### Access
As we mentioned before, access searches for a node $x$ then splays it if found. Since binary search has time complexity $\mathcal{O}(\log n)$, the work is dominated by the splaying.
### Insert
First, traverse $T$ as if to search for the node $x$. Once we reach a node $n$ and cannot traverse further (i.e. $\texttt{insert}(3)$ but the $2$ node has no right child), splay $n$.

Now, we can insert $x$ appropriately into $T$.
```goat
                      x                x
    n                 o                o
    o                / \              / \
   / \    ===>    n o   o     or     o   o n
  o   o            /     R          L     \
 L     R          o                        o
                 L                          R  
```
Again, the search and insertion ($\mathcal{O}(1)$) is dominated by the splaying.
### Delete
First, splay the node we want to delete $x$. Now consider its children subtrees $L,R$. After removing $x$, splay the right-most node in $L$. Clearly, this new $L'$ has no right child. Set $R$ to be its new right child.
```goat
    x                               L'
    o             L   R             o
   / \    ===>    o   o    ===>    / \
  o   o          / \ / \          o   o
 L     R                         / \   R       
```
As before, the search is dominated by the two splaying operations.

## Above and Beyond
### Static Optimality
> **Theorem**
> : Let T be any static search tree with $n$ nodes. Let $t$ be the cost of searching for all nodes in a sequence of $s$ accesses (sum of depths of all nodes). The cost of splaying that sequence of requests, starting with any initial splay tree is $\mathcal{O}(n^2+t)$.

{{<box info>}}Can be proved with $w(x)=\text{number of times $x$ is accessed)}${{</box>}}
For a static tree $T$, $n$ is constant. This is powerful since essentially, splay trees perform only a constant sum ($n^2$) of work worse than the most optimal tree for $s$.

### Sequential Access
> **Theorem**
> : The cost of accessing each of the $n$ nodes in a tree in in-order order is $\mathcal{O}(n)$


