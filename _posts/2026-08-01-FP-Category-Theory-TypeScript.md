---
layout: post
title: "Functional Programming: Category Theory: TypeScript"
date: 2026-08-01
math: true
---

# Functional Programming: Category Theory: TypeScript

## Introduction

### Another generalization of functions

While lambda calculus tried to generalize functions to apply to computation by reducing it to syntactic machinations, category theory generalizes functions by reducing its intensional[^1] definition: a function is a mapping between two sets (such that...), often drawn as an arrow between two circles with points in the circles symbolizing elements; a morphism in a category of objects is an arrow between two objects.

### Definitions

#### Categories

A **category** $\mathcal C$ consists of:

- a collection of **objects** $A,B,C,\ldots$,
- a collection of **morphisms** $f$ between objects, with a **domain** object and **codomain** object for every morphism, written $f : A \to B$
- an associative operation of **composition** for morphisms with compatible domain and codomain: if $f : A \to B$ domain is the same object as codomain of $g : B \to C$ then they compose to give morphism $g \circ f : A \to C$. Associativity means $h \circ (g \circ f) = (h \circ g) \circ f$ for all compotible morphisms, so brackets may be omitted.
- for every object $A$, there is an **identity morphism** $\operatorname{id}_A : A \to A$ such that $f \circ \operatorname{id}_A = f$ and $\operatorname{id}_A \circ g = g$ for compatible morphisms.

A morphism is sometimes also called an **arrow**.

## Footnotes
[^1]: Intensional: without reference to its internals.
