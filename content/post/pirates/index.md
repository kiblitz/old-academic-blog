---
title: Pirates!
description: Brain teaser that makes for a neat party trick
slug: pirates
date: 2023-06-11 00:00:00+0000
image: storm_seas_ship.jpg
categories:
    - brain teaser
tags:
    - game theory
    - recursion
---

This post has been heavily modularized so you can follow step-by-step to try to solve the puzzle.

## Problem

> A crew of $100$ pirates have found treasure: $100$ gold doubloons. They need to figure out a way to distribute the treasure amongst themselves.
> ```goat
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
>  .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.   .-.
> | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G | | G |
>  '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'   '-'
> ```
> Specifically, there is a linear chain of command that is publically known, and at the top of the chain is the pirate captain. The captain is the one who proposes the treasure distribution. Let's say we label every pirate with a number corresponding to chain power.
> $$ c\rightarrow 99 \rightarrow 98 \rightarrow 97 \rightarrow ... \rightarrow 4 \rightarrow 3 \rightarrow 2 \rightarrow 1$$
> The pirates then simultaneously say "aye!" (including the captain) if they agree with the proposal. If at least half of the pirates say "aye!", then it is realized. Otherwise, the captain walks the plank, and the next in command becomes captain!
> $$ c\rightarrow 98 \rightarrow 97 \rightarrow ... \rightarrow 4 \rightarrow 3 \rightarrow 2 \rightarrow 1$$
> What influences a pirate's vote?
> * Pirates are trying to maximize their own loot 
> * In the case where a vote does not affect their rewards, pirates are ruthless and won't say "aye!"
> 
> So, how should the captain distribute the doubloons?

## Some Ideas
Well, there are $100$ pirates splitting $100$ gold doubloons. Maybe the captain should $1$ to each pirate?

But the captain really only needs half of the votes. How about $2$ to $50$ of them?

There's also a chain of command... the pirates way down the hierarchy have no power right? Can the captain afford to give less doubloons to those pirates while still keeping their vote?

Clearly, there's a optimal strategy for the captain to maximize their own loot, otherwise it wouldn't make for a interesting puzzle.

{{<box important>}}Spoilers ahead{{</box>}}

## Thinking with Small Numbers
$100$ is a bit big to think about all of the possibilities. What if we were dealing with less pirates?

### Baby Steps
Let's start at the top and put ourselves in the captain's shoes. $1$ pirate. Clearly with no one to make us walk the plank, we should give ourselves all $100$ doubloons!

How about $2$ pirates? As long as we say "aye!", the second-in-command will never get enough votes. Again, we get all $100$ doubloons.

Ok, what about $3$ pirates? Now we are in danger, since there are enough pirates for a mutiny. We should entice one of them with some of the loot. But which one? And with how much?
$$c\rightarrow 2 \rightarrow 1$$

#### Buying Votes
Let's remember the pirate code.
> * Pirates are trying to maximize their own loot
> * In the case where a vote does not affect their rewards, pirates are ruthless and won't say "aye!"

Here's an observation. If we walk the plank, pirate $2$ becomes the new captain. How much would they get if this scenario happens?

$$2 \rightarrow 1$$
Doesn't this look familiar? It's the same case that we looked at before! Pirate $2$ will get all the doubloons here!
$$2[100] \rightarrow 1[0]$$

So, pirate $2$ will always vote to kill and try to be captain no matter what! Even if we as captain gave all the doubloons to pirate $2$, the pirate code states that pirates are ruthless!
{{<box info>}}It follows that no matter the number of alive pirates in the crew, the relative second-in-command will always vote to kill if they want to be captain.{{</box>}}
Clearly pirate $1$ is who we should be paying. How much should we spend?

Let's think back to the scenario where pirate $2$ becomes captain.
$$2[100] \rightarrow 1[0]$$
Pirate $1$ isn't getting anything here! So, pirate $1$ really doesn't want pirate $2$ to be captain!

Can we be greedy and not give anything to pirate $1$? Well, remember the pirate code. $0=0$, so we're walking the plank. But $1>0$!
$$c[99] \rightarrow 2[0] \rightarrow 1[1]$$

### Less Baby Steps

What about with $4$ pirates?
$$c\rightarrow 3 \rightarrow 2 \rightarrow 1$$
Applying our logic from before, pirate $3$ is getting $99$ doubloons if they get to be captain. If we want to buy their vote, we need to give them $100$ doubloons.
$$c[0]\rightarrow 3[100] \rightarrow 2[0] \rightarrow 1[0]$$
And we have $2$ votes: ourselves and pirate $3$. But this is kind of silly.

Ok, again using our logic from before, pirate $2$ will be sad and poor if pirate $3$ becomes captain. The solution follows naturally.
$$c[99]\rightarrow 3[0] \rightarrow 2[1] \rightarrow 1[0]$$

## Generalizing the Approach
### Solving the Problem
Looks like an alternating pattern. This happens because all the pirates who will get $0$ if the captain gets thrown overboard can be incentivized with $1$ doubloon. And since that makes up half of the crew, the captain can just give everyone else nothing!
$$ c[51]\rightarrow 99[0] \rightarrow 98[1] \rightarrow 97[0] \rightarrow ... \rightarrow 4[1] \rightarrow 3[0] \rightarrow 2[1] \rightarrow 1[0]$$
{{<box info>}}This is a tight upper bound on the captain's rewards. The crew is distributed $49$ doubloons. This is exactly the number of "ayes!" the captain needs (not including themselves) to not walk the plank.{{</box>}}
### More Generalization
The captain has earned a hefty sum. That means there's still space for more pirates!

It's clear that we can keep this alternating pattern all the up to $200$ pirates.
$$ c[1]\rightarrow 199[0] \rightarrow 198[1] \rightarrow 197[0] \rightarrow ... \rightarrow 4[1] \rightarrow 3[0] \rightarrow 2[1] \rightarrow 1[0]$$
The natural follow up is: what happens after $200$ pirates?
{{<box important>}}Spoilers ahead{{</box>}}
## Too Many Pirates

### One Step at a Time

What happens when there are $201$ pirates? We are $1$ doubloon short if we try to use our normal strategy.

Notice that, not including ourselves, we need $100$ votes. If we can't incentivize $100$ other pirates, then it's plank time.

Now it's obvious what the strategy should be.

$$ c[0]\rightarrow 200[0] \rightarrow 199[1] \rightarrow 198[0] \rightarrow ... \rightarrow 4[0] \rightarrow 3[1] \rightarrow 2[0] \rightarrow 1[1]$$

Ok, so we're a little sad that we don't get any of the loot. But we're not as sad as we would be if we lost this vote.

It's clear that we can use the same strategy for when there are $202$ pirates.

$$ c[0]\rightarrow 201[0] \rightarrow 200[1] \rightarrow 199[0] \rightarrow ... \rightarrow 4[1] \rightarrow 3[0] \rightarrow 2[1] \rightarrow 1[0]$$

### 203 Pirates

Ok, now what?

We need $101$ other pirates to say "aye!" but we only have $100$ doubloons to reward. Furthermore, there are $101$ pirates in the previous case that were not awarded any doubloons, which means that unless they are motivated to do so, they will vote to kill.

Does this mean that with $203$ pirates, we are doomed to walk the plank?

Unfortunately so. Hopefully we can wash upon an island and join another band of pirates.

So that means that with more than $202$ pirates, the captain is always destined to die right?
{{<box important>}}Spoilers ahead{{</box>}}
### 204 Pirates

#### Déjà Vu?

The argument in the previous case for our imminent death was that we needed $101$ other votes but could only buy $100$ of them. Again, we need $101$ other votes here. Isn't it the same case now?

Here's a hint. There is one pirate that doesn't require monetary motivation.

#### Another Motivator

Here is a proposed rewards distribution.

$$ c[0]\rightarrow 203[0] \rightarrow 202[0] \rightarrow 201[0] \rightarrow 200[0] \rightarrow 199[1] \rightarrow ... \rightarrow 4[0] \rightarrow 3[1] \rightarrow 2[0] \rightarrow 1[1]$$

As always, we get our $100$ "ayes!" from the last $200$ pirates. We get $1$ "aye!" from ourselves. Who else is on our side amongst pirates $201,202,203$?

Turns out death is a strong motivator. Remember, if $203$ becomes captain, they will walk the plank! So pirate $203$ will also say "aye!".

### Generalizing the New Strategy

It is the case that we will always distribute the coins amongst pirates $1$ through $200$ to get our first $100$ votes. Thus for our analysis, it is sufficient to just look at pirates $201$ onwards.
$$\boxed{201}\boxed{202}\xcancel{\boxed{203}}\boxed{204}$$

#### 205 Pirates

For sure, we are not getting pirate $204$'s "aye!" as they won't be sad as captain. And pirate $203$ also won't be sad if $204$ is captain. Pirates $201$ and $202$ are not on the chopping block, so we are not getting their votes either.
$$\boxed{201}\boxed{202}\xcancel{\boxed{203}}\boxed{204}\xcancel{\boxed{205}}$$
#### 206 Pirates

Using the logic from the previous case, pirates $201$ through $204$ won't be sad if pirate $205$ becomes captain. The only pirate that will be sad is pirate $205$. So we get their vote!
$$201,202,203,204\quad---\quad \underbrace{205, 206}_\text{aye!}$$
Darn, still walking the plank.

#### 207 Pirates
$$201,202,203,204\quad---\quad \underbrace{205, 206, 207}_\text{aye!}$$

#### 208 Pirates
$$201,202,203,204\quad---\quad \underbrace{205, 206, 207, 208}_\text{aye!}$$
Wow! We've survived!

Unfortunately for the captains in the next few cases, since pirates $205$ through $208$ are not sad with $208$ being captain, they will not be saying "aye!".

#### 216 Pirates
$$\begin{matrix}
   201,202,203,204,\newline 205,206,207,208\thickspace
\end{matrix}
\quad---\quad
\underbrace{
\begin{matrix}
    209,210,211,212, \newline 213,214,215,216\thickspace
\end{matrix}
}_\text{aye!}$$

Another surviving captain! Now the idea is clear. Every surviving captain creates a "sink state" where every pirate up to that point will never say "aye!" ever again (since they are not sad if they end up in that state). Every next surviving captain must have double the number of pirates labelled after $200$ in order to tie with these no-"aye!" pirates.


Thus, a pirate captain only survives if they are a power of $2$ after $200$.
$$201, 202, 204, 208, 216, 232, 264, ...$$
