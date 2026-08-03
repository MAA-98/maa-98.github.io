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

#### Hom-Sets

For any two objects $A$ and $B$, the collection of morphisms from $A$ to $B$ is denoted:

$$
\operatorname{Hom}_{\mathcal C}(A,B)
$$

or, when the category is clear from context:

$$
\operatorname{Hom}(A,B)
$$

Thus, if

$$
f : A \to B
$$

then:

$$
f \in \operatorname{Hom}(A,B)
$$

The collection $\operatorname{Hom}(A,B)$ is called a **hom-set**. More generally, it may be a class rather than a set, depending on the size of the category.

#### Paths

A **path** is a finite sequence of composable morphisms.

For example:

$$
A
\xrightarrow{f}
B
\xrightarrow{g}
C
\xrightarrow{h}
D
$$

is the path:

$$
(f,g,h)
$$

More formally, this path is an element of the Cartesian product:

$$
\operatorname{Hom}(A,B)
\times
\operatorname{Hom}(B,C)
\times
\operatorname{Hom}(C,D)
$$

The path has source $A$ and target $D$.

Every path determines a composite morphism:

$$
h \circ g \circ f : A \to D
$$

Thus, the path and its composite are distinct concepts:

- the path records the sequence of morphisms;
- the composite records the resulting single morphism.

For example:

$$
(\mathrm{OCR},\mathrm{ParseJSON})
$$

records a two-step transformation:

$$
\mathrm{PDF}
\xrightarrow{\mathrm{OCR}}
\mathrm{TXT}
\xrightarrow{\mathrm{ParseJSON}}
\mathrm{JSON}
$$

while:

$$
\mathrm{ParseJSON} \circ \mathrm{OCR}
:
\mathrm{PDF} \to \mathrm{JSON}
$$

represents the same transformation as one morphism.

Different paths may have the same source and target:

$$
(f,g)
\quad\text{and}\quad
h
$$

may both induce morphisms from $A$ to $C$:

$$
g \circ f : A \to C,
\qquad
h : A \to C
$$

They are not automatically equal. The category may or may not specify that:

$$
g \circ f = h
$$

#### Diagrammatic Representation

A category can be represented by a directed diagram:

$$
A
\xrightarrow{f}
B
\xrightarrow{g}
C
$$

The diagram indicates the morphisms:

$$
f : A \to B
$$

and:

$$
g : B \to C
$$

and therefore also the composite:

$$
g \circ f : A \to C
$$

which may be shown explicitly:

$$
\begin{array}{ccccc}
A & \xrightarrow{f} & B & \xrightarrow{g} & C \\
& \searrow_{g\circ f} & & &
\end{array}
$$

More conventionally:

$$
\require{AMScd}
\begin{CD}
A @>{f}>> B @>{g}>> C \\
@V{g\circ f}VV @. @. 
\end{CD}
$$

A diagram does not necessarily display every composite morphism. Composites may be implicit in the paths through the diagram.

#### Commutative Diagrams

A diagram is **commutative** when any two paths with the same source and target determine the same composite morphism.

For example:

$$
\begin{CD}
A @>{f}>> B \\
@V{h}VV @VV{g}V \\
C @>>{k}> D
\end{CD}
$$

commutes when:

$$
g \circ f = k \circ h
$$

There are two paths from $A$ to $D$:

$$
A \xrightarrow{f} B \xrightarrow{g} D
$$

and:

$$
A \xrightarrow{h} C \xrightarrow{k} D
$$

The diagram commutes when both paths have the same composite morphism.

In a data-transformation system, this could express that two different pipelines are considered equivalent:

$$
\mathrm{Validate}
\circ
\mathrm{ParseJSON}
\circ
\mathrm{OCR}
=
\mathrm{DirectExtraction}
$$

Both sides would be morphisms:

$$
\mathrm{PDF} \to \mathrm{ValidJSON}
$$

#### A Category of Data Transformations

A category of data transformations might be defined as follows:

- objects are data types, schemas, or domains;
- morphisms are valid data transformations;
- the domain of a morphism is its input type;
- the codomain of a morphism is its output type;
- composition is sequential execution of compatible transformations;
- identities are transformations that leave data unchanged.

## Footnotes
[^1]: Intensional: without reference to its internals.
