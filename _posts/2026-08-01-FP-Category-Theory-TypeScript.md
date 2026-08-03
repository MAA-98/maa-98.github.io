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
- a collection of **morphisms** $f$ between objects, a **domain** object and **codomain** object for every morphism, giving $f : A \to B$
- an operation of **composition** for compatible morphisms,
- an **identity morphism** for every object,

satisfying the laws of associativity and identity.

The object $A$ is the **domain** or **source** of $f$, and $B$ is the **codomain** or **target** of $f$.

A morphism is sometimes also called an **arrow**.

#### Hom-Sets

For any two objects $A$ and $B$, the collection of morphisms from $A$ to $B$ is denoted:

\[
\operatorname{Hom}_{\mathcal C}(A,B)
\]

or, when the category is clear from context:

\[
\operatorname{Hom}(A,B)
\]

Thus, if

\[
f : A \to B
\]

then:

\[
f \in \operatorname{Hom}(A,B)
\]

The collection $\operatorname{Hom}(A,B)$ is called a **hom-set**. More generally, it may be a class rather than a set, depending on the size of the category.

#### Composition

Given morphisms:

\[
f : A \to B
\]

and:

\[
g : B \to C
\]

we may compose them to obtain a morphism:

\[
g \circ f : A \to C
\]

The notation $g \circ f$ means “first apply $f$, then apply $g$.”

Composition therefore has the form:

\[
\circ :
\operatorname{Hom}(B,C)
\times
\operatorname{Hom}(A,B)
\to
\operatorname{Hom}(A,C)
\]

where:

\[
(g,f)
\mapsto
g \circ f
\]

The order of the two hom-sets reflects the notation: $g \circ f$ applies $f$ first and $g$ second.

For example:

\[
\mathrm{OCR} : \mathrm{PDF} \to \mathrm{TXT}
\]

and:

\[
\mathrm{ParseJSON} : \mathrm{TXT} \to \mathrm{JSON}
\]

compose to give:

\[
\mathrm{ParseJSON} \circ \mathrm{OCR}
:
\mathrm{PDF} \to \mathrm{JSON}
\]

Composition is defined only when the codomain of the first morphism agrees with the domain of the second:

\[
f : A \to B,
\qquad
g : B \to C
\]

If the types do not match, the composition is not defined. For example, if:

\[
h : \mathrm{IMAGE} \to \mathrm{TXT}
\]

then:

\[
h \circ \mathrm{OCR}
\]

is not defined, because $\mathrm{OCR}$ has codomain $\mathrm{TXT}$, while $h$ has domain $\mathrm{IMAGE}$.

#### Paths

A **path** is a finite sequence of composable morphisms.

For example:

\[
A
\xrightarrow{f}
B
\xrightarrow{g}
C
\xrightarrow{h}
D
\]

is the path:

\[
(f,g,h)
\]

More formally, this path is an element of the Cartesian product:

\[
\operatorname{Hom}(A,B)
\times
\operatorname{Hom}(B,C)
\times
\operatorname{Hom}(C,D)
\]

The path has source $A$ and target $D$.

Every path determines a composite morphism:

\[
h \circ g \circ f : A \to D
\]

Thus, the path and its composite are distinct concepts:

- the path records the sequence of morphisms;
- the composite records the resulting single morphism.

For example:

\[
(\mathrm{OCR},\mathrm{ParseJSON})
\]

records a two-step transformation:

\[
\mathrm{PDF}
\xrightarrow{\mathrm{OCR}}
\mathrm{TXT}
\xrightarrow{\mathrm{ParseJSON}}
\mathrm{JSON}
\]

while:

\[
\mathrm{ParseJSON} \circ \mathrm{OCR}
:
\mathrm{PDF} \to \mathrm{JSON}
\]

represents the same transformation as one morphism.

Different paths may have the same source and target:

\[
(f,g)
\quad\text{and}\quad
h
\]

may both induce morphisms from $A$ to $C$:

\[
g \circ f : A \to C,
\qquad
h : A \to C
\]

They are not automatically equal. The category may or may not specify that:

\[
g \circ f = h
\]

#### Identity Morphisms

For every object $A$, there is a distinguished morphism:

\[
\operatorname{id}_A : A \to A
\]

called the **identity morphism** on $A$.

The identity morphism represents doing nothing to an object. In a data-transformation category:

\[
\operatorname{id}_{\mathrm{PDF}} :
\mathrm{PDF} \to \mathrm{PDF}
\]

is the transformation that leaves a PDF unchanged.

Identities must satisfy:

\[
f \circ \operatorname{id}_A = f
\]

for every:

\[
f : A \to B
\]

and:

\[
\operatorname{id}_B \circ f = f
\]

For example:

\[
\mathrm{OCR}
\circ
\operatorname{id}_{\mathrm{PDF}}
=
\mathrm{OCR}
\]

and:

\[
\operatorname{id}_{\mathrm{TXT}}
\circ
\mathrm{OCR}
=
\mathrm{OCR}
\]

#### Associativity

Composition must be **associative**.

Given morphisms:

\[
f : A \to B
\]

\[
g : B \to C
\]

\[
h : C \to D
\]

we require:

\[
h \circ (g \circ f)
=
(h \circ g) \circ f
\]

This means that the placement of parentheses does not affect the resulting composite morphism.

The expressions:

\[
h \circ (g \circ f)
\]

and:

\[
(h \circ g) \circ f
\]

may therefore both be written simply as:

\[
h \circ g \circ f
\]

Associativity concerns the composition of morphisms. It does not necessarily mean that the paths:

\[
(f,g,h)
\]

and:

\[
((f,g),h)
\]

are the same data structure; rather, both path groupings yield the same composite morphism.

#### Category Laws

A category $\mathcal C$ must satisfy the following laws.

**Identity law:**

\[
f \circ \operatorname{id}_A = f
\]

and:

\[
\operatorname{id}_B \circ f = f
\]

for every:

\[
f : A \to B
\]

**Associativity law:**

\[
h \circ (g \circ f)
=
(h \circ g) \circ f
\]

for all composable morphisms:

\[
f : A \to B,
\qquad
g : B \to C,
\qquad
h : C \to D
\]

These laws, together with objects, morphisms, domains, codomains, and composition, define a category.

#### Diagrammatic Representation

A category can be represented by a directed diagram:

\[
A
\xrightarrow{f}
B
\xrightarrow{g}
C
\]

The diagram indicates the morphisms:

\[
f : A \to B
\]

and:

\[
g : B \to C
\]

and therefore also the composite:

\[
g \circ f : A \to C
\]

which may be shown explicitly:

\[
\begin{array}{ccccc}
A & \xrightarrow{f} & B & \xrightarrow{g} & C \\
& \searrow_{g\circ f} & & &
\end{array}
\]

More conventionally:

\[
\require{AMScd}
\begin{CD}
A @>{f}>> B @>{g}>> C \\
@V{g\circ f}VV @. @. 
\end{CD}
\]

A diagram does not necessarily display every composite morphism. Composites may be implicit in the paths through the diagram.

#### Commutative Diagrams

A diagram is **commutative** when any two paths with the same source and target determine the same composite morphism.

For example:

\[
\begin{CD}
A @>{f}>> B \\
@V{h}VV @VV{g}V \\
C @>>{k}> D
\end{CD}
\]

commutes when:

\[
g \circ f = k \circ h
\]

There are two paths from $A$ to $D$:

\[
A \xrightarrow{f} B \xrightarrow{g} D
\]

and:

\[
A \xrightarrow{h} C \xrightarrow{k} D
\]

The diagram commutes when both paths have the same composite morphism.

In a data-transformation system, this could express that two different pipelines are considered equivalent:

\[
\mathrm{Validate}
\circ
\mathrm{ParseJSON}
\circ
\mathrm{OCR}
=
\mathrm{DirectExtraction}
\]

Both sides would be morphisms:

\[
\mathrm{PDF} \to \mathrm{ValidJSON}
\]

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
