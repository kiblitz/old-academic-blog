---
title: "Type Checking: Subtyping"
description: What can be assigned to a variable of type A?
slug: subtyping
date: 2023-07-09 00:00:00+0000
image: purple.jpg
categories:
    - type theory
    - types
tags:
    - type checking
    - subtyping
---

## Introduction

At the very least when it comes to types, something of type $\texttt{A}$ can be used whenever something of type $\texttt{A}$ is expected.

$\texttt{int a = 5}\newline\texttt{float b = 5.0}$

Is there any leeway for flexibility? Specifically, does there exist a notion of type substitutability?

---

$\texttt{float b = 5}$

This makes sense because an $\texttt{int}$ is a subset of a $\texttt{float}$. In other words, a $\texttt{float}$ can do anything an $\texttt{int}$ can do.

$\cancel{\texttt{int b = 5.0}}$

This won't work. An $\texttt{int}$ cannot do everything a $\texttt{float}$ can do (cannot represent all of its values). A quick way to break this is: $$\texttt{int b = 5.5}$$

## Definition

$S <: T$ means that $S$ is a subtype of $T$.

This means that anything of type $S$ can be used whenever type $T$ is expected.

So in the above example:
$$\texttt{int}<:\texttt{float}$$

{{<box tip>}}You can think of it as $S$ is *more* restrictive **(to typecheck)** than $T$ since if you want to do something with $T$, you should also be able to do it with $S$.{{</box>}}

## Structures

### Width Subtyping

#### Class Hierarchy

Suppose a class lineage has the following structures.

$$
\begin{align*}
&\texttt{class Animal:}\newline
&\texttt{\qquad string name}\newline
&\texttt{\qquad int age}\newline
&\texttt{\qquad void}\rightarrow\texttt{void makeNoise}\newline
&\newline
&\texttt{class Mammal extends Animal:}\newline
&\texttt{\qquad string furColor}\newline
&\newline
&\texttt{class Human extends Mammal:}\newline
&\texttt{\qquad string occupation}\newline
&\texttt{\qquad int netWorth}\newline
&\texttt{\qquad void}\rightarrow\texttt{int doTaxes}
\end{align*}
$$

By definition of class hierarchy, a child has every characteristic its parent has. For example, $\texttt{Human}$ has a $\texttt{furColor}$, and transitively, since $\texttt{Mammal}$ has a $\texttt{name}$ (among other things), so does $\texttt{Human}$.

Since children have everything their parents have (and thus can do anything the parents can do):
$$\texttt{Human} <: \texttt{Mammal}$$
$$\texttt{Mammal} <: \texttt{Animal}$$
$$- \textit{and transitively } -$$
$$\texttt{Human} <: \texttt{Animal}$$

Generally:
> $$\texttt{Child} <: \texttt{Parent}$$

{{<box info>}}This is called **width subtyping** since supertypes (opposite of subtypes) contain a subset of fields (along the width of the class definition).{{</box>}}

#### Unrelated structures

The notion of width subtyping can be extended to datatypes without hierarchical relationships.

For example, suppose there are two unrelated datatypes with the following definitions.

$$
\begin{align*}
&\texttt{struct Named:}\newline
&\texttt{\qquad string name}\newline
&\newline
&\texttt{struct User:}\newline
&\texttt{\qquad string name}\newline
&\texttt{\qquad string age}
\end{align*}
$$

They have no explicit relationship. However, it sort of makes sense that whenever a program expects a $\texttt{Named}$ entity that we can pass it a $\texttt{User}$ (since anything a $\texttt{Named}$ has, a $\texttt{User}$ also has).

{{<box info>}} Example:

$$
\begin{align*}
&\texttt{func rename(Named entity, string newName):}\newline
&\texttt{\qquad entity.name = newName}\newline
&\text{}\newline
&\texttt{User user = }\lbrace\texttt{name: "glee", age: 1000}\rbrace\newline
&\texttt{rename(user, "not glee")}
\end{align*}
$$
{{</box>}}

This can get fairly messy to type check in various type systems.
- C++ has [template metaprogramming](https://en.wikipedia.org/wiki/Template_metaprogramming) ([undecideable](https://en.wikipedia.org/wiki/Undecidable_problem) and thus limited by recursion depth)
- Python uses [duck typing](https://en.wikipedia.org/wiki/Duck_typing)

### Depth Subtyping

#### Definition

Instead of subtyping at the top level, could we subtype at the field level (and thus, their fields recursively)?

$$
\begin{align*}
&\texttt{class LivingSpace:}\newline
&\texttt{\qquad Animal resident}\newline
&\newline
&\texttt{class Studio:}\newline
&\texttt{\qquad Human resident}
&\end{align*}
$$

Does $\texttt{Studio} <: \texttt{LivingSpace}$?

{{<box important>}}Notice that these cannot have hierarchical relationships.{{</box>}}

It might make sense logically that if our program requires a $\texttt{LivingSpace}$ that we may provide it with a $\texttt{Studio}$ since the latter can do anything the former can do (in this case, provide $\texttt{resident.name}$).

$$
\begin{align*}
&\texttt{func owner(LivingSpace home)} \rightarrow \texttt{string:}\newline
&\texttt{\qquad return home.resident.name}\newline
&\text{}\newline
&\texttt{Studio studio = }\lbrace\texttt{resident: }\lbrace- \textit{some human} -\rbrace\rbrace\newline
&\texttt{print(owner(studio))}
\end{align*}
$$

#### Caveat

$$
\begin{align*}
&\texttt{func reassignResident(LivingSpace home, Animal newResident):}\newline
&\texttt{\qquad home.resident = newResident}\newline
&\text{}\newline
&\texttt{Studio studio = }\lbrace\texttt{resident: }\lbrace- \textit{some human} -\rbrace\rbrace\newline
&\texttt{reassignResident(studio, }\lbrace- \textit{some animal } -\rbrace)
\end{align*}
$$

Ah, so here's where it breaks down. We want to assign our subtyped field. But the true underlying structure requires more!

$\texttt{reassignResident}$ is under the impression that an $\texttt{Animal}$ is enough to fill up a $\texttt{LivingSpace}$, which is logically true. But if we pass $\texttt{reassignResident}$ a $\texttt{Studio}$, it should really expect a $\texttt{Human}$ instead of an $\texttt{Animal}$.

> It turns out that the breakdown occurs because of [aliasing](https://en.wikipedia.org/wiki/Aliasing_(computing)#:~:text=In%20computing%2C%20aliasing%20describes%20a,symbolic%20names%20in%20the%20program.). Thus, **depth subtyping** works with *immutable structures* (records).

## Functions

### Introduction
Functions are values. So how do we typecheck function assignment? We'll reuse some types from above.

$$
\begin{align*}
&\texttt{Mammal} \rightarrow \texttt{Mammal someFunction}=\ldots\newline
&\texttt{Mammal input = }\lbrace\ldots\rbrace\newline
&\texttt{Mammal output = someFunction(input)}
\end{align*}
$$

The question is: what types can we assign to $\texttt{someFunction}$?

### Outputs
Let's trial and error, and try to reason the solution out.

$$
\begin{align*}
&\texttt{Mammal} \rightarrow \texttt{Human functionValue}=\ldots\newline
&\texttt{Mammal} \rightarrow \texttt{Mammal someFunction = functionValue}
\end{align*}
$$

$\texttt{functionValue}$ returns a $\texttt{Human}$. Executions of $\texttt{someFunction}$ expect a return value that can do everything a $\texttt{Mammal}$ can do, which a $\texttt{Human}$ satisfies. This works.

$$\texttt{actualOutput} <: \texttt{expectedOutput}$$

---

For sake of clarity, let's try it the other way.

$$
\begin{align*}
&\texttt{Mammal} \rightarrow \texttt{Animal functionValue}=\ldots\newline
&\cancel{\texttt{Mammal} \rightarrow \texttt{Mammal someFunction = functionValue}}
\end{align*}
$$

Clearly, an execution to $\texttt{someFunction}$ which expects a $\texttt{Mammal}$ return type will miss out on the $\texttt{furColor}$ attribute.

### Inputs

Again, let's trial and error.

$$
\begin{align*}
&\texttt{Human} \rightarrow \texttt{Mammal functionValue}=\ldots\newline
&\cancel{\texttt{Mammal} \rightarrow \texttt{Mammal someFunction = functionValue}}
\end{align*}
$$

Clearly, whatever we provide as input must be able to do whatever $\texttt{someFunction}$ requires of it. If we provide $\texttt{Mammal}$, $\texttt{functionValue}$ expects a $\texttt{Human}$ which might use attributes like $\texttt{occupation}$ which aren't present in $\texttt{Mammal}$.

---

$$
\begin{align*}
&\texttt{Animal} \rightarrow \texttt{Mammal functionValue}=\ldots\newline
&\texttt{Mammal} \rightarrow \texttt{Mammal someFunction = functionValue}
\end{align*}
$$

Providing $\texttt{someFunction}$ with an input that can do at least what $\texttt{Mammal}$ can do will allow it to safely be used as input to its value, $\texttt{functionValue}$.

$$\texttt{expectedInput} <: \texttt{actualInput}$$

### Putting it All Together

It's a little counterintuitive, but the inputs and outputs of a function subtype have opposite directionality (as we saw above).

$$
\begin{align*}
&\texttt{Animal} \rightarrow \texttt{Human functionValue}=\ldots\newline
&\texttt{Mammal} \rightarrow \texttt{Mammal someFunction = functionValue}
\end{align*}
$$

> $$\begin{align*}
\texttt{I}&\texttt{'} <:\texttt{I}\newline
&\texttt{O} <:\texttt{O'}\newline
&\implies\newline
\texttt{I}\rightarrow\medspace&\texttt{O} <: \texttt{I'}\rightarrow\texttt{O'}
\end{align*}$$
