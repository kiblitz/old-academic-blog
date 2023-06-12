---
title: III. Garbage Qubit Disposal
description: Taking out the quantum trash with reversible operations
slug: garbage-qubit-disposal
date: 2023-06-12 00:00:00+0000
image: mountain_contrast.jpg
categories:
    - 15-459
    - quantum
tags:
    - amplitude
    - reversible computing
    - temporary variables
---

## Motivation

### Example
Suppose we had the following quantum code.

$\texttt{INIT}(A)\newline\texttt{HAD}(A)\newline\texttt{HAD}(A)$
```goat
            0
            o
       0    |    1
       .---' '---.
      |           |
    0 o           o 1
     / \         / \
   1/   \1     1/   \-1
   /     \     /     \
  o       o   o       o
  0       1   0       1                                
```
We can see that the amplitudes on the paths to value $1$ cancel out, leaving $A=0$ with probability $1$.

What happens when we throw a random qubit into the program?

### Correctness Bug
$\texttt{INIT}(A,B)\newline\texttt{HAD}(A)\newline\texttt{CNOT}(AB)\newline\texttt{HAD}(A)$
```goat
            00
            o
       0    |    1
       .---' '---.
      |           |
   00 o           o 10
      |           |
     1|           |1
      |           |
   00 o           o 11
     / \         / \
   1/   \1     1/   \-1
   /     \     /     \
  o       o   o       o
 00      10   01      11                               
```
Now that $B$ has a value, amplitude cancellations don't work anymore. We don't modify $A$ at all with this change. Yet, $A=0$ and $A=1$ each have probability $0.5$.

This example is a bit contrived, but you can imagine that you might need to introduce extra temporary qubits and perform computations on them like you would temporary variables in classical computing. These types of qubits are called **garbage qubits**.

## Taking Out the Trash

So here's the fix. Just set all the garbage qubits to $0$. That way all leaves have the same garbage qubit values and interference works as if they didn't exist.

The catch? Remember that all quantum computation must be reversible. Assignment isn't reversible. We can't actually just set qubits to $0$.

### Reversible Computation

Instead, we can perform computations with our garbage qubits, use their outcomes, and then perform ***reversed computations*** on the garbage qubits.

#### Example

$\texttt{INIT}(A,B)\newline\texttt{HAD}(A)\newline\texttt{CNOT}(AB)\newline\boxed{\texttt{CNOT}(AB)}\newline\texttt{HAD}(A)$

Again, this example is a bit contrived. But the idea is still applicable. Once you are finished working with garbage qubits, you need to apply the inverse of the computations on those qubits so they can be reversed to $0$.

{{<box info>}}Reversal example

$
\texttt{CCNOT}(ABC)
\newline\texttt{NOT}(A)
\newline\texttt{CNOT}(AB)
\newline\quad\newline
\newline\texttt{CNOT}(AB)
\newline\texttt{NOT}(A)
\newline\texttt{CCNOT}(ABC)
$
{{</box>}}

#### Hadamard
Recall that Hadamard performs the following mapping on qubit values.
$$\begin{align*}0&\xmapsto{1}1\newline0&\xmapsto{1}0\end{align*}$$$$\begin{align*}1&\xmapsto{1}0\newline1&\xmapsto{-1}1\newline\end{align*}$$
Notice that if we flip the arrows, the operation is the same. Thus, $\texttt{HAD}$ is its own reverse.
{{<box info>}}This explains why executing $\texttt{HAD}$ twice has no effect{{</box>}}

{{<box important>}}Technically this reverse operation is called [dagger](https://en.wikipedia.org/wiki/Conjugate_transpose) (will be covered in a later post). {{</box>}}
