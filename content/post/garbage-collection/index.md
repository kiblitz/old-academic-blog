---
title: Garbage Collection
description: Taking out the memory allocated trash
slug: garbage-collection
date: 2023-07-17 00:00:00+0000
image: garbage.jpg
categories:
    - memory management
    - algorithms
tags:
    - reference counting
    - mark sweep
    - copy collection
    - generational collection
    - conservative collection
---

## Introduction

Oftentimes, your program will need to allocate memory on the heap. Unlike the stack where deallocations occur upon scope exiting, something needs to explicitly free memory from the heap.

In some languages like $\texttt{C}$, it is the programmer's responsibility to call $\texttt{free}$ when it is no longer needed. Manual memory leak debugging and tests were required (i.e. [valgrind](https://en.wikipedia.org/wiki/Valgrind)).

A lot of higher-level programming languages have a "garbage collector" (gc). Aptly named, they execute independently of the main program and free memory that is no longer being used (garbage).

## Techniques

Garbage collection is a difficult task. There is a juggling tradeoff between the completeness of the gc (what leaks can we allow) versus the resource complexity of the gc (time, space, processing).

### Reference Counting

For every allocated piece of memory, store a referencing counter representing the number of pointers that point to it. If it ever reaches $0$, free the memory chunk.

#### Algorithm

$$
\begin{align*}
&\texttt{on new $M$:}\newline
&\texttt{\qquad $M$.ref\\_count = 1}\newline
&\newline
&\texttt{on $x$ = $M$:}\newline
&\texttt{\qquad $M$.ref\\_count++}\newline
&\newline
&\texttt{on ($x$ = $M$) out of scope:}\newline
&\texttt{\qquad $M$.ref\\_count-\-}\newline
&\texttt{\qquad if $M$.ref\\_count = 0}\newline
&\texttt{\qquad \qquad free($M$)}\newline
&\texttt{\qquad \qquad recurse out-of-scope check on $M$ fields}
\end{align*}
$$

#### Pros

- Very simple to implement
- No background thread running in background (all operations are triggered)

#### Cons

- Word overhead for storing $\texttt{ref\\_count}$ per allocated memory block
- Chain overhead when freeing a linked data structure
{{<box info>}}
```goat
A --->  B --->  C --->  D --->  E --->  F
```
Imagine $\texttt{A}$ is freed. Then, $\texttt{B}$ is freed. Then, $\texttt{C}$ is freed. And so on. So there is a potential high overhead when freeing linked data structures (program pause).
{{</box>}}
- **Cycles leak memory**
{{<box info>}}
```goat
A --->  B                                
        |                                
^       v                                
|                                        
D  <--- C                                
```
These nodes point to each other. So, their $\texttt{ref\\_count}$ will never be $0$ even if nothing else in the program points to them (should be garbage).
{{</box>}}
{{<box tip>}}
[Python](https://en.wikipedia.org/wiki/Python_(programming_language)) uses reference counting for its gc. Watch it leak by making a large number of cycle structures.
{{</box>}}

### Mark Sweep

For every allocated block of memory, store a mark bit (initialized to $0$). Every now and then, mark all memory that have pointers to them within the current program scope. Free all allocated memory that has not been marked (cannot be accessed $\implies$ garbage). Unmark all marks.

#### Algorithm

$$
\begin{align*}
&\texttt{let markRecurse(ptr $p$):}\newline
&\texttt{\qquad if $p$ is marked then return}\newline
&\texttt{\qquad mark $p$}\newline
&\texttt{\qquad for all pointers $f$ that are fields of $p$:}\newline
&\texttt{\qquad \qquad markRecurse($f$)}\newline
&\newline
&\texttt{on gc trigger:}\newline
&\texttt{\qquad pause program}\newline
&\newline
&\texttt{\qquad // mark phase}\newline
&\texttt{\qquad for all pointers $p$ in scope:}\newline
&\texttt{\qquad \qquad markRecurse($p$)}\newline
&\newline
&\texttt{\qquad // sweep phase}\newline
&\texttt{\qquad for all allocated memory blocks $b$:}\newline
&\texttt{\qquad \qquad if $b$ is marked then free($b$)}\newline
&\texttt{\qquad \qquad unmark $b$}\newline
&\newline
&\texttt{\qquad resume program}
\end{align*}
$$

{{<box info>}}The gc trigger is heuristical. It could be based on time intervals, memory utilization, processor utilization, etc.{{</box>}}

#### Pros

- The gc will collect all garbage

#### Cons

- Long pauses when gc is running

### Copy Collection

Divide the heap into two equal sections: *from* and *to*. For every allocated block of memory, store a "forwarding address" (initialized to $0$). All newly allocated blocks are placed in the *from* space. Once full, the gc is triggered. It moves all non-garbage blocks to the *to* space and frees the *from* space. Then, it swaps the *from* and *to* space labels.

#### Algorithm

$$
\begin{align*}
&\texttt{let copyRecurse(ptr $p$):}\newline
&\texttt{\qquad if $p$ has no forwarding address:}\newline
&\texttt{\qquad \qquad let $m$ = next available block in $to$ space}\newline
&\texttt{\qquad \qquad copy non-ptr fields of $p$ into $m$}\newline
&\texttt{\qquad \qquad $p$ forwarding address = $m$ (in $from$ space)}\newline
&\texttt{\qquad \qquad for all ptr fields $f$ of $p$:}\newline
&\texttt{\qquad \qquad \qquad $f$ forwarding address = copyRecurse($f$)}\newline
&\texttt{\qquad return $p$ forwarding address}\newline
&\newline
&\texttt{on gc trigger:}\newline
&\texttt{\qquad pause program}\newline
&\newline
&\texttt{\qquad for all pointers $p$ in scope:}\newline
&\texttt{\qquad \qquad copyRecurse($p$)}\newline
&\texttt{\qquad free($from$ space)}\newline
&\newline
&\texttt{\qquad swap $to$ and $from$ labels}\newline
&\newline
&\texttt{\qquad resume program}
\end{align*}
$$

{{<box info>}}The "forwarding address" is for the gc to move cyclical data structures to the *to* space.

Without it, recursing on pointers to blocks already in the *to* space will move the same block again (essentially duplicating it infinitely in the *to* space).

```goat
+--------+--------+  +--------+--------+
|  from  |  to    |  |  from  |  to    |
|        |        |  |        |        |
| +-> A  |        |  | +-> A  |   A    |
| |   |  |        |  | |   |  |        |
| |   v  |        |  | |   v  |        |
| |      |        |  | |      |        |
| +-- B  |        |  | +-- B  |        |
+--------+--------+  +--------+--------+

+--------+--------+  +--------+--------+
|  from  |  to    |  |  from  |  to    |
|        |        |  |        |        |
| +-> A  |   A    |  | +-> A  |   A    |
| |   |  |   |    |  | |   |  |   |    |
| |   v  |   v    |  | |   v  |   v    |
| |      |        |  | |      |        |
| +-- B  |   B    |  | +-- B  |   B    |
+--------+--------+  |        |   |    |
                     |        |   v    |
                     |        |        |
                     |        |   A    |
                     +--------+--------+
```
{{</box>}}

#### Example

```goat
+--------+--------+  +--------+--------+
|  from  |  to    |  |  from  |  to    |
|        |        |  |        |        |
| +-> A  |        |  | +-> o--+-> A    |
| |   |  |        |  | |   |  |        |
| |   v  |        |  | |   v  |        |
| |      |        |  | |      |        |
| +-- B  |        |  | +-- B  |        |
+--------+--------+  +--------+--------+

+--------+--------+  +--------+--------+
|  from  |  to    |  |  to    |  from  |
|        |        |  |        |        |
| +-> o--+-> A    |  |        |   A    |
| |   |  |   |    |  |        |   |    |
| |   v  |   v    |  |        |   v    |
| |      |        |  |        |        |
| +-- o--+-> B    |  |        |   B    |
+--------+--------+  +--------+--------+
```

#### Pros

- The gc will collect all garbage
- Efficient memory utilization (all allocations are contiguous: no fragmentation)
- Only $1$ iteration (as opposed to $2$ from mark/sweep)

#### Cons

- Long pauses when gc is running
- Only half of the heap can be operated on at a time

## Lifetime Optimization

A general analysis of programs yields the following result.
- Allocated memory blocks that live past a certain point tend to live a really long time
- Other blocks tend to be short lived

This sort of makes sense. These long lasting blocks are likely to be data structures that persist throughout the program. On the other hand, short lived blocks might be temporaries that we only need for a scope (i.e. within function).

An optimization to employ is **generational collection**. The heap is divided into two sections: *nursery* and *tenured*.

```goat
+-------------------------+-------------------------+
|                         |                         |
|                         |                         |
|                         |                         |
|         nursery         |         tenured         |
|          [new]          |          [old]          |
|                         |                         |
|                         |                         |
|                         |                         |
+-------------------------+-------------------------+
```

All newly allocated blocks are placed in the *nursery*. When a *nursery* gc triggers and cleans up the trash, all blocks that were not cleaned up are "promoted" to live in the *tenured* space. There is also a *tenured* gc, but that executes less frequently than the *nursery* gc (since those blocks are less likely to be garbage).
{{<box info>}}Observe that after a *nursery* gc execution, the entire *nursery* is freed. Much like in **copy collection**, we can just reclaim the entire space at once rather than for each individual block.{{</box>}}

## Type Unsafe Problem
Lots of higher-level programming languages have type systems that keep code "safe". In a lower-level language like $\texttt{C/C++}$, anything goes. Specifically, any piece of data could be casted to a pointer.

{{<box info>}}For instance, $\texttt{int x = 5}$ could technically also be a pointer to the memory address $\texttt{0x5}$.{{</box>}}

An approach is to employ **conservative collection**. Any *data* in scope is conservatively considered to be a pointer. If there exists allocated memory at the address specified by that data, then consider it "not garbage" (don't free it).

{{<box info>}}If we had $\texttt{int x = 5}$ and there was $\texttt{Object obj}$ allocated at address $\texttt{0x4}$ with size $\texttt{0x8}$, then consider $\texttt{obj}$ to be in use.

The reason being is that we could technically use $\texttt{x}$ to access $\texttt{obj}$, so we can't deem $\texttt{obj}$ as garbage yet.
{{</box>}}

It's a conservative approach because the gc might falsely mark a piece of memory as still in use, thus leaking it.
