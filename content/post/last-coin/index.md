---
title: Last Coin
description: Brain teaser that makes for a neat party trick
slug: last-coin
date: 2023-06-16 00:00:00+0000
image: coin.jpeg
categories:
    - brain teaser
tags:
    - game theory
    - coins
    - dice
---

## Problem

> You and your friend come accross an adversary $\mathcal{E}$ who challenges you to a game. 
>
> One of you will privately visit $\mathcal{E}$ at his grand table. WLOG, let's say it was you. There are $n$ coins that can fit in $n$ empty slots. One-by-one, $\mathcal{E}$ will point at an empty slot and you must place a coin positioned either heads or tails on that slot. This goes on until $n-1$ slots have been filled.
>
> At this point, $\mathcal{E}$ will fill the last slot $L$ with a coin oriented at their discretion. Now, your friend will come to the grand table. It is their job to figure out which slot is $L$ without consulting you. Specifically, they must select a set of $K$ slots for which they can guarantee one of these slots is $L$.
>
> What strategy will you and your friend employ to minimize $K$?
> ```goat
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | H | | T | | H | | H | | T | | H | | T | | T | | H | | H | | H | | H | | H |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.  +---+  .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | T | | H | | H | | H | | T | | L | | H | | T | | H | | T | | T | | H | | T |
>  '-'   '-'   '-'   '-'   '-'  +---+  '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | H | | T | | H | | H | | T | | T | | T | | H | | H | | H | | T | | H | | T |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | H | | T | | H | | H | | T | | T | | T | | H | | T | | H | | T | | T | | H |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | H | | T | | T | | H | | H | | H | | T | | T | | H | | T | | T | | H | | H |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
> ```
## Some Ideas
What if we could signal to our friend what the last slot is like an airport runway?
```goat
 .-.   .-.   .-.   .-.   .-.                                                
| T | | T | | H | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| T | | T | | H | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.  +---+  .-.   .-. 
| H | | H | | L | | H | | H |
 '-'   '-'  +---+  '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| T | | T | | H | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| T | | T | | H | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
```
The issue is, we don't know what $L$ is ahead of time. Specifically, the last slots chosen by $\mathcal{E}$ could all be disconnected from each other so we couldn't coordinate the signal.

```goat
 .-.  +---+  .-.   .-.   .-.                                                
| T | |   | | T | | T | | T |
 '-'  +---+  '-'   '-'   '-' 
 .-.   .-.   .-.   .-.  +---+
| T | | T | | T | | T | |   |
 '-'   '-'   '-'   '-'  +---+
 .-.  +---+  .-.   .-.   .-. 
| T | |   | | T | | T | | T |
 '-'  +---+  '-'   '-'   '-' 
 .-.   .-.   .-.  +---+  .-. 
| T | | T | | T | |   | | T |
 '-'   '-'   '-'  +---+  '-' 
+---+  .-.   .-.   .-.   .-. 
|   | | T | | T | | T | | T |
+---+  '-'   '-'   '-'   '-' 
```

{{<box important>}}Spoilers ahead{{</box>}}

## Solution

The trick is to force $\mathcal{E}$ into a position where either final board state yields information.

### The Right Direction

What if we designate the first $\frac{n}{2}$ slots to be for heads, and the rest to be for tails?
```goat
 .-.   .-.   .-.   .-.   .-.                                                
| H | | H | | H | | H | | H |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| H | | H | | H | | H | | H |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.  +---+  .-.   .-. 
| H | | H | | L | | T | | T |
 '-'   '-'  +---+  '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| T | | T | | T | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
 .-.   .-.   .-.   .-.   .-. 
| T | | T | | T | | T | | T |
 '-'   '-'   '-'   '-'   '-' 
```
Whichever is the majority, we know that $L$ must be among them (in the case where $n$ is even, if we picked more heads and the final board state was split, then we know $L$ must be tails).

Thus, $K$ has approximately (rounded up) $\frac{n}{2}$ coins.

### Grouping

#### Pairs
Here's an idea. Make pairs of coin slots. The first chosen slot in a pair will be heads, the second tails. When $\mathcal{E}$ picks the last orientation, one of two cases occurs.
* $\mathcal{E}$ chooses heads. Then, one pair will have two heads so we know that it was one of these two slots.
```goat
 .-.   .-.   |   .-.   .-.   |  +---+  .-.   |   .-.   .-.   |   .-.   .-. 
| H | | T |  |  | T | | H |  |  | H | | H |  |  | H | | T |  |  | H | | T |
 '-'   '-'   |   '-'   '-'   |  +---+  '-'   |   '-'   '-'   |   '-'   '-'
```
* $\mathcal{E}$ chooses tails. Then, all pairs will have one coin of each orientation, so we know it was one of the tails.
```goat
 .-.   .-.   |   .-.   .-.   |  +---+  .-.   |   .-.   .-.   |   .-.   .-. 
| H | | T |  |  | T | | H |  |  | T | | H |  |  | H | | T |  |  | H | | T |
 '-'   '-'   |   '-'   '-'   |  +---+  '-'   |   '-'   '-'   |   '-'   '-'
```
In the worst case (when all groups are similar), $K$ has $\frac{n}{2}$ coins.

#### Triples
Make triples of coin slots. The first two chosen slots in a triple will be heads, the last tails. When $\mathcal{E}$ picks the last orientation, one of two cases occurs.
* $\mathcal{E}$ chooses heads. Then, one triple will have all heads so we know that it was one of these slots.
```goat
 .-.   .-.   .-.   |   .-.  +---+  .-.   |   .-.   .-.   .-.               
| H | | H | | T |  |  | H | | H | | H |  |  | H | | T | | H |
 '-'   '-'   '-'   |   '-'  +---+  '-'   |   '-'   '-'   '-'
```
* $\mathcal{E}$ chooses tails. Then, all triples will have one coin oriented at tails, so we know it was one of the tails.
```goat
 .-.   .-.   .-.   |   .-.  +---+  .-.   |   .-.   .-.   .-.               
| H | | H | | T |  |  | H | | T | | H |  |  | H | | T | | H |
 '-'   '-'   '-'   |   '-'  +---+  '-'   |   '-'   '-'   '-'
```
In the worst case (when all groups are similar), $K$ has $\frac{n}{3}$ coins.

#### Generalizing
We can keep increasing group sizes at the cost of number of groups. At what group size/count is $K$ optimal?

Once the group size surpasses the number of groups, the worst case scenario will be when one group differs from the rest (last coin within a group is heads). 
```goat
|   .-.   .-.  +---+  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   |    
|  | H | | H | | H | | H | | H | | H | | H | | H | | H | | H | | H |  |
|   '-'   '-'  +---+  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   |
```

$\min(\text{size, count})$ is maximized when $\text{size}=\text{count}$. Since $\text{size}*\text{count}=n$, our group sizes are optimally set to $\sqrt{n}$ (specifically, $\lceil\sqrt n\rceil$). Thus, $K$ is minimized to $\lceil\sqrt n\rceil$ coins.

{{<box info>}}Ceiling since if $n$ is not a perfect square, we should create groups as if it were a perfect square and blank out the nonexistent coins afterwards{{</box>}}

## Extension
### Problem
Suppose we play the same game, except instead of coins we play with $m$-sided dice. How does the strategy change, and how does $K$ change?

Obviously if $m=1$, then vacuously $K$ must have all $n$ dice.

$m=2$ is just the original problem.

If $m>2$, one strategy is to use the same strategy as in $m=2$, and only utilize $2$ values of the dice. This means that optimal $K$ has at most $\lceil\sqrt{n}\rceil$ dice. With one extra value, we can do better.

{{<box important>}}Spoilers ahead{{</box>}}
### 3-sided Dice
#### The Right Direction
We need to approach this $m$-dimensionally. Intuitively, $K$ should optimally have $\sqrt[m]{n}$, since extra orientations should correlate with axes to work with. But let's start with a concrete example: $n=27$ and $m=3$.

Again, we want the last dice to shrink the "last dice space", specifically to $3$ possibilities here. As in the original problem, perhaps we want groups to mainly consist of $1$ so that if $\mathcal{E}$ sets the $L$ dice to $1$, we know it is in that group.
```goat
 .-.   .-.   .-.                                                           
| 1 | | 1 | |   |
 '-'   '-'   '-'
```
Thinking $3$-dimensionally, what if we had $3$ groups of $3$ groups of $3$? Let's start with just the first group of groups, or ***"row"***.
```goat
 .-.   .-.   .-.   |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | |   |  |  | 1 | |   | | 1 |  |  | 1 | | 1 | |   |
 '-'   '-'   '-'   |   '-'   '-'   '-'   |   '-'   '-'   '-'
```
Suppose $L$ is part of this "row". In other words, $L$ is one of these unfilled slots. Let's employ the same strategy as before where the last dice of each group is oriented at $2$.
```goat
 .-.   .-.  +---+  |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | |   |  |  | 1 | | 2 | | 1 |  |  | 1 | | 1 | | 2 |
 '-'   '-'  +---+  |   '-'   '-'   '-'   |   '-'   '-'   '-'
```
If $L$ is set to $1$, we know it is in the first group. If $L$ is set to $2$, then we know that it is among the "$2$ dice" in this row.

We still have two more rows though.

#### "3 dice" Advantage
If we naively just use the $m=2$ strategy for all rows:
```goat
 .-.   .-.  +---+  |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | |   |  |  | 1 | | 2 | | 1 |  |  | 1 | | 1 | | 2 |
 '-'   '-'  +---+  |   '-'   '-'   '-'   |   '-'   '-'   '-'

 .-.   .-.   .-.   |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 2 | | 1 | | 1 |  |  | 2 | | 1 | | 1 |  |  | 1 | | 2 | | 1 |
 '-'   '-'   '-'   |   '-'   '-'   '-'   |   '-'   '-'   '-'

 .-.   .-.   .-.   |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | | 2 |  |  | 1 | | 1 | | 2 |  |  | 1 | | 2 | | 1 |
 '-'   '-'   '-'   |   '-'   '-'   '-'   |   '-'   '-'   '-'
```
$\mathcal{E}$ orienting the $L$ dice to $2$ only tells us it's one of all of the "$2$ dice" among all of the rows. It's time to use the "3 dice".

Following the pattern, if we assign the last of every group to be $2$, what if we assign the last of every row to be $3$?
```goat
 .-.   .-.  +---+  |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | |   |  |  | 1 | | 2 | | 1 |  |  | 1 | | 1 | | 2 |
 '-'   '-'  +---+  |   '-'   '-'   '-'   |   '-'   '-'   '-'

 .-.   .-.   .-.   |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 2 | | 1 | | 1 |  |  | 2 | | 1 | | 1 |  |  | 1 | | 3 | | 1 |
 '-'   '-'   '-'   |   '-'   '-'   '-'   |   '-'   '-'   '-'

 .-.   .-.   .-.   |   .-.   .-.   .-.   |   .-.   .-.   .-.               
| 1 | | 1 | | 3 |  |  | 1 | | 1 | | 2 |  |  | 1 | | 2 | | 1 |
 '-'   '-'   '-'   |   '-'   '-'   '-'   |   '-'   '-'   '-'
```
* If $L$ is assigned $1$, we know it is in the group with all $1$s
* If $L$ is assigned $2$, then we know it is among the $2$s in the row that doesn't have a single $3$
* If $L$ is assigned $3$, we know that it is among the $3$s across rows

Thus, $K$ is minimized to $\lceil\sqrt[3] n\rceil$ dice.

### 4-sided Dice
#### Extending 4-sided Dice
It is difficult to imagine things with dimensionality greater than $3$. Instead, we can think about it as just extending the idea in $m=3$ with more groups of groups.

Let $g=\sqrt[4]{n}$ be the group size. Each row has $g^3$ dice, and there are $g$ rows. Partitioned further, each row has $g$ groups of groups, or ***"inner rows"*** each with $g$ groups of $g$ dice each. Groups are assigned $1$ until the last which is assigned $2$. Except in inner rows, where it is assigned $3$. Except in rows where it is assigned $4$.

#### Concrete Example
Suppose $n=16$.
```goat
 .-.   .-.   |   .-.   .-.         .-.   .-.   |   .-.   .-.               
| 1 | | 4 |  |  | 1 | | 2 |       | 3 | | 1 |  |  | 2 | | 1 |
 '-'   '-'   |   '-'   '-'         '-'   '-'   |   '-'   '-'

 .-.   .-.   |   .-.   .-.        +---+  .-.   |   .-.   .-.
| 1 | | 2 |  |  | 3 | | 1 |       |   | | 1 |  |  | 1 | | 2 |
 '-'   '-'   |   '-'   '-'        +---+  '-'   |   '-'   '-'
```
As before, the last of each group is assigned $2$ until it is the last of its parent group. Each of the four quadrants represents an inner row, and its last die is assigned $3$ (top right and bottom left). The rows are just rows and have its last die assigned $4$.

* If $L$ is assigned $1$, we know it is in the group with all $1$s
* If $L$ is assigned $2$, we know that it is among the $2$s in the inner row with all $2$s 
* If $L$ is assigned $3$, then we know it is among the $3$s in the row that doesn't have a single $4$
* If $L$ is assigned $4$, then we know it is among the $4$s across rows

Thus, $K$ is minimized to $\lceil\sqrt[4] n\rceil$ dice.

### Generalizing
We can continue to recursively create more parent groups as $m$ increases as shown above. Doing so allows us to optimize $K$ to have $\lceil\sqrt[m]n\rceil$ dice.
