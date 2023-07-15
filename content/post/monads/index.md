---
title: Monads
description: A computation builder with operation chaining 
slug: monads
date: 2023-07-15 00:00:00+0000
image: abstract1.png
categories:
    - functional programming
    - types
tags:
    - option
    - result
    - async await
---

## Definition

Monads are structures that wrap values. They are useful for creating computation pipelines that abstract away control flow and side effects.

### Return and Bind
A monad of type $\texttt{'a t}$ has the following functions.

$$
\begin{align*}
&\texttt{return: 'a -> 'a t}\newline
&\texttt{bind: 'a t -> ('a -> 'b t) -> 'b t}
\end{align*}
$$

{{<box info>}}
Monads are [parametric types](https://en.wikipedia.org/wiki/Parametric_polymorphism). $\texttt{t}$ represents the monad itself and $\texttt{'a}$ is the type parameter.

For example, $\texttt{Option}$ is a monad. The value it wraps has the parameter type $\texttt{'a}$. So, you could have types like $\texttt{int Option}$ or $\texttt{string Option}$.

$\texttt{'b}$ is another parameter. Bind takes a function that essentially "maps" the current monad type parameter $\texttt{'a}$ to another monad of type $\texttt{'b}$.
{{</box>}}

It helps to follow the types. $\texttt{return}$ constructs the monad from a value. $\texttt{bind}$ transforms the value within the monad (computation pipeline).

### Map

The definition for the monadic function $\texttt{map}$ follows from the functions above.

$$
\begin{align*}
&\texttt{map: 'a t -> ('a -> 'b) -> 'b t}\newline
&\newline
&\texttt{let map t f =}\newline
&\texttt{\qquad let g a = return (f a) in}\newline
&\texttt{\qquad bind t g}
\end{align*}
$$

{{<box info>}}$\texttt{map}$ is the same as $\texttt{bind}$ except the function ($\texttt{f}$) returns a raw $\texttt{'b}$ rather than a $\texttt{'b t}$.

Here, we construct the $\texttt{g}$ necessary for $\texttt{bind}$ by just applying $\texttt{return}$ to the result of $\texttt{f}$.
{{</box>}}

## OCaml ppx_let

The [ppx_let](https://github.com/janestreet/ppx_let) library for OCaml provides an elegant way to code with monads.

{{<box info>}}Although The [ppx_let](https://github.com/janestreet/ppx_let) library introduces various monadic syntax (i.e. monadic pattern matching), we will only display monadic let bindings.

They work similarly to these monadic let bindings (see documentation).{{</box>}}

$$
\begin{align*}
&\texttt{let\\%bind a = a\\_monad in (\* a : 'a; a\\_monad : 'a t \*)}\newline
&\texttt{\qquad ...}\newline
&\texttt{\qquad b\\_monad (\* b\\_monad : 'b t \*)}
\end{align*}
$$

{{<box info>}}$\texttt{(\* ... \*)}$ are comments.{{</box>}}

With this syntactic sugar, we define the function parameter in (call it $\texttt{f}$ with type $\texttt{'a -> 'b t}$) in the body of the $\texttt{let\\%bind}$, where $\texttt{a}$ is the parameter to $\texttt{f}$.

---

$$
\begin{align*}
&\texttt{let\\%map a = a\\_monad in (\* a : 'a; a\\_monad : 'a t \*)}\newline
&\texttt{\qquad ...}\newline
&\texttt{\qquad b (\* b : 'b \*)}
\end{align*}
$$

$\texttt{let\\%map}$ does basically the same thing except the return type of its body is just a $\texttt{'b}$ (as is in $\texttt{f}$ with type $\texttt{'a -> 'b}$).

{{<box info>}}These might be easier to grasp with the examples below.{{</box>}}

---

Important to note that the type of the entire $\texttt{let\\%bind}$ and $\texttt{let\\%map}$ expressions are $\texttt{'b t}$, consistent with the return types of $\texttt{bind}$ and $\texttt{map}$.

{{<box info>}}
Typically, to chain monadic operations we will end with a single $\texttt{let\\%map}$. All previous bindings will be $\texttt{let\\%bind}$.

$$
\begin{align*}
&\texttt{let\\%bind a = a\\_monad in}\newline
&\texttt{let\\%bind b = b\\_monad in}\newline
&\texttt{let\\%bind c = c\\_monad in}\newline
&\texttt{let\\%map d = d\\_monad in}\newline
&\texttt{(a, b, c, d)}
\end{align*}
$$

Why is this the case?

- $\texttt{let\\%bind}$ expects its function parameter to return a monad, which both $\texttt{let\\%bind}$ and $\texttt{let\\%map}$ do. This is why all except the last level must be $\texttt{let\\%bind}$.
- It is natural to perform computations without monads. Since this usually occurs after "unwrapping" all the monads, the last level is usually a $\texttt{let\\%map}$.

---

Note that if the last level were a $\texttt{let\\%bind}$, we would have to return a monad.

$$
\begin{align*}
&\texttt{let\\%bind a = a\\_monad in}\newline
&\texttt{let\\%bind b = b\\_monad in}\newline
&\texttt{let\\%bind c = c\\_monad in}\newline
&\texttt{let\\%bind d = d\\_monad in}\newline
&\texttt{return (a, b, c, d)}
\end{align*}
$$

{{</box>}}

## Examples

### Option

#### Definition

Options give optionality to the existence of an underlying value.

$$
\begin{align*}
&\texttt{type 'a Option =}\newline
&\texttt{\quad | None}\newline
&\texttt{\quad | Some of 'a}\newline
&\newline
&\texttt{let return a = Some a }\newline
&\texttt{let bind a\\_opt f =}\newline
&\texttt{\qquad match a\\_opt with}\newline
&\texttt{\qquad \qquad None -> None}\newline
&\texttt{\qquad \quad | Some a -> f a}
\end{align*}
$$

#### Use

Suppose we want to implement the $\texttt{option\\_plus}$ function which operates on two parameters of type $\texttt{int Option}$.

$$
\begin{align*}
&\texttt{let option\\_plus (a\\_opt : int Option) (b\\_opt : int Option) : int Option =}\newline
&\texttt{\qquad let\\%bind a = a\\_opt in}\newline
&\texttt{\qquad let\\%map b = b\\_opt in}\newline
&\texttt{\qquad a + b}
\end{align*}
$$

{{<box info>}}This is much more elegant than "if-statement spamming" (or in OCaml, "pattern match spamming").

$$
\begin{align*}
&\texttt{let option\\_plus (a\\_opt : int Option) (b\\_opt : int Option) : int Option =}\newline
&\texttt{\qquad match a\\_opt with}\newline
&\texttt{\qquad \qquad None -> None}\newline
&\texttt{\qquad \quad | Some a ->}\newline
&\texttt{\qquad \qquad (match b\\_opt with}\newline
&\texttt{\qquad \qquad \qquad None -> None}\newline
&\texttt{\qquad \quad \qquad | Some b -> a + b)}
\end{align*}
$$
{{</box>}}

### Result

#### Definition

Results allow values to have a fail condition. They are like options except we can tag the $\texttt{None}$ case with information (i.e. error information).

$$
\begin{align*}
&\texttt{type ('a, 'b) Result =}\newline
&\texttt{\quad | Ok of 'a}\newline
&\texttt{\quad | Error of 'b}\newline
&\newline
&\texttt{let return a = Ok a }\newline
&\texttt{let bind a\\_res f =}\newline
&\texttt{\qquad match a\\_res with}\newline
&\texttt{\qquad \qquad Ok a -> f a}\newline
&\texttt{\qquad \quad | Error \\\_ as err -> err}
\end{align*}
$$

{{<box info>}}The $\texttt{Error \\\_ as err -> err}$ just assigns the entire result into $\texttt{err}$. Alternatively, we could have written $\texttt{Error e -> Error e}$.{{</box>}}

#### Use

Suppose we have a function $\texttt{input: unit -> string}$ which reads from $\texttt{stdin}$.

{{<box info>}}The $\texttt{unit}$ type has only one possible value: $\texttt{()}$. It is useful for when we want a function that requires zero arguments.{{</box>}}

Suppose we have a function which attempts to convert a $\texttt{string}$ to an $\texttt{int}$ and stores the error as a $\texttt{string}$.

$$\texttt{atoi: string -> (int, string) Result}$$

Now, let's write a function to add two user inputs.

$$
\begin{align*}
&\texttt{let add\\_user\\_inputs (() : unit) : (int, string) Result =}\newline
&\texttt{\qquad let\\%bind a = input () |> atoi in}\newline
&\texttt{\qquad let\\%map b = input () |> atoi in}\newline
&\texttt{\qquad a + b}
\end{align*}
$$

{{<box info>}}$\texttt{input () |> atoi}$ is syntactic sugar for $\texttt{atoi (input ())}${{</box>}}

### Deferred

#### Definition

Deferreds allow us to make asynchronous computation. The implementation is a bit more involved, since the idea is computation is queued to a scheduler within a deferred monad. For this reason, *the following code is pseudocode*.

$$
\begin{align*}
&\texttt{type 'a Deferred =}\newline
&\texttt{\quad | Determined of 'a}\newline
&\texttt{\quad | Undetermined}\newline
&\newline
&\texttt{let return a = Determined a }\newline
&\texttt{let bind a\\_def f = ...}\newline
&\texttt{(\* f is queued on the scheduler. The return value of the bind}\newline
&\texttt{ \* (call it x) resolves immediately to undetermined. Upon f's}\newline
&\texttt{ \* execution completion, x becomes a determined. Monadic chains}\newline
&\texttt{ \* (bind to bind to ... to bind to map) continue asynchronously}\newline
&\texttt{ \* on x.}\newline
&\texttt{ \*}\newline
&\texttt{ \* These deferred monadic chains are "upon" computations.}\newline
&\texttt{ \*)}\newline
\end{align*}
$$

#### Use

Suppose we have a function  write a function to crawl a webpage and click all links and print visited urls. The function halts once it has reached a given depth (all links on the first page result have depth $1$; all of their links have depth $2$; etc).

Here are the functions we are given to use:
- $\texttt{print: string -> unit}$ outputs to $\texttt{stdout}$ 
- $\texttt{curl: string -> string Deferred}$ queries the web for an html page given its web address
- $\texttt{get\\_links: string -> string List}$ grabs a list of all links on an html page
- $\texttt{List.map: 'a List -> ('a -> 'b) -> 'b List}$ takes a list of items and performs a computation on all of its elements
- $\texttt{Deferred.all: 'a Deferred List -> 'a List Deferred}$ transforms a list of deferreds into a single deferred holding a list 

{{<box info>}}Notice the $\texttt{List.map}$. Indeed there exists a list monad! We will not be covering it.{{</box>}}

$$
\begin{align*}
&\texttt{let crawl (i : int) (url : string) : unit Deferred =}\newline
&\texttt{\qquad let () = print url in}\newline
&\texttt{\qquad if i = 0 then return () else}\newline
&\texttt{\qquad let j = i - 1 in}\newline
&\texttt{\qquad let\\%bind html = curl url in}\newline
&\texttt{\qquad let links = get\\_links html in}\newline
&\texttt{\qquad let crawls = List.map links (crawl j) in}\newline
&\texttt{\qquad let\\%map (\\\_ : unit List) = Deferred.all crawls in}\newline
&\texttt{\qquad ()}
\end{align*}
$$

{{<box info>}}
Note that $\texttt{crawl}$ is non-blocking. If we call it, it will immediately return a $\texttt{unit Deferred}$ and we can continue the program ($\texttt{crawl}$ is queued by the scheduler). If at any point, we want to block until $\texttt{crawl}$ completes, we can monadic bind on the $\texttt{unit Deferred}$ return value.

$$
\begin{align*}
&\texttt{let unit\\_def = crawl 5 "https\://www.wikipedia.org" in}\newline
&\texttt{(\* ... do some stuff ... \*)}\newline
&\texttt{let\\%map () = unit\\_def in ()}\newline
\end{align*}
$$

{{</box>}}
