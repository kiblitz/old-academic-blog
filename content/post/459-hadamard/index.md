---
title: I. The Hadamard Gate 
description: An introduction to quantum computing
slug: the-hadamard-gate
date: 2023-06-07 00:00:00+0000
image: mountain_cross.jpeg
categories:
    - 15-459
    - quantum
tags:
    - amplitude
    - probability
---

## Probabilistic Computing Analogy
The first computing models were entirely deterministic. The idea of probabilistic computing brought about a single new instruction.
```goat
                   0.5
                 .-------
   +-----------+/
---+ coin flip |
   +-----------+\
                 '-------                                    
                   0.5
```
It provided computers with greater functionality and the ability to solve problems in novel ways (randomized/approximation algorithms) as well as introduce a new space of problems not possible before (i.e. [interactive proof systems](https://en.wikipedia.org/wiki/Interactive_proof_system)).


Quantum computing does something similar. Enter the Hadamard gate.
{{<box important>}}Technically more but for now just the Hadamard gate{{</box>}}

## Hadamard

The Hadamard gate is a modification operation on a single qubit.
{{<box info>}}We will go over what a qubit is later. For now, it is sufficient to think of a qubit as a bit{{</box>}}

### What does it do?

#### Testing
What happens if we execute the following?
{{< closable_alert style="success" >}}Refresh the page to see!{{< /closable_alert >}}

$$\texttt{INIT}(A)\newline\texttt{HAD}(A)$$
:---:
{{< raw_html >}}
<p id="demo1"></p>
<script>
        document.getElementById("demo1").innerHTML = "$A = " + Math.round(Math.random()) + "$";
</script>
{{< /raw_html >}}

Seems like a coin flip. What about if $A=1$?

$$\texttt{INIT}(A)\newline\texttt{NOT}(A)\newline\texttt{HAD}(A)$$
:---:
{{< raw_html >}}
<p id="demo2"></p>
<script>
        document.getElementById("demo2").innerHTML = "$A = " + Math.round(Math.random()) + "$";
</script>
{{< /raw_html >}}

Also a coin flip. Is quantum computing really just a computer that can coin flip?

---

Ok let's see what happens when we chain $\texttt{HAD}$ operations.

| $$\texttt{INIT}(A)\newline\texttt{HAD}(A)\newline\texttt{HAD}(A)$$ | $$\texttt{INIT}(A)\newline\texttt{NOT}(A)\newline\texttt{HAD}(A)\newline\texttt{HAD}(A)$$ |
| --- | --- |
| $$A=0$$ | $$A=1$$ |

???

#### The Qubit

Before we can explain what's going on, let's revisit the qubit. It looks like a bit and acts like a bit with non-quantum instructions. But is it really a bit?

In reality, a qubit can be defined by a pair of **amplitudes**. Amplitudes in quantum computing are like what probabilities are in probabilistic computing. After a coin flip, the random bit has a $0.5$ probability of being $0$ and a $0.5$ probability of being $1$. The Hadamard gate has a similar effect except with the qubit's amplitudes.

{{<box info>}}Quantum pairs well with linear algebra. A qubit can be represented as a 2-dimensional vector where its components are the qubit's amplitudes. In the example below, qubit $A$ has all of its amplitude on the $0$ state. $$A=\begin{bmatrix}1 \newline 0 \end{bmatrix}$$In quantum, we typically use [Bra-ket notation](https://en.wikipedia.org/wiki/Bra%E2%80%93ket_notation).$$A=1|0\rangle + 0|1\rangle = |0\rangle$${{</box>}}

So how does a qubit resolve its amplitude state? A qubit with a non-trivial state is said to be in [superposition](https://en.wikipedia.org/wiki/Quantum_superposition). The probability that a qubit measures to a value is its amplitude on that value ***squared***.

{{<box info>}}Suppose $A=\frac{\sqrt{2}}{2}|0\rangle + \frac{\sqrt{2}}{2}|1\rangle$. Then, $A$ has a $0.5$ probability to measure with value $0$ and likewise $1$. Observe that the sum of the squares of amplitudes must sum to $1$. Because of this strict ratio property, we can represent qubits with their unnormalized state.$$A=|0\rangle + |1\rangle$${{</box>}}

So how does it offer any more than probabilistic computing?

#### Hadamard Definition

In simple terms, the Hadamard gate has the following value mappings (amplitudes on arrows).
$$\begin{align*}0&\xmapsto{\frac{\sqrt{2}}{2}}1\newline0&\xmapsto{\frac{\sqrt{2}}{2}}0\end{align*}$$$$\begin{align*}1&\xmapsto{\frac{\sqrt{2}}{2}}0\newline1&\xmapsto{-\frac{\sqrt{2}}{2}}1\newline\end{align*}$$

{{<box info>}}It may be easier to think about the mappings in their unnormalized amplitudes$$\begin{align*}0&\xmapsto{1}1\newline0&\xmapsto{1}0\end{align*}$$$$\begin{align*}1&\xmapsto{1}0\newline1&\xmapsto{-1}1\newline\end{align*}$${{</box>}}

The Hadamard gate can be represented by the following matrix.$$H=\frac{\sqrt{2}}{2}\begin{bmatrix}1 & 1 \newline 1 & -1\end{bmatrix}$$

Represented as a pair of amplitude trees based on starting state:
```goat
     0           1
     o           o
   1/ \1       1/ \-1
   /   \       /   \
  o     o     o     o
  0     1     0     1                                       
```
{{<box info>}}The root of the tree represents the starting state and the leaves the end state. The edges along any path represent the amplitudes (unnormalized in this example).{{</box>}}

The interesting capability quantum offers over probabilistic computing is that qubits can have *negative amplitudes*.

#### Examples

$\texttt{INIT}(A)\newline\texttt{HAD}(A)$
```goat
     0
     o
   1/ \1
   /   \
  o     o
  0     1                                                   
```
$\mathbb{P}[A=0]=0.5\newline\mathbb{P}[A=1]=0.5$

---
$\texttt{INIT}(A)\newline\texttt{NOT}(A)\newline\texttt{HAD}(A)$
```goat
      0
      o
      |
     1|
      o 1
     / \
   1/   \-1
   /     \
  o       o
  0       1                                                 
```
$\mathbb{P}[A=0]=0.5\newline\mathbb{P}[A=1]=0.5$

---
This is consistent with our first two testing results where $\texttt{HAD}$ acted like a coin flip.

$\texttt{INIT}(A)\newline\texttt{HAD}(A)\newline\texttt{HAD}(A)$
```goat
          0
          o
      1   |   1
      .--' '--.
    0|         |1
     o         o
   1/ \1     1/ \-1
   /   \     /   \
  o     o   o     o
  0     1   0     1                                          
```
$\mathbb{A}[A=0]=1\times 1 + 1\times 1 = 2\newline \mathbb{A}[A=1]=1\times 1 + 1\times (-1) = 0$
{{<box important>}}Unnormalized{{</box>}}
$\mathbb{P}[A=0]=1$

Ok, so this explains our last testing result. Essentially, there are two paths resulting in $A=1$ whose amplitudes cancelled each other out. This is the unique factor in quantum computing.

{{<box info>}}This is called [destructive interference](https://en.wikipedia.org/wiki/Wave_interference){{</box>}}

## Concluding Remarks

### Quantum Computing Advantage

Notice that the state of qubit $A$ only resolves at the end. If we were to only look at the right subtree, we would not know that an amplitude cancellation occured. In other words, to understand the behavior of a qubit we need knowledge on the entire amplitude tree. This is why _simulating a quantum computer using a classical computer has **exponential** complexity_.

### Resolving Qubit Values

At what point do the amplitude calculations resolve to probabilities? In other words, at what point is the value of $A$ known? In our examples, it seems to resolve at the end of the program. In actuality, a qubit is known when it is [measured](https://en.wikipedia.org/wiki/Measurement_in_quantum_mechanics).

In our last example, if we had measured qubit $A$ at every step, then the value of $A$ at the end is equally $0$ or $1$ at every step (essentially just a series of coin flips).
