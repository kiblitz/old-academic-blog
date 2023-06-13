---
title: IV. Minus World
description: Entering the minus quantum realm
slug: minus-world
date: 2023-06-13 00:00:00+0000
image: mountain_colorful_horizon.jpg
categories:
    - 15-459
    - quantum
tags:
    - amplitude
---

## Problem

We know that the unique edge in quantum computing is derived from negative amplitudes. How can we cause a qubit value to have negative amplitude?

## Solution
### Working with the Hadamard Gate

The only instruction that we've seen that introduces negative amplitude is the hadamard gate. Specifically, $1\xmapsto{-1} 1$.

$$H=\frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix}$$

So if we want to enter minus world, we probably want to perform $\texttt{HAD}$ on a qubit with value $1$.

$$\frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix}\cdot\begin{bmatrix}0 \newline 1\end{bmatrix} = \frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}$$

Recall that $\texttt{HAD}$ is its own reverse ($\texttt{HAD}^\dagger=\texttt{HAD}\implies H^2=I$).
$$\begin{align*}(\frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix})^\dagger\cdot \frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix}\cdot\begin{bmatrix}0 \newline 1\end{bmatrix} &= (\frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix})^\dagger\cdot\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}\newline \begin{bmatrix}0 \newline 1\end{bmatrix} &= \frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix}\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}\end{align*}$$

### The Magic

The left side of the above equation shows a qubit with amplitude $1$ on value $1$. Looking at the computations in matrix form, it is clear that what we want is to somehow negate both sides of the equation.

Here is a simple equality.

$$-\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}=\frac{\sqrt{2}}{2}\begin{bmatrix}-1  \newline 1\end{bmatrix}$$

Now the solution is clear.
$$\texttt{NOT}(\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix})=\frac{\sqrt{2}}{2}\begin{bmatrix}-1  \newline 1\end{bmatrix}=-\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}$$

### Putting it All Together
The first step, we applied $\texttt{HAD}$ on a qubit with amplitude $1$ on value $1$ to get the following state.
$$\frac{\sqrt{2}}{2}\begin{bmatrix}1  \newline -1\end{bmatrix}$$
In the next step, we negated this expression with a $\texttt{NOT}$ operation.
$$\frac{\sqrt{2}}{2}\begin{bmatrix}-1  \newline 1\end{bmatrix}$$
Finally, we applied $\texttt{HAD}$ on this updated qubit.
$$\begin{bmatrix}0  \newline -1\end{bmatrix}$$

{{<box info>}}Notice that we require in the first step for the qubit to have value $1$. If it had value $0$, the steps would be the same but without the minus sign. In other words, no effect would take place.

This is powerful, because now we have a $\texttt{If }A\texttt{=1 THEN MINUS}(A)$ subroutine.
{{</box>}}

## Quantum Code
$
\texttt{INIT}(A)
\newline\texttt{NOT}(A)
\newline\quad\newline
\newline\texttt{HAD}(A)
\newline\texttt{NOT}(A)
\newline\texttt{HAD}(A)
$
```goat
                                .-----o 0                   
                               / 1
                    0   1   1 /
              1 .---o--------o
               /              \
              /                \-1
           1 /                  '-----o 1
A 0 o-------o HAD(A)  NOT(A)  HAD(A)
        1    \                  .-----o 0
              \                / 1
      NOT(A)   \              /
             -1 '---o--------o
                    1   1   0 \
                               \ 1
                                '-----o 1
```
$
\mathbb{A}[0] = (1\times 1\times 1\times 1) + (1\times -1\times 1\times 1) = 0\newline
\mathbb{A}[1] = (1\times 1\times 1\times -1) + (1\times -1\times 1\times 1) = -2
$
{{<box info>}}Unnormalized. Clearly when normalized, $\mathbb{A}[1]=-1$.{{</box>}}

## New Power
Suppose we created some subroutine $\texttt{MAJ}(ABCM)$ which negates $M$ if the majority value in $ABC$ is $1$ (we will use it with $M=0$, so essentially this function just sets $M$ to the majority value).

Now, with our new code, the following subroutine is possible.

$
\texttt{INIT}(ABCM)
\newline\texttt{// assign }ABC
\newline\texttt{MAJ}(ABCM)
\newline\texttt{HAD}(M)
\newline\texttt{NOT}(M)
\newline\texttt{HAD}(M)
$

Going forward, we can just execute subroutines like this with $\texttt{IF MAJ}(ABCM)\texttt{ THEN MINUS}(M)$.
