---
title: "Dynamic Dispatch: Memory Organization"
description: How exection of a possibly overridden function works
slug: dynamic-dispatch
date: 2023-07-14 00:00:00+0000
image: blocks.jpg
categories:
    - memory management
tags:
    - inheritance
    - multiple inheritance
    - vtable
---

## Introduction

Suppose a class and its two children have the following structures.

$$
\begin{align*}
&\texttt{class Pet:}\newline
&\texttt{\qquad string name}\newline
&\texttt{\qquad int age}
&\newline
&\newline
&\texttt{class Cat extends Pet:}\newline
&\texttt{\qquad string catBreed}\newline
&\text{}\newline
&\texttt{class Dog extends Pet:}\newline
&\texttt{\qquad string dogBreed}
\end{align*}
$$

So maybe a person wants to keep track of their pets. Since both $\texttt{Cat}$ and $\texttt{Dog}$ are [subtypes](/p/subtyping/) of $\texttt{Pet}$, we can store them in a $\texttt{Pet}$ array.

$$\texttt{Pet[] pets}$$

{{<box info>}}Note that elements in $\texttt{pets}$ won't know if they are a $\texttt{Cat}$ or $\texttt{Dog}$ so we can only access fields that belong to $\texttt{Pet}$ ($\texttt{name}$ and $\texttt{age}$).{{</box>}}

## Single Inheritance 

If a $\texttt{Pet}$ doesn't know if it is a $\texttt{Cat}$ or a $\texttt{Dog}$, we need to make sure that the procedure for accessing $\texttt{name}$ and $\texttt{age}$ is the same in both of its children.

### Fields

What if we put the parent ($\texttt{Pet}$) fields at the top of the memory structure.

```goat
   pet            cat             dog
+--------+   +-----------+   +-----------+
|  name* |   |   name*   |   |   name*   |
+--------+   +-----------+   +-----------+
|   age  |   |    age    |   |    age    |
+--------+   +-----------+   +-----------+
             | catBreed* |   | dogBreed* |
             +-----------+   +-----------+                          
```

{{<box info>}}Strings are just pointers (*) since there is no bound on string length.{{</box>}}

Now, no matter what a $\texttt{Pet pet}$ actually is, $\texttt{pet+0x0}$ is going to be $\texttt{name}$, and $\texttt{pet+0x8}$ is going to be $\texttt{age}$.

{{<box important>}}This is in our theoretical memory model, though the effect would be similar in your favorite programming language.{{</box>}}

### Non-virtual Functions

{{<box info>}}Think of non-virtual functions as just normal class methods. We will discuss virtual functions afterwards.{{</box>}}

Now, let's add a function to our class hierarchy.

$$
\begin{align*}
&\texttt{class Pet:}\newline
&\texttt{\qquad string name}\newline
&\texttt{\qquad int age}\newline
&\text{}\newline
&\texttt{\qquad void noise():}\newline
&\texttt{\qquad\qquad print("pet noises")}
\end{align*}
$$

Could we do the same thing?

```goat
   pet            cat             dog
+--------+   +-----------+   +-----------+
|  name* |   |   name*   |   |   name*   |
+--------+   +-----------+   +-----------+
|   age  |   |    age    |   |    age    |
+--------+   +-----------+   +-----------+
| noise* |   |   noise*  |   |   noise*  |
+--------+   +-----------+   +-----------+
             | catBreed* |   | dogBreed* |
             +-----------+   +-----------+


  .data
+-------+
| noise |
+-------+                                                           
```

{{<box info>}}$\texttt{.data}$ just stores the function definition.{{</box>}}

While this is correct, we can optimize based on the observation that method definitions are shared between all objects of the same class.

First, note that a method call $\texttt{pet.noise()}$ is syntactic sugar for $\texttt{Pet.noise(pet)}$ (object is passed as parameter). Since $\texttt{noise}$ is not specific to the object $\texttt{pet}$, it can be resolved at compile-time.

```goat
   pet            cat             dog
+--------+   +-----------+   +-----------+
|  name* |   |   name*   |   |   name*   |
+--------+   +-----------+   +-----------+
|   age  |   |    age    |   |    age    |
+--------+   +-----------+   +-----------+
             | catBreed* |   | dogBreed* |
             +-----------+   +-----------+


  .data
+-------+
| noise |
+-------+                                                           
```

{{<box info>}}Everytime $\texttt{Pet.noise(pet)}$ is called, just replace it (at compile-time) with the known address of $\texttt{Pet.noise}$ (in our memory model: $\texttt{.data+0x0}$).

---

For example:

$$
\begin{align*}
&\texttt{Cat cat}\newline
&\texttt{Dog dog}\newline
&\texttt{cat.noise()}\newline
&\texttt{dog.noise()}\newline
\newline
&\implies\newline
\newline
&\texttt{Cat cat}\newline
&\texttt{Dog dog}\newline
&\texttt{(.data+0x0)(cat)}\newline
&\texttt{(.data+0x0)(dog)}
\end{align*}
$$
{{</box>}}

We save space on every object holding a pointer to $\texttt{noise}$ ($\texttt{+0x8}$ size per object). 

### Virtual Functions

#### Introduction

The heart of inheritance is the concept of a virtual function. The idea is that an inherited method can take different definitions depending on the class hierarchy lineage.

Let's take our code from above and modify it so that $\texttt{noise}$ is a virtual function.

{{<box important>}}Other class data omitted.{{</box>}}

$$
\begin{align*}
&\texttt{class Pet:}\newline
&\texttt{\qquad void noise():}\newline
&\texttt{\qquad\qquad print("pet noises")}\newline
&\text{}\newline
&\texttt{class Cat extends Pet:}\newline
&\texttt{\qquad void noise():}\newline
&\texttt{\qquad\qquad print("meow")}\newline
&\text{}\newline
&\texttt{class Dog extends Pet:}\newline
&\texttt{\qquad void noise():}\newline
&\texttt{\qquad\qquad print("bark")}
\end{align*}
$$

The children can override the definition of a virtual method (in this case, $\texttt{noise}$).

{{<box info>}}Note that if the child does not provide a definition override, it keeps the parent's definition (recursively).

$$
\begin{align*}
&\texttt{class Fish extends Pet:}\newline
&\texttt{\qquad string color}\newline
&\text{}\newline
&\texttt{Fish fish}\newline
&\texttt{fish.noise()\qquad\thickspace\thickspace// "pet noises"}\newline
&\text{}\newline
&\texttt{class Betta extends Fish:}\newline
&\texttt{\qquad string bettaSpecies}\newline
&\text{}\newline
&\texttt{Betta betta}\newline
&\texttt{betta.noise()\qquad// "pet noises"}
\end{align*}
$$
{{</box>}}

The issue now is that we cannot know at compile-time which virtual function (in this case $\texttt{noise}$) to call.

$$
\begin{align*}
&\texttt{Pet[] pets = [Cat(), Dog()]}\newline
&\texttt{pets[0].noise()\qquad// Cat.noise() -> "meow"}\newline
&\texttt{pets[1].noise()\qquad// Dog.noise() -> "bark"}
\end{align*}
$$

{{<box important>}}
One (naive) solution is to again provide the function pointer as part of the object's structure.

```goat
     pet              cat              dog
+------------+   +------------+   +------------+
|    name*   |   |    name*   |   |    name*   |
+------------+   +------------+   +------------+
|     age    |   |     age    |   |     age    |
+------------+   +------------+   +------------+
| Pet.noise* |   | Cat.noise* |   | Dog.noise* |
+------------+   +------------+   +------------+
                 |  catBreed* |   |  dogBreed* |
                 +------------+   +------------+


    .data
+-----------+
| Pet.noise |
+-----------+
| Cat.noise |
+-----------+
| Dog.noise |
+-----------+                                                       
```

But again, objects of the same type share the same set of virtual functions (every $\texttt{Cat}$ is going to have the same $\texttt{noise=cat+0x10}$ value).
{{</box>}}

#### Vtable

Enter the [virtual method table](https://en.wikipedia.org/wiki/Virtual_method_table). Every object in a class hierarchy (with virtual methods) now also contains a pointer (virtual pointer) to a table (virtual table) containing all of the virtual methods.

```goat
     pet                cat               dog
+-------------+   +-------------+   +-------------+
| Pet.vtable* |   | Cat.vtable* |   | Dog.vtable* |
+-------------+   +-------------+   +-------------+
|    name*    |   |    name*    |   |    name*    |
+-------------+   +-------------+   +-------------+
|     age     |   |     age     |   |     age     |
+-------------+   +-------------+   +-------------+
                  |  catBreed*  |   |  dogBreed*  |
                  +-------------+   +-------------+


  Pet.vtable       Cat.vtable        Dog.vtable
+------------+   +------------+    +------------+
| Pet.noise* |   | Cat.noise* |    | Dog.noise* |
+------------+   +------------+    +------------+


    .data
+-----------+
| Pet.noise |
+-----------+
| Cat.noise |
+-----------+
| Dog.noise |
+-----------+                                                       
```

{{<box info>}}In this case, there is more memory used than in our naive approach. But for every additional virtual function, under the naive approach every object increases in size linearly.

With vtables, only the appropriate vtable sizes increase (and it is expected that there are significantly less vtables than object instances).{{</box>}}

$$
\begin{align*}
&\texttt{Pet[] pets = [Cat(), Dog()]}\newline
&\texttt{pets[0].noise()}\newline
&\texttt{pets[1].noise()}\newline
&\texttt{/* }\newline
&\texttt{ * vptr = pets[\textit{x}]+0x0}\newline
&\texttt{ * vtable = \\&vptr}\newline
&\texttt{ * noise = vtable+0x0}\newline
&\texttt{ \*/}
\end{align*}
$$

This also works with inheritance lineages. Suppose that $\texttt{Pet}$ is a child class of $\texttt{Thing}$.

{{<box important>}}Other class data omitted.{{</box>}}

$$
\begin{align*}
&\texttt{class Thing:}\newline
&\texttt{\qquad string class():}\newline
&\texttt{\qquad \qquad return "Thing"}\newline
&\text{}\newline
&\texttt{class Pet extends Thing:}\newline
&\texttt{\qquad string class():}\newline
&\texttt{\qquad \qquad return "Pet"}\newline
&\text{}\newline
&\texttt{class Dog extends Pet:}\newline
&\texttt{\qquad string class():}\newline
&\texttt{\qquad \qquad return "Dog"}\newline
&\text{}\newline
&\texttt{class Cat extends Pet:}\newline
&\texttt{\qquad string class():}\newline
&\texttt{\qquad \qquad return "Cat"}
\end{align*}
$$

```goat

      thing
+---------------+
| Thing.vtable* |
+---------------+

     pet                cat               dog
+-------------+   +-------------+   +-------------+
| Pet.vtable* |   | Cat.vtable* |   | Dog.vtable* |
+-------------+   +-------------+   +-------------+
|    name*    |   |    name*    |   |    name*    |
+-------------+   +-------------+   +-------------+
|     age     |   |     age     |   |     age     |
+-------------+   +-------------+   +-------------+
                  |  catBreed*  |   |  dogBreed*  |
                  +-------------+   +-------------+


  Thing.vtable
+--------------+
| Thing.class* |
+--------------+

  Pet.vtable       Cat.vtable        Dog.vtable
+------------+   +------------+    +------------+
| Pet.class* |   | Cat.class* |    | Dog.class* |
+------------+   +------------+    +------------+
| Pet.noise* |   | Cat.noise* |    | Dog.noise* |
+------------+   +------------+    +------------+


    .data
+-------------+
| Thing.class |
+-------------+
|  Pet.class  |
+-------------+
|  Cat.class  |
+-------------+
|  Dog.class  |
+-------------+
|  Pet.noise  |
+-------------+
|  Cat.noise  |
+-------------+
|  Dog.noise  |
+-------------+                                                     
```

{{<box important>}}The only constraint is that *parent virtual functions should be represented in memory before child virtual functions*. Doing so keeps an ordering across a lineage. In this case, the $\texttt{class}$ virtual function will always be at $\texttt{vtable+0x0}$ for any class that has that function.
{{</box>}}

{{<box info>}}If a class doesn't have that virtual function, the type checker would catch the error at compile time (i.e. $\texttt{Thing.noise()}$).{{</box>}}

## Multiple Inheritance

### Introduction

New example.

$$
\begin{align*}
&\texttt{class Printable:}\newline
&\texttt{\qquad string output}\newline
&\text{}\newline
&\texttt{\qquad void print():}\newline
&\texttt{\qquad \qquad print(output)}\newline
&\text{}\newline
&\texttt{class Stringable:}\newline
&\texttt{\qquad string value}\newline
&\text{}\newline
&\texttt{\qquad string toString():}\newline
&\texttt{\qquad \qquad return value}\newline
&\text{}\newline
&\texttt{class Person extends Printable, Stringable:}\newline
&\texttt{\qquad string name}\newline
&\text{}\newline
&\texttt{\qquad void print():}\newline
&\texttt{\qquad \qquad print(output)}\newline
&\texttt{\qquad \qquad print(value)}\newline
&\texttt{\qquad \qquad print(name)}\newline
&\text{}\newline
&\texttt{\qquad string toString():}\newline
&\texttt{\qquad \qquad return output + value + name}
\end{align*}
$$

Ok, so now we can have two parents. How can we possibly represent this in memory?

### The Problem

What ordering can we keep?

```goat
      printable               stringable               person
+-------------------+   +--------------------+   +----------------+
| Printable.vtable* |   | Stringable.vtable* |   | Person.vtable* |
+-------------------+   +--------------------+   +----------------+
|      output*      |   |       value*       |   |     output*    |
+-------------------+   +--------------------+   +----------------+
                                                 |     value*     |
                                                 +----------------+
                                                 |      name*     |
                                                 +----------------+


  Printable.vtable        Stringable.vtable         Person.vtable
+------------------+   +---------------------+   +-----------------+
|  Printable.print |   | Stringable.toString |   |   Person.print  |
+------------------+   +---------------------+   +-----------------+
                                                 | Person.toString |
                                                 +-----------------+
```

If we put $\texttt{print}$ first in the $\texttt{Person}$ vtable, $\texttt{toString}$ calls and $\texttt{value}$ accesses fail.

{{<box warning>}}
$$
\begin{align*}
&\texttt{Stringable[] stringables = [Stringable(), Person()]}\newline
&\texttt{Stringable[0].toString()}\newline
&\texttt{Stringable[1].toString()}\newline
&\texttt{/* }\newline
&\texttt{ * vptr = stringables[\textit{x}]+0x0}\newline
&\texttt{ * vtable = \\&vptr}\newline
&\texttt{ * toString = vtable+???}\newline
&\texttt{ * // 0x0 in Stringable.vtable; 0x8 in Person.vtable}\newline
&\texttt{ \*/}\newline
&\newline
&\texttt{Stringable[0].value}\newline
&\texttt{Stringable[1].value}\newline
&\texttt{/* }\newline
&\texttt{ * value = stringables[\textit{x}]+???}\newline
&\texttt{ * // 0x0 in Stringable layout; 0x8 in Person layout}\newline
&\texttt{ \*/}
\end{align*}
$$
{{</box>}}

### The Solution
#### Part 1: Contiguous Layout

```goat
      printable               stringable
+-------------------+   +--------------------+
| Printable.vtable* |   | Stringable.vtable* |
+-------------------+   +--------------------+
|      output*      |   |       value*       |
+-------------------+   +--------------------+


person -------> +---------------------------+
                |  Person.Printable.vtable* |
                +---------------------------+
                |          output*          |
stringable ---> +---------------------------+
                | Person.Stringable.vtable* |
                +---------------------------+
                |           value*          |
                +---------------------------+
                |           name*           |
                +---------------------------+


  Printable.vtable        Stringable.vtable
+------------------+   +---------------------+
|  Printable.print |   | Stringable.toString |
+------------------+   +---------------------+

Person.Printable.vtable ----> +-----------------+
                              |   Person.print  |
Person.Stringable.vtable ---> +-----------------+
                              | Person.toString |
                              +-----------------+                   
```

> The key idea is this: whenever we pass a $\texttt{Person}$ where $\texttt{Stringable}$ is expected, it is replaced at compile-time with the $\texttt{stringable}$ address $\texttt{person+0x10}$.

{{<box info>}}
$$\begin{align*}
\texttt{\\&person}&\neq\texttt{\\&cast<Stringable\>(person)}\newline
\texttt{\\&person+0x10}&=\texttt{\\&cast<Stringable\>(person)}
\end{align*}
$$
{{</box>}}

```goat
stringable ---> +---------------------------+
                | Person.Stringable.vtable* |
                +---------------------------+
                |           value*          |
                +---------------------------+
                |           name*           |
                +---------------------------+


Person.Stringable.vtable ---> +-----------------+
                              | Person.toString |
                              +-----------------+                   
```

$$
\begin{align*}
&\texttt{Person person = Person()}\newline
&\texttt{Stringable[] stringables = [Stringable(), person] // person+0x10}\newline
&\texttt{Stringable[0].toString()}\newline
&\texttt{Stringable[1].toString()}\newline
&\texttt{/* }\newline
&\texttt{ * vptr = stringables[\textit{x}]+0x0}\newline
&\texttt{ * vtable = \\&vptr}\newline
&\texttt{ * toString = vtable+0x0}\newline
&\texttt{ \*/}\newline
\newline
&\texttt{Stringable[0].value}\newline
&\texttt{Stringable[1].value}\newline
&\texttt{/* }\newline
&\texttt{ * value = stringables[\textit{x}]+0x8}\newline
&\texttt{ \*/}
\end{align*}
$$

Relative memory location is consistent!

One problem: how does $\texttt{Person.toString}$ access $\texttt{output}$ (a field derived from $\texttt{Printable}$)?

{{<box warning>}}$
\texttt{string Person.toString():}\newline
\texttt{\qquad return output + value + name}\newline
\texttt{\qquad // output is at person+0x8}\newline
\texttt{\qquad // output is lost when using person+0x10}\newline
${{</box>}}

#### Part 2: Top Offset

```goat
person -------> +---------------------------+<-+
                |  Person.Printable.vtable* |  |
                +---------------------------+  |0x10
                |          output*          |  |
stringable ---> +---------------------------+<-+
                | Person.Stringable.vtable* |
                +---------------------------+
                |           value*          |
                +---------------------------+
                |           name*           |
                +---------------------------+


  Printable.vtable        Stringable.vtable
+------------------+   +---------------------+
|        0x0       |   |         0x0         |
+------------------+   +---------------------+
|  Printable.print |   | Stringable.toString |
+------------------+   +---------------------+

Person.Printable.vtable ----> +-----------------+
                              |       0x0       |
                              +-----------------+
                              |   Person.print  |
Person.Stringable.vtable ---> +-----------------+
                              |      -0x10      |
                              +-----------------+
                              | Person.toString |
                              +-----------------+                   
```

We've added two new entries to the $\texttt{Person.vtable}$: the top offsets.

> Recall that $\texttt{person.toString()}$ is syntactic sugar for $\texttt{Person.toString(person)}$. So now, when we call functions in the vtable, instead of passing the object address, we pass the object address *plus the top offset* (which gives a pointer to the original object). This is called **this pointer adjustment**.

$$\begin{align*}
&\texttt{Person person = Person()}\newline
&\texttt{Stringable[] stringables = [Stringable(), person] // person+0x10}\newline
&\texttt{Stringable[0].toString()}\newline
&\texttt{Stringable[1].toString()}\newline
&\texttt{/* }\newline
&\texttt{ * vptr = stringables[\textit{x}]+0x0}\newline
&\texttt{ * vtable = \\&vptr}\newline
&\texttt{ * toString = vtable+0x8}\newline
&\texttt{ * topOffset = \*(vtable+0x0)}\newline
&\texttt{ * this = stringables[\textit{x}]+topOffset}\newline
&\texttt{ * toString(this)}\newline
&\texttt{ \*/}
\end{align*}$$

Now, $\texttt{output}$ is at $\texttt{this+0x8}$.

{{<box info>}}Observe that this works with $\texttt{Printable}$.

$$\texttt{\\&person}=\texttt{\\&cast<Printable\>(person)}$$

The $\texttt{Person.Printable}$ top offset is $\texttt{0x0}$, so the **this pointer adjustment** has no effect.
{{</box>}}

{{<box tip>}}
In general, if $\texttt{Child extends A, B, C, D}$:

```goat
Child ---> +-----------------+<-+--+--+--+
           |  Child.vtable*  |  ^  ^  ^  ^
           +-----------------+  |a |  |  |
           |    Child.data   |  |  |  |  |
           |        …        |  v  |  |  |
A -------> +-----------------+<-+  |b |  |
           | Child.A.vtable* |     |  |  |
           +-----------------+     |  |c |
           |   Child.A.data  |     |  |  |
           |        …        |     v  |  |
B -------> +-----------------+<----+  |  |d
           | Child.B.vtable* |        |  |
           +-----------------+        |  |
           |   Child.B.data  |        |  |
           |        …        |        v  |
C -------> +-----------------+<-------+  |
           | Child.C.vtable* |           |
           +-----------------+           |
           |   Child.C.data  |           |
           |        …        |           v
D -------> +-----------------+<----------+
           | Child.D.vtable* |            
           +-----------------+
           |   Child.D.data  |
           |        …        |
           +-----------------+

Child.vtable -----> +-------------------+
                    |        0x0        |
                    +-------------------+
                    |  Child.functions  |
                    |         …         |
Child.A.vtable ---> +-------------------+
                    |        -a         |
                    +-------------------+
                    | Child.A.functions |
                    |         …         |
Child.B.vtable ---> +-------------------+
                    |        -b         |
                    +-------------------+
                    | Child.B.functions |
                    |         …         |
Child.C.vtable ---> +-------------------+
                    |        -c         |
                    +-------------------+
                    | Child.C.functions |
                    |         …         |
Child.D.vtable ---> +-------------------+
                    |        -d         |
                    +-------------------+
                    | Child.D.functions |
                    |         …         |
                    +-------------------+                           
```
{{</box>}}
