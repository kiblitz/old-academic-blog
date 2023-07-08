---
title: VI. Bias Busting
description: Deustz-Josza
slug: bias-busting
date: 2023-06-27 00:00:00+0000
image: mountain_stairs.jpg
categories:
    - 15-459
    - quantum
    - algorithms
tags:
    - bias
    - hadamard 
---

# Introduction
Rather than try to formulate the algorithm for bias busting, we will "stumble upon" it by trying to learn more about the Hadamard gate.

# Revisiting Hadamard

## Definition

Recall the Hadamard unnormalized mapping.

$$\begin{align*}0&\xmapsto{1}1\newline0&\xmapsto{1}0\end{align*}$$$$\begin{align*}1&\xmapsto{1}0\newline1&\xmapsto{-1}1\newline\end{align*}$$

So when operating on a qubit with value $0$, applying a Hadamard gate puts it in superposition (each state has equal amplitude).

Recall that Hadamard is its own dagger. Executing Hadamard twice undoes its effects.

## Operations on Superposition

What happens when we try to execute a function $\texttt{F}$ in between the Hadamards?

So the first Hadamard puts qubits into superposition, $\texttt{F}$ performs some operations on this superposition state, and the final Hadamard does something (?) on the qubits.

$\texttt{HAD}(ABC)\newline\texttt{F}(ABC)\newline\texttt{HAD}(ABC)$
