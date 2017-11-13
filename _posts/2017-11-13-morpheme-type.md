---
layout: post
title:  "A Morpheme Based Type Notation"
date:   2017-11-13 01:19:00 -0700
categories: plt, loli
---

# Why
To keep the purity of S-Exp and readability in LoLi while giving LoLi the power to express complicated types.

# How
The idea came from inflection of natural languages, where we have bound morphemes that don't have meaning by themselves but express the inflectional changes. Therefore, we can adopt the idea into programming languages design, by defining several 'morpheme's, we can then perform type-level combinator programming via combining 'type morpheme's.

# What
As you know from natural languages, we have three different morphemes here:
## Prefix
- **To be satisfied:**
```
!pred
```
This prefix goes in front of a predicate, and keeps going only if the predicate returns true.

- **To negate:**
```
~pred
```
Similar to the previous one, this negates the return value of a predicate.

- **To refine a type:**
```
type^pred
type^prop
```
In the former case, `type` is refined at term level, while in the latter case, `type` is refined at propositional level.

## Infix
- **Has the type:**
```
symbol:type
```
`symbol` has the type `type` in the current environment.

- **Composition:**
```
f|g
```
This composites `f` and `g`, and gives `f \circ g`.

## Suffix
- **To be applied to:**
```
f@arg
```
This applies the function / type constructor `f` to the argument.

# Some Example
- **Singleton**
```clojure
(def singleton
    (-> Bool Type)
    (lambda (x)
        (if x Nat (List Nat))))

(def mkSingle
    (-> x:Bool singleton@x)
    (lambda (x)
        (if x 0 nil)))
```

- **Refined**
```clojure
(def nthVect
    (-> n:Nat vct:Vect@(m elem)^(< n m))
    ...)
```