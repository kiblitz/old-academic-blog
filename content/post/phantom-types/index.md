---
title: Phantom Types
description: Improving program correctness using the type system
slug: phantom-types
date: 2023-07-16 00:00:00+0000
image: ghost.jpeg
categories:
    - types
tags:
    - units
    - access control
---

## Introduction

As implied by the name, phantom types are defined by type parameters that are not actually used in implementation. Instead, they are used by the type system to restrict operations on that type. We can use them to improve program correctness (by the type checker).

## Examples

### Units

Suppose we want to have a type for measurement units.

#### Definition

$$
\begin{align*}
&\texttt{module Unit \: sig}\newline
&\texttt{\qquad type 'a t}\newline
&\texttt{\qquad val of\\_float \: float -> 'a t}\newline
&\texttt{\qquad val (+.) \: 'a t -> 'a t -> 'a t}\newline
&\texttt{end = struct}\newline
&\texttt{\qquad type 'a t = float}\newline
&\texttt{\qquad let of\\_float x = x}\newline
&\texttt{\qquad let (+.) = (+.)}\newline
&\texttt{end}
\end{align*}
$$

{{<box info>}}The module syntax is for defining stuff (in this case types and functions) in a context. So you would have to call $\texttt{of\\_float}$ with $\texttt{Unit.of\\_float}$.

$\texttt{sig}$ represents the module signature and $\texttt{struct}$ represents its actual definition (you can have multiple $\texttt{struct}$ definitions for a single $\texttt{sig}$).
{{</box>}}

{{<box info>}}$\texttt{(+.)}$ is just an infix operator for float addition. Here, we define it for operations involving two values of type $\texttt{Unit}$.{{</box>}}

Notice how $\texttt{'a}$ is never actually used: its underlying type is just a $\texttt{float}$. However, notice that $\texttt{(+.)}$ has type $\texttt{'a t -> 'a t -> 'a t}$. This means that the two parameters we pass into $\texttt{(+.)}$ better have the same $\texttt{'a}$ type parameter. Similarly, it will return a value with that same $\texttt{'a}$.

#### Use

$$
\begin{align*}
&\texttt{type meters}\newline
&\texttt{type lbs}\newline
&\newline
&\texttt{open Unit}\newline
&\texttt{let m1 \: meters t = of\\_float 2.}\newline
&\texttt{let m2 \: meters t = of\\_float 4.}\newline
&\texttt{let p1 \: lbs t = of\\_float 3.}\newline
&\texttt{let p2 \: lbs t = of\\_float 5.}\newline
&\texttt{let total\\_m = m1 +. m2}\newline
&\texttt{let total\\_p = p1 +. p2}
\end{align*}
$$

{{<box info>}}$\texttt{open Unit}$ just allows us to access $\texttt{Unit}$ things without prepending them.{{</box>}}

Notice how $\texttt{meter}$ and $\texttt{lbs}$ are never actually used outside of declaring types. This is how we can enforce that $\texttt{(+.)}$ operations can never be used on different units.


{{<box warning>}}If you try to add units with different $\texttt{'a}$ parameter types, your program will fail to type check.

$$\cancel{\texttt{total\\_m (+.) total\\_p}}$$

Also, if you try to instantiate a variable with a phantom type without the explicit $\texttt{'a}$, your program will also fail to type check.

$$\cancel{\texttt{let x = of\\_float 6.}}$$
{{</box>}}

### Access Control

This example was inspired by a [Ron Minsky post](https://blog.janestreet.com/howto-static-access-control-using-phantom-types/).

Suppose we want to have a type for reference cells that have access control permissions (read and read/write).

{{<box info>}}Reference cells allow for mutable values by storing them as addresses containing the data.

- $\texttt{let x = ref 10}$ creates a reference cell $\texttt{x}$ storing the value $\texttt{10}$.
- $\texttt{x := 4}$ sets the value in $\texttt{x}$ to $\texttt{4}$.
- $\texttt{let v = !x}$ extracts the value in $\texttt{x}$ (so $\texttt{v = 4}$).
{{</box>}}

#### Definition

$$
\begin{align*}
&\texttt{type read}\newline
&\texttt{type write}\newline
&\newline
&\texttt{module Ref \: sig}\newline
&\texttt{\qquad type ('a, 'b) t}\newline
&\texttt{\qquad create : 'b -> (write, 'b) t}\newline
&\texttt{\qquad set : (write, 'b) t -> 'b -> unit}\newline
&\texttt{\qquad get : ('a, 'b) t -> 'b}\newline
&\texttt{\qquad readonly : ('a, 'b) t -> (read, 'b) t}\newline
&\texttt{end = struct}\newline
&\texttt{\qquad type ('a, 'b) t = 'b ref}\newline
&\texttt{\qquad create x = ref x}\newline
&\texttt{\qquad set t x = t := x}\newline
&\texttt{\qquad get x = !x}\newline
&\texttt{\qquad readonly x = x}\newline
&\texttt{end}
\end{align*}
$$

$\texttt{'a}$ is discarded in the definition of $\texttt{Ref}$ (what makes it a phantom type). $\texttt{'b}$ just makes it polymorphic.

{{<box info>}}Polymorphic just means that the underlying type of the data within the cell can be anything. In the use example below, we will use $\texttt{'b = int}$.{{</box>}}

As shown, $\texttt{Ref}$ has $\texttt{'a = write}$ access upon creation, and you can only call $\texttt{set}$ on a $\texttt{Ref}$ which has $\texttt{'a = write}$. Additionally, you can cast any $\texttt{Ref}$ to have $\texttt{'a = read}$ using $\texttt{readonly}$ without actually changing any of its underlying data (just its $\texttt{'a}$ parameter type).

#### Use

$$
\begin{align*}
&\texttt{open Ref}\newline
&\texttt{let write\\_ref = create 10}\newline
&\texttt{let read\\_ref = readonly write\\_ref}\newline
&\texttt{let value1 = get write\\_ref}\newline
&\texttt{let () = set write\\_ref 4}\newline
&\texttt{let value2 = get read\\_ref}
\end{align*}
$$

{{<box warning>}}If you try to call $\texttt{set}$ on a $\texttt{Ref}$ with $\texttt{'a = readonly}$, your program will fail to type check.

$$\cancel{\texttt{set read\\_ref 4}}$$
{{</box>}}
