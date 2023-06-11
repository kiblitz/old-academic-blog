---
title: II. Quantum Computing Foundations
description: Basics to quantum programming
slug: quantum-computing-foundations
date: 2023-06-10 00:00:00+0000
image: mountain_winter.jpeg
categories:
    - 15-459
    - quantum
tags:
    - amplitude
    - logic gate
---

## Qubit System Representations

### Single Qubit Recap

Previously, we saw that a single qubit can be represented as a pair of amplitudes.
$$A=\begin{bmatrix} v_0 \newline v_1 \end{bmatrix}$$
$$\text{OR}$$
$$A=v_0|0\rangle + v_1|1\rangle$$
Where $v_0$ represents $A$'s amplitude on $0$ and $v_1$ represents $A$'s amplitude on $1$.

### Extending to Multi-Qubit States

In $q$-qubit systems, the representing vectors are $2^q$-dimensional where each component's magnitude represents the amplitude on a specific value. The mapping of axis to value follows from the tensor product order.
{{<box info>}}Suppose $AB$ had the following state.
$$AB=v_{00}|00\rangle + v_{01}|01\rangle + v_{10}|10\rangle + v_{11}|11\rangle$$
The corresponding vector would be the following.
$$AB=\begin{bmatrix} v_{00} \newline v_{01} \newline v_{10} \newline v_{11} \end{bmatrix}$$
{{</box>}}

## Classical Quantum Gates

With the exception of qubit initialization, quantum computing operations are all bijective modifications on their input qubits with no outputs. 
{{<box info>}}The reason for this strict rule is for [computational reversibility](https://en.wikipedia.org/wiki/Reversible_computing) which is essential for taking out the garbage (to be explained in a later post).

This is why qubit value assignment isn't possible. It is not possible to reverse the assigned value $o$ since every possible input $i$ maps to $o$.
{{</box>}}

### Definitions

#### Initialize

$\texttt{INIT}(A)$

Creates a new qubit $A$ with full amplitude on value $0$.

$$\begin{bmatrix} 1 \newline 0 \end{bmatrix}$$

#### Not

$\texttt{NOT}(A)$

Negates $A$. In other words, adds $1$ to qubit $A$ ($\bmod\thickspace2$).

$$\begin{bmatrix} 0 & 1 \newline 1 & 0 \end{bmatrix}$$

#### Controlled Not

$\texttt{CNOT}(AB)$

If $A=1$ then negates $B$. In other words, adds $A$ to qubit $B$ ($\bmod\thickspace2$).

$$\begin{bmatrix} 1 & 0 & 0 & 0 \newline 0 & 1 & 0 & 0 \newline 0 & 0 & 0 & 1 \newline 0 & 0 & 1 & 0\end{bmatrix}$$

{{<box info>}}
$$
\begin{array}{c|c}
   AB & \texttt{CNOT}(AB) \newline
   00 & 00 \newline
   01 & 01 \newline
   \boxed{10} & 11 \newline
   \boxed{11} & 10
\end{array}
$$
{{</box>}}

#### Controlled Controlled Not

$\texttt{CCNOT}(ABC)$

If $A=1$ and $B=1$ then negates $C$. In other words, adds $A\And B$ to qubit $C$ ($\bmod\thickspace 2$).

$$\begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 & 0 & 0 \newline
0 & 1 & 0 & 0 & 0 & 0 & 0 & 0 \newline
0 & 0 & 1 & 0 & 0 & 0 & 0 & 0 \newline
0 & 0 & 0 & 1 & 0 & 0 & 0 & 0 \newline
0 & 0 & 0 & 0 & 1 & 0 & 0 & 0 \newline
0 & 0 & 0 & 0 & 0 & 1 & 0 & 0 \newline
0 & 0 & 0 & 0 & 0 & 0 & 0 & 1 \newline
0 & 0 & 0 & 0 & 0 & 0 & 1 & 0
\end{bmatrix}$$

{{<box info>}}
$$
\begin{array}{c|c}
   ABC & \texttt{CCNOT}(ABC) \newline
   000 & 000 \newline
   001 & 001 \newline
   010 & 010 \newline
   011 & 011 \newline
   100 & 100 \newline
   101 & 101 \newline
   \boxed{110} & 111 \newline
   \boxed{111} & 110
\end{array}
$$
{{</box>}}



### Syntactic Sugar

Sometimes we will define/use a subroutine $F(X_1X_2...X_n)$ that outputs a value which we later execute in some pseudocode fashion (i.e. $\text{If }F(AB)\text{ Then}...$). But technically returning a value isn't allowed.

This is just syntactic sugar for creating a temporary qubit $T$ and applying $F$'s logic to modify $T$ as the output.
$$\texttt{INIT}(T)\newline ...\newline \text{// Apply $F$ logic onto $T$}\newline ... \newline\text{If }T\text{ Then}...$$

Also note that since quantum computation must be reversible, branching must be done intelligently.
For example, $\text{If}$ branches are syntactic sugar for controlled operations ($\texttt{CTRUEBRANCH}(T...)$).

{{<box info>}}
Example for negation of adding $\texttt{CCNOT}$ to input.
$$
\begin{array}{c|c}
   \text{compiled} & \text{syntactic sugar} \newline\newline
   \texttt{INIT}(A) & \texttt{INIT}(A) \newline
   \texttt{INIT}(B) & \texttt{INIT}(B) \newline
   \texttt{INIT}(C) & \texttt{INIT}(C) \newline
   \texttt{INIT}(D) & \texttt{INIT}(D) \newline
   \texttt{INIT}(T_1) &  \newline
   \texttt{INIT}(T_2) &  \newline
   & \texttt{def F}(X_1X_2)\lbrace \newline
   & \texttt{INIT}(T) \newline
   & \texttt{NOT}(X_1) \newline
   & \texttt{NOT}(X_2) \newline
   & \texttt{CCNOT}(X_1X_2T) \newline
   & \texttt{NOT}(X_1) \newline
   & \texttt{NOT}(X_2) \newline
   & \texttt{Output }T \newline
   & \rbrace\quad\quad\quad\quad\quad\quad \newline
   & \newline
   \texttt{NOT}(A) & \texttt{If F}(AB)\texttt{ Then}\lbrace \newline
   \texttt{NOT}(B) & \newline
   \texttt{CCNOT}(ABT_1) & \newline
   \texttt{NOT}(A) & \newline
   \texttt{NOT}(B) & \newline
   \texttt{CNOT}(T_1A)& \texttt{NOT}(A) \newline
   \texttt{CNOT}(T_1B)& \texttt{NOT}(B) \newline
   & \rbrace\quad\quad\quad\quad\quad\quad\quad \newline
   & \newline
   \texttt{NOT}(C) & \texttt{If F}(CD)\texttt{ Then}\lbrace \newline
   \texttt{NOT}(D) & \newline
   \texttt{CCNOT}(CDT_2) & \newline
   \texttt{NOT}(C) & \newline
   \texttt{NOT}(D) & \newline
   \texttt{CCNOT}(T_2CD)& \texttt{CNOT}(CD) \newline
   & \rbrace\quad\quad\quad\quad\quad\quad\quad \newline
\end{array}
$$
{{</box>}}
