---
layout: post
title: "Functional Programming: Untyped Lambda Calculus"
date: 2025-11-01
---

# Functional Programming: Untyped Lambda Calculus

*Part of a series on the theory and practice of functional programming.*

## Introduction

### Contrast between C and Haskell 

The Haskell programming language is very different in its *paradigm* to the common "practical" languages based upon C; many of those languages can be understood as extensions of C (with classes, with automatic reference counting, with borrow checking, etc.), a purely functional programming language requires a different perspective altogether. 

C was developed as a language for UNIX and hence it closely matches the abilities of the OS, whereas Haskell developed as an implementation of theoretical theory. To understand C design decisions, you should understand UNIX processes and POSIX system calls, and to understand the design decisions of functional programming languages you should understand lambda calculus and type theory. While C is "bottom-up", from metal to abstractions in its design, Haskell is "top-down", from abstractions to metal.

For this reason, if you can already program in C, learning Haskell is more stimulating than diving into Go, Rust, Swift, C++, Carbon, Zig, D, etc. It provides a different set of tools, rather than a different version of a tool you already have; the situations where C shines (low-level control, transparent implementation) and Haskell shines (concise, transparent references, strong correctness) are close to distinct.

### What is a Function?

In C, if you remember (future link), a function call creates a new stack frame and the parameters fed in are copies of the outer scope values (pass by value). The program does some procedure with allocated values in that frame, returns a value to the calling stack frame addressed above, and frees the memory space of the finished stack frame. In that sense, a function in C is really a *procedure* isolated from the functions/procedures above it. This is why C is called a *procedural* language, a subclass of imperative programming. This isolated scoping of the variables is called lexical (or static) scope, it forces the use of pointers as input parameters for changes that persist past the scope. 

In Haskell, a function is an expression of its parameters that may be evaluated (given concrete input values, get the output) or reduced (given parameters as symbols, may be reduced to just those symbols). This is more similar to mathematics; a function is a many-to-one mapping between the domain and codomain, i.e. the functions are a particular class of subsets of the domain times the codomain, denoted as $C^D$. 
This is a very static object; the definition of such a *pure* function has no procedure to complete, just names with expressions. Each name can only defined once in a scope, like in mathematics, which means there is no particular order to evaluate or reduce a sequence of expressions, they will always mean the same thing logically and we can write an expression using another to stack these definitions. This purity is called *referential transparency* ([HPFP](#HFPF) p.30).

Haskell's scope is also lexical/static, but in some way it is a more extreme example where variables are never redefined in the function definition. You can use the same name outside the scope and be confident it refers to a totally different object, this is sometimes called 'shadowing'.

## Untyped Lambda Calculus

Untyped lambda calculus is a Turing complete model of computing[^1]. Historically, Turing machines have been found easier to implement but lambda calculus has been found easier to reason about. Lambda calculus makes the process of function reduction and evaluation explicit, in fact its objects are only functions and variables. Since lambda calculus seems so abstract from what we'd first consider to be computation, it is helpful to know its history.
Church looked to study functions in purely syntactic terms, just as propositional logic was formalized and studied as application of syntax rules. Functions in mathematics are defined through semantics of sets, rather than pure syntax rules. Lambda calculus is thinking of "functions-as-rules" instead of the traditional "functions-as-sets" (SEP, sec. 1.2).
It should then be considered in this light as a 'game' of syntax manipulation, before looking for any semantics (or model) of it. The connection to computation was discovered only after the formalism. 

### Definitions

#### Expressions

The lambda calculus has three types of expressions/terms:

- variables, denoted by some identifier of which there are infinitely many available. 
- abstractions, consist of some variable, say `x`, and some expression in the lambda calculus, potentially containing `x`. We'll denote it as `\x.M` where `x` is the variable and `M` is an expression[^2]. 
- applications, which is an ordered pair of expressions `(M N)`. 

Examples of lambda terms: `x`, `\x.x`, `(x y)`, `(x \y.y)`, `(x (\y.y x))`, etc.

#### Left Associativity

To avoid writing too many parenthesis, the convention is that: 

- two expressions in sequence are an application: `M N` means `(M N)`,
- more than two expressions in a sequence are left associative unless indicated otherwise, so a sequence of expressions like `M N L` is meant as `((M N) L)`,
- multiple nested applications are written with bound variables in a sequence: `\x\y\z.M` or even `\xyz.M` means `\x.(\y.(\z.M))`,
- abstraction is lower precedence than application, so `\x.M N` means `\x.(M N)` unless parenthesized otherwise.

#### Alpha Equivalence

Breaking slightly from the pure syntactics, and approaching semantics, is the notion of alpha equivalence, without which lambda calculus would just become a naive rewriting/macro system. There may be situations where you have a variable used as a *bound* variable in an abstraction, but if we wanted to do a naive replacement of every instance of x with another expresssion, then a variable that was not bound by a lambda abstraction may not caught up in one (this situation is called variable capture). 
A simple example would be substituting variable `y` for `x` in expression `\y.x`, naive substituting gives `\y.y` as the result but it should be `\z.y`, the meaning of the term changed with the variable capture; the abstraction meant `x` whatever `y` is, and got replaced to mean return whatever the input is. To avoid this we need labels for bound variables and free variables.

Bound variables of lambda expressions are easily defined inductively as:
```txt
BV(x) = {}
BV(\x.M) = {x} + BV(M)
BV(M N) = BV(M) + BV(N)
```
where `+` is set union. Free variables are the complement, i.e. variables that are not bound in each expression and can be given similar definition:
```txt
FV(x) = {x}
FV(\x.M) = FV(M) - {x}
FV(M N) = FV(M) + FV(N)
```

*Alpha equivalence* means bounded variables may be replaced without changing the meaning of the lambda expression. In fact, they often have to be renamed in reductions.

#### Beta Reduction

A lambda expression represents a computation yet to be attempted, like a recipe yet to be performed. The order of the steps to be taken is not yet specified, doing a computation is analogous to a *reduction*. Given a lambda expression, *beta-reduction* may be applied to applications within it if the first term is an abstraction:
```
(\x.M) N → M[x := N]
```
where `M[x := N]` means every instance of `x` variable in `M` is replaced by `N` expression.

As shown in the previous example, `(\x\y.x) y → \y.y` is an instance of variable capture that changes meaning of the function and is treated as incorrect. So instead beta-reduction is only acceptable when the bounded variables in `M` are distinct from the free variables in `N`. Otherwise, we use alpha equivalence to make the two distinct and go ahead with the reduction:

```
(\x\y.x) y → (\x\z.x) y → \z.y 
```
is a correct reduction.

"A computation therefore consists of an initial lambda expression \[...\] plus a finite sequence of lambda terms, each deduced from the preceding term by one application of beta reduction." (HPFP, p. 36) Note that there is a choice of reduction strategy by which applications one chooses to reduce first.

#### Eta Equivalence

Eta equivalence is defined as regarding 

`\x.(M x)` where `x` is not free in `M` (ensuring `x` is not binding instances of `x` in `M`), 

as the same as 

`M`.

Eta equivalence is implied by the definition of beta reduction (TTFP, p.39); if we only care about the behavior of functions in beta reduction, then we only care about abstractions up to eta equivalence, seeing how it has exactly the same behavior in applicaitons: 

`\x.(M x) N → M N`.

How eta equivalence is treated depends on the context; it is usually not thought of as a computational step but instead expressing the same computation like alpha equivalence.

### Reduction

"There are two approaches to evaluating \[nested\] function applications. For both, the function expression is evaluated to return a function. Next, all occurences of the function’s bound variable in the function’s body expression are replaced by either

- the value of the argument expression, or
- the unevaluated argument expression

Finally, the function body expression is then evaluated. The first approach is called applicative order and is like \[...\] ‘call by value’: the actual parameter expression is
evaluated before being passed to the formal parameter. The second approach is called normal order and is like ‘call by name’ \[...\]: the actual parameter expression
is not evaluated before being passed to the formal parameter." (FPLC)

More formally, if `M → M'` then 

- `(M N) → (M' N)`
- `(N M) → (N M')`
- `\x.M → \x.M'`

are all valid beta-reductions (TTFP, p.35). The normal order is when we always prefer to use the first beta-reduction when evaluating nested applications, and the applicative order is when we always choose the second.

A lambda expression being in normal form is when there are no unevaluated applicative terms `(\x.M N)`, called a beta-redux, so normal form is analogous to 'fully evaluated'. It is a theorem in lambda calculus that the normal order of evaluation will always find a normal form if it exists. I.e. lambda expressions that *can* terminate will terminate with the normal order strategy. This is not true with applicative order, as intuitively one may have an expression that never terminates but is not used by an outer abstraction anyway. It is also true that any normal form found through reductions is unique, so choice of reduction will not change the final result as long as both terminate.

For this reason, Haskell and most functional programming languages use the normal order, or more accurately lazy evaluation which is an optimization of it. A neat advantage is that one can define infinite data values, but if the outer abstraction only uses a finite subset then it will terminate. 

### Interesting Lambda Expresssions

Non-terminating lambda expressions are a great source of study. *Combinators* are lambda abstractions with no free variables, so they just combine arguments.

The self-application combinator is 

`S := \x.(x x)`

where whatever argument is given, it applies it twice over. E.g. `S \y.xy → \y.(xy xy)` for `x` free.

If we apply it to itself we get a non-terminating expresssion:

`S S → S S`.

Fixed point combinators are so that any function applied to it is a fixed point of that function: `(Y g) = g (Y g)`.

Most well-known, and similar to the self-application combinator, is the Y combinator: 

`Y := \f. \x.f(xx) \x.f(xx)`

where

```txt
Y g = \x.g(x x) \x.g(x x)
    = g (\x.g(x x) \x.g(x x))
    = g (Y g)
```

The Y combinator is commonly used in functional programming to be able to write recursive functions.

The theory of lambda calculus is deep and growing, but for the rest of this article we return to Haskell.

## Haskell Implementation

### Currying and Evaluation

As shown in the convention of left associativity (LINK TODO), multiple nested abstractions `\x(\y(\z.M))` mean the same as a multi-argument function `\xyz.M`. This is called currying, and is the standard way to deal with functions in functional programming. 

Haskell's reduction strategy for lambda expressions is left-to-right, or some may say outer-in, this has some advantages (and disadvantages) and is different than most imperative languages that work out the value of an argument before using it in a procedure. The technical term for left-to-right is *normal* reduction as opposed to the in-out being called *applicative* reduction.

This has implications for syntax straight away: in the Haskell REPL, 

```txt
ghci> 2 + 2
4
ghci> (+) 2 2
4
ghci> (+ 2) 2
4
```

The operator `+` may be considered as some shortcut for a lambda abstraction, it accepts two arguments by currying. The first line is the common infix notation, the second line explicitly invokes the `+` lambda abstraction and applies it twice over, the second line explicitly invokes the lambda abstraction of a single argument `(+ 2)` that just adds `2` to any argument[^3].

The same idea works for negative `-` expression, it is a lambda function with a single argument, so we can expect `(+) 2 (- 2)` to work, but unfortuntely `(+) 2 - 2` will not because the outer lambda abstraction `(+ 2)` expects an integer as an arguement but gets `-` instead, writing `(- 2)` alters the left-to-right evaluation strategy to feed an integer for `(+ 2)`.

In actuality, Haskell's operators also have precedence levels that influence evaluation; they're slightly different than just functions, but the above is still correct because `+` and `-` have the same precedence (6 out of 10). 

### Lambdas

A lambda abstraction in Haskell is defined with syntax using `->` instead of a dot:

```hs
\x -> x + 2
```

and can be applied:

```txt
ghci> (\x -> x + 2) 4
6
```

Currying is explicit, i.e. without the conventional nesting:

```hs
ghci> (\x -> \y -> x + y + 2) 3 2
7
```

And nesting expresssions is simple, just be careful about the left associative convention:

```hs
ghci> (\z -> [z])((\x -> \y -> x + y + 2) 3 2)
[7]
```

without the parentheses around the second expression, you create an expression of type `[\x ->\y -> M]` which is a list, not a function that accepts integers so the application is not defined.

To illustrate the normal/lazy evaluation strategy, we can import the built-in module with the `fix` fixed point combinator.

If we execute something like 

```hs
ghci> import Data.Function
ghci> fix (\x -> x + 1)

```
then it keeps evaluating until interrupted, but if we have it as an unused value then it is ignored:

```hs
ghci> (\x -> \y -> x) 2 (fix (\x -> x + 1))
2
```

We talk about types next, which helps to see why we needed parenthesis above to avoid creating a list containing a function, and instead have the list contain the results of the function. It's a *type error*.


## Footnotes
[^1]: Adding type restrictions to lambda calculus stops it being Turing complete. 
[^2]: Abstractions are meant to be generalized functions, so 'lambda functions' is synonymous. Lambda functions are often called "anonymous functions" since they're not given an identifier to be called.
[^3]: Via Church numeral encodings, integers themselves may be considered as lambda abstractions, so the infix notation `2 + 2` is really `(((2) +) 2)` nested abstractions that Haskell then evaluates (normalizes the expression) to the lambda abstraction with encoding `4`. For convenience and type checking Haskell just treats integers as a base type.


## References

<a id="HFPF"></a>
HFPF: Haskell Programming From First Principles C. Allen, J. Moronuki, 2016

Type Theory & Functional Programming, S. Thompson, 1999 (TTFP)
The Lambda Calculus, J. Alama, Stanford Encyclopedia of Philosophy, 2023 (SEP)
An Introduction to Functional Programming through Lambda Calculus, G. Michaelson, 2011 (FPLC)
Real World Haskell, B. O'Sullivan, D. Stewart, J. Goerzen, 2008 (RWH)
Highlights of the History of the Lambda Calculus [link](https://lawrencecpaulson.github.io/papers/Rosser-Lambda-Calculus.pdf)

