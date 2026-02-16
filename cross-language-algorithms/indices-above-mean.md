---
created: 2026-02-06
---

# Finding Indices of Elements Above the Mean: A Cross-Language Study

_February 6, 2026_

> Part of a series: [overview](three-algorithms-overview.md) | **this document** | [expression trees](expression-tree-evaluation.md) | [concurrent pipelines](concurrent-producer-consumer.md)

This document explores a single algorithm implemented across dozens of programming languages, organized by language family. By examining how different languages and paradigms approach the same problem, we can understand the philosophies, trade-offs, and aesthetics that distinguish programming communities from one another.

## Table of Contents

1. [The Algorithm](#the-algorithm)
2. [The APL Family](#the-apl-family)
   - [APL](#apl)
   - [J](#j)
   - [K](#k)
   - [BQN](#bqn)
   - [Uiua](#uiua)
   - [APL Family Comparison](#apl-family-comparison)
3. [The ML Family](#the-ml-family)
   - [Haskell](#haskell)
   - [Standard ML](#standard-ml)
   - [OCaml](#ocaml)
   - [F#](#f)
   - [Elm](#elm)
   - [PureScript](#purescript)
   - [ML Family Comparison](#ml-family-comparison)
4. [The Lisp Family](#the-lisp-family)
   - [Common Lisp](#common-lisp)
   - [Scheme](#scheme)
   - [Racket](#racket)
   - [Clojure](#clojure)
   - [Lisp Family Comparison](#lisp-family-comparison)
5. [The BEAM Family](#the-beam-family)
   - [Erlang](#erlang)
   - [Elixir](#elixir)
   - [Gleam](#gleam)
   - [BEAM Family Comparison](#beam-family-comparison)
6. [Newer Functional Languages](#newer-functional-languages)
   - [Roc](#roc)
   - [Unison](#unison)
7. [Proof Assistants and Dependently Typed Languages](#proof-assistants-and-dependently-typed-languages)
   - [Lean](#lean)
   - [Agda](#agda)
   - [Idris](#idris)
   - [Coq (Rocq)](#coq-rocq)
   - [Proof Assistant Comparison](#proof-assistant-comparison)
8. [The JVM Family](#the-jvm-family)
   - [Java](#java)
   - [Scala](#scala)
   - [Kotlin](#kotlin)
   - [JVM Family Comparison](#jvm-family-comparison)
9. [The C Family](#the-c-family)
   - [C](#c)
   - [C++](#c-1)
   - [Objective-C](#objective-c)
   - [C Family Comparison](#c-family-comparison)
10. [Modern Systems Languages](#modern-systems-languages)
    - [Rust](#rust)
    - [Go](#go)
    - [Zig](#zig)
    - [Systems Language Comparison](#systems-language-comparison)
11. [The Wirth Family](#the-wirth-family)
    - [Pascal](#pascal)
    - [Modula-2](#modula-2)
    - [Ada](#ada)
    - [Wirth Family Comparison](#wirth-family-comparison)
12. [Scripting Languages](#scripting-languages)
    - [Python](#python)
    - [Ruby](#ruby)
    - [JavaScript](#javascript)
    - [TypeScript](#typescript)
    - [Perl](#perl)
    - [Lua](#lua)
    - [Scripting Language Comparison](#scripting-language-comparison)
13. [The Apple Ecosystem](#the-apple-ecosystem)
    - [Swift](#swift)
14. [Historical Languages](#historical-languages)
    - [FORTRAN](#fortran)
    - [COBOL](#cobol)
    - [BASIC](#basic)
    - [Algol 60](#algol-60)
    - [Historical Language Comparison](#historical-language-comparison)
15. [Other Notable Languages](#other-notable-languages)
    - [Forth](#forth)
    - [Prolog](#prolog)
    - [Smalltalk](#smalltalk)
    - [SQL](#sql)
    - [MATLAB](#matlab)
    - [R](#r)
    - [Julia](#julia)
16. [Cross-Family Comparison](#cross-family-comparison)
17. [Conclusion](#conclusion)

---

## The Algorithm

The task we will implement across all languages is this: given an array of numbers, return the indices of all elements whose value is greater than the arithmetic mean of the array.

More formally, given an array A of n numbers, we wish to compute the set of indices I such that for each i in I, A[i] > (Σ A[j]) / n.

For example, given the array `[3, 1, 4, 1, 5, 9, 2, 6]`, the sum is 31 and the mean is 3.875. The elements greater than 3.875 are 4, 5, 9, and 6, which appear at indices 2, 4, 5, and 7 (using zero-based indexing) or 3, 5, 6, and 8 (using one-based indexing).

The algorithm naturally decomposes into these steps:

1. Compute the sum of all elements.
2. Compute the count of elements.
3. Divide the sum by the count to obtain the mean.
4. Compare each element to the mean.
5. Collect the indices where the comparison yields true.

Different languages make these steps more or less explicit. In the APL family, steps 4 and 5 are essentially fused into implicit array operations. In languages with explicit loops, each step requires its own code. This variation in explicitness is one of the most interesting axes of comparison across language families.

Throughout this document, we will use zero-based indexing unless the language conventionally uses one-based indexing (as with APL, FORTRAN, Lua, MATLAB, R, and Julia).

---

## The APL Family

The APL family represents programming's most radical experiment in notation. These languages treat arrays as the fundamental unit of computation, making iteration implicit rather than explicit. A transformation that requires a loop in most languages is expressed as a single operation applied to an entire array.

### APL

APL (A Programming Language) was designed by Kenneth Iverson in the 1960s as a mathematical notation for describing algorithms. It uses special Unicode glyphs that require a custom keyboard layout.

```apl
{⍸⍵>(+/⍵)÷≢⍵}
```

Reading right to left within the expression: `≢⍵` computes the tally (count) of the argument ⍵. The expression `+/⍵` reduces (folds) the argument with addition, yielding the sum. Dividing the sum by the count with `÷` produces the mean. The comparison `⍵>` tests each element of ⍵ against this scalar mean, producing a boolean array of 0s and 1s. Finally, `⍸` (called "where" or "indices of") returns the indices where the boolean array contains 1s.

The curly braces define an anonymous function (called a "dfn" or direct function), and ⍵ represents the right argument. APL uses one-based indexing by default.

### J

J was created by Kenneth Iverson and Roger Hui in 1990 as an ASCII-friendly successor to APL. It replaces APL's special glyphs with digraphs formed from ASCII characters.

```j
I. @ (] > +/ % #)
```

Here `#` is tally, `+/` is sum, and `%` is division. The expression `+/ % #` is a *fork*: when applied to an array y, J evaluates `(+/ y) % (# y)`, computing the mean.

The outer expression `] > +/ % #` is another fork where `]` is the identity function. This computes `y > mean(y)`, producing a boolean array.

The `I.` primitive returns indices of 1s, and `@` composes it with the comparison fork. The result is a fully tacit (point-free) function. J uses zero-based indexing.

### K

K was created by Arthur Whitney and is used extensively in financial computing. It pushes terseness to an extreme.

```k
{&x>(+/x)%#x}
```

Here `#x` is count, `+/x` is sum, and `%` is divide. The comparison `x>` produces a boolean list, and `&` returns indices of 1s. K programmers often extract the mean computation: `avg:{(+/x)%#x}` allows writing `{&x>avg x}`. K uses zero-based indexing.

### BQN

BQN is a modern redesign of APL by Marshall Lochbaum, aiming to fix historical inconsistencies while preserving array programming's essence.

```bqn
{/𝕩>(+´𝕩)÷≠𝕩}
```

Here `≠` is length, `+´` is sum (the `´` modifier is fold/reduce), and `÷` is divide. The comparison `𝕩>` tests each element, and `/` returns indices of 1s. The symbol `𝕩` represents the right argument. BQN uses zero-based indexing and was designed with a more systematic glyph assignment than historical APL.

### Uiua

Uiua (pronounced "wee-wuh") is an experimental stack-based array language that eliminates named variables entirely.

```uiua
⊚>⊃(÷⊃⧻/+)∘
```

Operations consume values from a stack and push results back. The glyph `⧻` is length, `/+` is reduce-sum, `÷` divides to get the mean. The `⊃` glyph (fork) applies multiple functions to the same input. The `>` compares, and `⊚` extracts indices. Uiua uses zero-based indexing.

### APL Family Comparison

The APL family shares a commitment to implicit array operations and extreme brevity. Within the family, the main variations are notational (ASCII vs. Unicode glyphs) and philosophical (how tacit programming is encouraged).

APL and BQN use Unicode symbols that visually suggest their meanings, requiring special keyboard input but offering a kind of pictorial clarity. J and K use ASCII, making them more accessible in plain-text environments but requiring memorization of less intuitive symbol combinations.

K represents the terseness extreme, optimizing for minimal keystrokes. BQN represents the pedagogical extreme, trying to be learnable while preserving APL's power. Uiua experiments with eliminating variables entirely, pushing tacit programming beyond even J.

All these languages share the fundamental insight that loops are often incidental complexity: if you want to add two arrays element-wise, you should be able to write `A + B` rather than iterating over indices. This philosophy makes them remarkably powerful for array-heavy computation but creates a steep learning curve for programmers accustomed to explicit iteration.

---

## The ML Family

The ML (Meta Language) family descends from Robin Milner's work on theorem provers in the 1970s. These languages are characterized by strong static typing with type inference, algebraic data types, and pattern matching. They maintain traditional function application syntax where functions precede their arguments.

### Haskell

Haskell is a purely functional language with lazy evaluation, named after logician Haskell Curry.

```haskell
import Data.List (findIndices)

-- Using the standard library function findIndices
aboveMean :: [Double] -> [Int]
aboveMean xs = findIndices (> mean) xs
  where
    mean = sum xs / fromIntegral (length xs)

-- Alternative using a list comprehension
aboveMean' :: [Double] -> [Int]
aboveMean' xs = [i | (i, x) <- zip [0..] xs, x > mean]
  where
    mean = sum xs / fromIntegral (length xs)

-- Point-free style using the (&) operator
aboveMean'' :: [Double] -> [Int]
aboveMean'' = findIndices =<< (>) . ((/) <$> sum <*> (fromIntegral . length))
```

The first version uses `findIndices` from Data.List, which takes a predicate and returns indices where the predicate holds. The `where` clause defines `mean` locally. The `fromIntegral` conversion is necessary because `length` returns an `Int` but we need a `Double` for division.

The list comprehension version explicitly zips each element with its index, then filters. The point-free version demonstrates that Haskell can achieve APL-like terseness, though idiomatically Haskell programmers favor the readable first version.

### Standard ML

Standard ML is one of the original ML dialects, known for its formal definition and module system.

```sml
fun aboveMean xs =
    let
        val n = Real.fromInt (length xs)
        val mean = foldl op+ 0.0 xs / n
        fun check (i, x, acc) = if x > mean then i :: acc else acc
    in
        rev (foldli check [] xs)
    end
```

Standard ML requires explicit type conversions with `Real.fromInt`. The `foldli` function provides the index along with each element. We accumulate indices in reverse order (for efficiency with cons) and reverse at the end.

### OCaml

OCaml extends ML with object-oriented features (rarely used in practice) and is known for its practical focus and good performance.

```ocaml
let above_mean xs =
  let n = float_of_int (List.length xs) in
  let mean = List.fold_left (+.) 0.0 xs /. n in
  xs
  |> List.mapi (fun i x -> (i, x))
  |> List.filter_map (fun (i, x) -> if x > mean then Some i else None)
```

OCaml uses `+.` and `/.` for floating-point operations, distinct from integer `+` and `/`. The pipeline operator `|>` allows data to flow left-to-right through transformations. `List.filter_map` combines filtering and mapping in one pass.

### F#

F# is a functional-first language for the .NET platform, heavily inspired by OCaml.

```fsharp
let aboveMean xs =
    let mean = List.average xs
    xs
    |> List.indexed
    |> List.choose (fun (i, x) -> if x > mean then Some i else None)
```

F# provides `List.average` in the standard library, eliminating manual mean computation. `List.indexed` pairs each element with its index, and `List.choose` is equivalent to OCaml's `filter_map`. The syntax is cleaner than OCaml's, reflecting F#'s more recent design.

### Elm

Elm is a purely functional language for web front-ends, designed for simplicity and helpful error messages.

```elm
aboveMean : List Float -> List Int
aboveMean xs =
    let
        mean =
            List.sum xs / toFloat (List.length xs)
    in
    xs
        |> List.indexedMap Tuple.pair
        |> List.filterMap
            (\( i, x ) ->
                if x > mean then
                    Just i
                else
                    Nothing
            )
```

Elm deliberately limits its feature set for simplicity. There is no `findIndices` in the standard library, so we build indexed pairs with `List.indexedMap` and use `List.filterMap` to extract indices. The verbose lambda syntax reflects Elm's preference for explicitness over terseness.

### PureScript

PureScript is a Haskell-like language that compiles to JavaScript, often used for front-end development.

```purescript
import Data.Array (findIndices, length)
import Data.Int (toNumber)
import Data.Foldable (sum)

aboveMean :: Array Number -> Array Int
aboveMean xs = findIndices (\x -> x > mean) xs
  where
    mean = sum xs / toNumber (length xs)
```

PureScript's syntax closely mirrors Haskell's. The main differences are practical: it uses strict evaluation (unlike Haskell's laziness), compiles to JavaScript, and has a different standard library (using `Array` instead of linked lists by default).

### ML Family Comparison

The ML family shares a commitment to strong static typing with inference, algebraic data types, and immutability by default. Within the family, the main variations are in strictness (Haskell is lazy, others are strict), purity (Haskell enforces purity, OCaml allows side effects), and standard library design.

Haskell offers the most powerful abstraction mechanisms (type classes, monads, higher-kinded types) but has a steeper learning curve. OCaml and F# are more pragmatic, offering escape hatches to imperative code when needed. Elm deliberately restricts features to maintain simplicity and guarantee no runtime errors.

All these languages make the algorithm's structure explicit: we name the intermediate values (mean), explicitly map over the list to pair elements with indices, and explicitly filter. This verbosity compared to APL is the price paid for readability and type safety. The compensating benefit is that the code is self-documenting and errors are caught at compile time.

---

## The Lisp Family

The Lisp family, originating with John McCarthy's work in 1958, is characterized by homoiconicity (code is data), prefix notation with S-expressions, and a tradition of powerful macro systems. These languages use parentheses to delimit all expressions uniformly.

### Common Lisp

Common Lisp is the standardized descendant of many early Lisp dialects, known for its extensive standard library and multiple programming paradigms.

```common-lisp
(defun above-mean (xs)
  (let* ((mean (/ (reduce #'+ xs) (length xs))))
    (loop for x in xs
          for i from 0
          when (> x mean)
          collect i)))
```

Common Lisp's `loop` macro provides an embedded iteration language. The `let*` form binds variables sequentially (needed here because we only have one binding, but `let*` is idiomatic when order matters). The `reduce` function folds with `+`, and `length` returns the count.

Alternatively, using more traditional recursion:

```common-lisp
(defun above-mean (xs)
  (let ((mean (/ (reduce #'+ xs) (length xs))))
    (labels ((helper (xs i)
               (cond ((null xs) nil)
                     ((> (car xs) mean) (cons i (helper (cdr xs) (1+ i))))
                     (t (helper (cdr xs) (1+ i))))))
      (helper xs 0))))
```

### Scheme

Scheme is a minimalist Lisp dialect known for its clean semantics and use in teaching.

```scheme
(define (above-mean xs)
  (let ((mean (/ (apply + xs) (length xs))))
    (filter-map (lambda (i x) (and (> x mean) i))
                (iota (length xs))
                xs)))

;; If filter-map is unavailable (not in R5RS):
(define (above-mean xs)
  (let ((mean (/ (apply + xs) (length xs))))
    (let loop ((xs xs) (i 0) (acc '()))
      (cond ((null? xs) (reverse acc))
            ((> (car xs) mean) (loop (cdr xs) (+ i 1) (cons i acc)))
            (else (loop (cdr xs) (+ i 1) acc))))))
```

Scheme's minimalism means different implementations provide different utilities. The named `let` form (`let loop`) creates a local recursive function, a common Scheme idiom for iteration.

### Racket

Racket evolved from Scheme into a language-oriented programming platform with extensive libraries.

```racket
#lang racket

(define (above-mean xs)
  (define mean (/ (apply + xs) (length xs)))
  (for/list ([x (in-list xs)]
             [i (in-naturals)]
             #:when (> x mean))
    i))
```

Racket's `for/list` comprehension combines iteration and collection cleanly. The `in-naturals` sequence provides indices starting from 0, and `#:when` filters during iteration.

### Clojure

Clojure is a modern Lisp on the JVM, emphasizing immutability and concurrency.

```clojure
(defn above-mean [xs]
  (let [mean (/ (reduce + xs) (count xs))]
    (keep-indexed (fn [i x] (when (> x mean) i)) xs)))
```

Clojure's `keep-indexed` combines indexing, filtering, and mapping: it calls the function with each index and element, keeping non-nil results. The `when` form returns `nil` when the condition is false, which `keep-indexed` discards. This is remarkably concise for a Lisp.

Alternatively, using a for comprehension:

```clojure
(defn above-mean [xs]
  (let [mean (/ (reduce + xs) (count xs))]
    (for [[i x] (map-indexed vector xs)
          :when (> x mean)]
      i)))
```

### Lisp Family Comparison

The Lisp family shares the fundamental syntax of S-expressions, where every compound form is a parenthesized list with the operator first. Within the family, the main variations are in minimalism versus batteries-included, and in functional purity versus pragmatic mutability.

Scheme represents the minimalist extreme, specifying only core primitives and leaving libraries to implementations. Common Lisp represents the maximalist extreme, with a huge standard library including the powerful `loop` macro. Racket extends Scheme with extensive libraries and a focus on language-oriented programming. Clojure brings Lisp to the JVM with a strong emphasis on immutability and modern data structures.

All these languages share Lisp's characteristic flexibility: code is data, macros can transform syntax, and programs can be built interactively at a REPL. The prefix notation requires some adjustment but becomes natural with practice, and the uniformity of syntax makes macros and metaprogramming more tractable than in languages with complex grammars.

---

## The BEAM Family

The BEAM (Bogdan's Erlang Abstract Machine) family runs on Erlang's virtual machine, designed for distributed, fault-tolerant systems. These languages share Erlang's actor model and supervision trees but vary in syntax and type systems.

### Erlang

Erlang was developed at Ericsson for telecom systems, pioneering the actor model for concurrency and "let it crash" fault tolerance.

```erlang
-module(stats).
-export([above_mean/1]).

above_mean(Xs) ->
    Mean = lists:sum(Xs) / length(Xs),
    {Indices, _} = lists:foldl(
        fun(X, {Acc, I}) ->
            case X > Mean of
                true -> {[I | Acc], I + 1};
                false -> {Acc, I + 1}
            end
        end,
        {[], 0},
        Xs
    ),
    lists:reverse(Indices).
```

Erlang uses pattern matching and explicit recursion or folds. Variables are immutable and single-assignment. The tuple `{Acc, I}` threads both the accumulator and current index through the fold. Module and export declarations are required.

### Elixir

Elixir is a modern language on the BEAM with Ruby-influenced syntax and powerful metaprogramming.

```elixir
defmodule Stats do
  def above_mean(xs) do
    mean = Enum.sum(xs) / length(xs)
    
    xs
    |> Enum.with_index()
    |> Enum.filter(fn {x, _i} -> x > mean end)
    |> Enum.map(fn {_x, i} -> i end)
  end
end
```

Elixir's pipe operator `|>` makes the transformation chain readable. `Enum.with_index` pairs each element with its index, then we filter and extract indices. The syntax is more approachable than Erlang's while retaining the same runtime semantics.

A more concise version using a comprehension:

```elixir
def above_mean(xs) do
  mean = Enum.sum(xs) / length(xs)
  for {x, i} <- Enum.with_index(xs), x > mean, do: i
end
```

### Gleam

Gleam is a statically typed language for the BEAM, offering type safety without sacrificing Erlang interoperability.

```gleam
import gleam/list
import gleam/float
import gleam/int

pub fn above_mean(xs: List(Float)) -> List(Int) {
  let sum = list.fold(xs, 0.0, float.add)
  let count = int.to_float(list.length(xs))
  let mean = sum /. count

  xs
  |> list.index_map(fn(x, i) { #(i, x) })
  |> list.filter_map(fn(pair) {
    let #(i, x) = pair
    case x >. mean {
      True -> Ok(i)
      False -> Error(Nil)
    }
  })
}
```

Gleam adds static types to the BEAM ecosystem. The `/.` operator is for float division, and `>.` for float comparison (distinct from integer operations). The `filter_map` function keeps `Ok` values and discards `Error` values.

### BEAM Family Comparison

The BEAM family shares Erlang's runtime, meaning all these languages have access to OTP (Open Telecom Platform) patterns for fault tolerance, the actor model for concurrency, and hot code reloading. The main variations are in syntax and type systems.

Erlang has Prolog-influenced syntax with atoms, pattern matching, and explicit recursion. Elixir offers Ruby-influenced syntax with macros and protocols, attracting developers who find Erlang's syntax off-putting. Gleam adds static typing, catching errors at compile time that Erlang and Elixir would only find at runtime.

The algorithm implementations differ mainly in stylistic preferences. Elixir's pipe operator makes chains of transformations readable. Gleam's type system requires explicit float operations. All three express the same underlying computation on the same runtime.

---

## Newer Functional Languages

Several recent languages are exploring new points in the design space, often combining ideas from multiple traditions.

### Roc

Roc is a new functional language focused on speed, simplicity, and a pleasant development experience. It draws from Elm's user-friendliness and aims for performance competitive with systems languages.

```roc
aboveMean : List F64 -> List U64
aboveMean = \xs ->
    mean = List.sum xs / Num.toF64 (List.len xs)
    List.keepIf (List.mapWithIndex xs \x, i -> { x, i }) \{ x, i } -> x > mean
    |> List.map \{ x: _, i } -> i
```

Roc uses backslash for lambda expressions and has a clean record syntax. The pipeline shows the data flow clearly. Types use abbreviations like `F64` for 64-bit floats and `U64` for unsigned 64-bit integers.

### Unison

Unison is a content-addressed language where code is stored by hash rather than as text files. This enables unique features like seamless distributed computing.

```unison
aboveMean : [Float] -> [Nat]
aboveMean xs =
  mean = List.foldLeft (+) 0.0 xs / Float.fromNat (List.size xs)
  xs
    |> List.indexed
    |> List.filterMap cases
        (i, x) | x > mean -> Some i
        _ -> None
```

Unison's syntax resembles Haskell but with some unique features. The `cases` keyword introduces pattern matching with guards inline. Content-addressing means that the function's identity is its hash, enabling distributed execution without deployment concerns.

---

## Proof Assistants and Dependently Typed Languages

Proof assistants blur the line between programming and mathematics. They use dependent types to express properties about code that can be verified by the type checker.

### Lean

Lean 4 is both a proof assistant and a practical programming language, developed primarily at Microsoft Research.

```lean
def aboveMean (xs : Array Float) : Array Nat :=
  let sum := xs.foldl (· + ·) 0.0
  let mean := sum / xs.size.toFloat
  xs.zipWithIndex.filterMap fun (x, i) =>
    if x > mean then some i else none
```

Lean 4 has a clean syntax that feels more like a conventional programming language than earlier proof assistants. The `·` notation creates anonymous functions with placeholders. While Lean can express and verify mathematical properties, this implementation focuses on the computational aspect.

### Agda

Agda is a dependently typed language often used for research in type theory and formal verification.

```agda
open import Data.List using (List; []; _∷_; length; sum; zip; filterᵇ; map)
open import Data.Nat using (ℕ)
open import Data.Float using (Float; _/_; _>_)
open import Data.Product using (_,_)
open import Data.Bool using (if_then_else_)

aboveMean : List Float → List ℕ
aboveMean xs = 
  let mean = sum xs / fromℕ (length xs)
      indexed = zip (enumFromTo 0 (length xs - 1)) xs
      filtered = filterᵇ (λ { (i , x) → x > mean }) indexed
  in map (λ { (i , _) → i }) filtered
```

Agda uses Unicode symbols extensively and has powerful dependent types. The syntax `λ { (i , x) → ... }` is pattern-matching lambda. Agda can verify properties about programs, though this example doesn't exploit that capability.

### Idris

Idris is designed to bring dependent types to practical programming, aiming to be useful for real software development.

```idris
aboveMean : List Double -> List Nat
aboveMean xs = 
  let mean = sum xs / cast (length xs)
      indexed = zip [0 .. length xs] xs
  in map fst $ filter (\(_, x) => x > mean) indexed
```

Idris syntax resembles Haskell but with strict evaluation and dependent types. The `cast` function handles numeric conversions, and `$` is low-precedence function application as in Haskell. Idris 2, the current version, compiles to Scheme.

### Coq (Rocq)

Coq (recently renamed Rocq) is one of the oldest and most established proof assistants, used extensively in verified software like CompCert and mathematical proofs like the four-color theorem.

```coq
Require Import List Floats.
Import ListNotations.

Fixpoint indexed {A : Type} (xs : list A) (start : nat) : list (nat * A) :=
  match xs with
  | [] => []
  | x :: rest => (start, x) :: indexed rest (S start)
  end.

Definition aboveMean (xs : list float) : list nat :=
  let sum := fold_left Fadd 0%float xs in
  let mean := Fdiv sum (of_nat (length xs)) in
  map fst (filter (fun '(_, x) => Flt_bool mean x) (indexed xs 0)).
```

Coq's syntax is more verbose, reflecting its origin as a tool for formal proofs rather than programming. The `'(_, x)` notation is pattern matching in a lambda. Coq's type system can express and verify mathematical properties about programs.

### Proof Assistant Comparison

Proof assistants share the ability to express precise specifications in types and verify them mechanically. Within this family, the main variations are in usability, proof automation, and practical programming support.

Lean 4 has invested heavily in being usable as a practical programming language, with syntax that feels familiar and good tooling. Agda is research-oriented, exploring the frontiers of type theory. Idris explicitly targets practical programming with dependent types. Coq has the longest track record and largest library of verified software and proofs.

For our algorithm, the implementations look similar to ML-family code because we're not using dependent types to verify properties. The power of these languages would show if we wanted to prove, say, that the returned indices are all valid indices into the original array and that the corresponding elements all exceed the mean.

---

## The JVM Family

Languages on the Java Virtual Machine share access to Java's vast ecosystem of libraries while offering varying approaches to syntax and programming paradigms.

### Java

Java is the original JVM language, known for its verbosity but also its stability, tooling, and widespread adoption.

```java
import java.util.*;
import java.util.stream.*;

public class Stats {
    public static List<Integer> aboveMean(double[] xs) {
        double mean = Arrays.stream(xs).average().orElse(0.0);
        return IntStream.range(0, xs.length)
            .filter(i -> xs[i] > mean)
            .boxed()
            .collect(Collectors.toList());
    }
}
```

Modern Java (8+) has functional features via streams. The `IntStream.range` generates indices, `filter` keeps those where the element exceeds the mean, and `collect` gathers results into a list. Pre-streams Java would require explicit loops.

### Scala

Scala combines object-oriented and functional programming on the JVM, with a sophisticated type system.

```scala
def aboveMean(xs: Seq[Double]): Seq[Int] = {
  val mean = xs.sum / xs.length
  xs.zipWithIndex.collect { case (x, i) if x > mean => i }
}
```

Scala's `collect` combines filtering and mapping with pattern matching. The `case` guard `if x > mean` filters, and the right side `=> i` extracts the index. This is remarkably concise compared to Java.

Scala 3 syntax allows even cleaner expressions:

```scala
def aboveMean(xs: Seq[Double]): Seq[Int] =
  val mean = xs.sum / xs.length
  for (x, i) <- xs.zipWithIndex if x > mean yield i
```

### Kotlin

Kotlin is designed to be a better Java, with null safety, concise syntax, and full Java interoperability.

```kotlin
fun aboveMean(xs: DoubleArray): List<Int> {
    val mean = xs.average()
    return xs.indices.filter { xs[it] > mean }
}
```

Kotlin's `indices` property returns the valid index range, and `filter` keeps matching indices. The `it` keyword refers to the implicit lambda parameter. This is much more concise than Java while remaining readable.

### JVM Family Comparison

The JVM family shares the Java ecosystem, garbage collection, and JIT compilation. The main variations are in type systems, syntax, and programming paradigms.

Java prioritizes backward compatibility and explicitness, resulting in verbose but predictable code. Scala offers powerful abstractions and type system features, appealing to those who want ML-family sophistication on the JVM. Kotlin targets pragmatic improvement over Java, popular for Android development and server-side work.

Our algorithm shows the conciseness spectrum clearly: Java requires the most ceremony (stream setup, collectors, boxing), Kotlin is intermediate (built-in `indices`, simple `filter`), and Scala is most concise (pattern matching with guards).

---

## The C Family

The C family encompasses languages that descended from or were heavily influenced by C's syntax and systems-programming orientation.

### C

C is the lingua franca of systems programming, created by Dennis Ritchie at Bell Labs in the early 1970s.

```c
#include <stdio.h>
#include <stdlib.h>

// Returns a dynamically allocated array of indices
// Writes the count to *count_out
int* above_mean(double* xs, int n, int* count_out) {
    // Compute mean
    double sum = 0.0;
    for (int i = 0; i < n; i++) {
        sum += xs[i];
    }
    double mean = sum / n;
    
    // First pass: count how many elements exceed the mean
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (xs[i] > mean) count++;
    }
    
    // Allocate result array
    int* result = malloc(count * sizeof(int));
    if (result == NULL) return NULL;
    
    // Second pass: collect indices
    int j = 0;
    for (int i = 0; i < n; i++) {
        if (xs[i] > mean) {
            result[j++] = i;
        }
    }
    
    *count_out = count;
    return result;
}
```

C requires manual memory management and explicit loops. The function signature shows C's constraints: we must pass the array length separately, return the result array via pointer, and communicate the result count via an output parameter. Two passes are needed because we must know the count before allocating. The caller is responsible for freeing the returned memory.

### C++

C++ extends C with classes, templates, and (in modern versions) functional features.

```cpp
#include <vector>
#include <numeric>
#include <algorithm>

std::vector<size_t> aboveMean(const std::vector<double>& xs) {
    double mean = std::accumulate(xs.begin(), xs.end(), 0.0) / xs.size();
    
    std::vector<size_t> result;
    for (size_t i = 0; i < xs.size(); ++i) {
        if (xs[i] > mean) {
            result.push_back(i);
        }
    }
    return result;
}
```

Modern C++ offers `std::vector` for dynamic arrays with automatic memory management. `std::accumulate` handles the sum. We still need an explicit loop for the index collection, though C++20 ranges make more functional approaches possible:

```cpp
// C++20 with ranges
#include <ranges>
#include <vector>
#include <numeric>

std::vector<size_t> aboveMean(const std::vector<double>& xs) {
    double mean = std::accumulate(xs.begin(), xs.end(), 0.0) / xs.size();
    
    auto indices = std::views::iota(0uz, xs.size())
                 | std::views::filter([&](size_t i) { return xs[i] > mean; });
    
    return {indices.begin(), indices.end()};
}
```

### Objective-C

Objective-C adds Smalltalk-style messaging to C, historically used for macOS and iOS development before Swift.

```objectivec
#import <Foundation/Foundation.h>

NSArray<NSNumber *>* aboveMean(NSArray<NSNumber *>* xs) {
    // Compute mean
    double sum = 0.0;
    for (NSNumber* x in xs) {
        sum += [x doubleValue];
    }
    double mean = sum / [xs count];
    
    // Collect indices above mean
    NSMutableArray<NSNumber *>* result = [NSMutableArray array];
    [xs enumerateObjectsUsingBlock:^(NSNumber* x, NSUInteger i, BOOL* stop) {
        if ([x doubleValue] > mean) {
            [result addObject:@(i)];
        }
    }];
    
    return result;
}
```

Objective-C uses Foundation's `NSArray` for dynamic arrays. The block-based enumeration provides indices. The square bracket syntax `[object message]` is Objective-C's distinctive Smalltalk heritage. Numbers must be boxed as `NSNumber` objects in collections.

### C Family Comparison

The C family shares C's low-level access to memory and its basic syntax of braces, semicolons, and operator precedence. The main variations are in abstraction level and safety.

C provides minimal abstraction over the machine, requiring manual memory management and explicit size tracking. C++ adds classes, templates, and RAII for automatic resource management, but retains C's potential for memory errors. Objective-C adds object-orientation with reference counting (now automatic with ARC).

Our algorithm shows how C's simplicity becomes complexity when we need dynamic data: we must allocate, track sizes, and communicate via output parameters. C++ and Objective-C's collections eliminate much of this ceremony, though explicit iteration remains.

---

## Modern Systems Languages

These languages aim to provide systems-level performance with modern safety features and ergonomics.

### Rust

Rust provides memory safety without garbage collection through its ownership system.

```rust
fn above_mean(xs: &[f64]) -> Vec<usize> {
    let mean: f64 = xs.iter().sum::<f64>() / xs.len() as f64;
    xs.iter()
        .enumerate()
        .filter(|(_, &x)| x > mean)
        .map(|(i, _)| i)
        .collect()
}
```

Rust's iterator adaptors (`enumerate`, `filter`, `map`) compose into an efficient pipeline. The type annotation on `sum::<f64>()` helps inference. The ownership system guarantees memory safety at compile time without runtime overhead.

### Go

Go emphasizes simplicity and fast compilation, with built-in concurrency support.

```go
package stats

func AboveMean(xs []float64) []int {
    // Compute mean
    sum := 0.0
    for _, x := range xs {
        sum += x
    }
    mean := sum / float64(len(xs))
    
    // Collect indices above mean
    var result []int
    for i, x := range xs {
        if x > mean {
            result = append(result, i)
        }
    }
    return result
}
```

Go deliberately omits many features (generics were only recently added, no map/filter/reduce in the standard library). The idiom is explicit loops with `range`. This verbosity is intentional: Go prioritizes readability and simplicity over expressiveness.

### Zig

Zig is a newer systems language aiming to be a better C, with compile-time execution and explicit error handling.

```zig
const std = @import("std");

fn aboveMean(allocator: std.mem.Allocator, xs: []const f64) ![]usize {
    // Compute mean
    var sum: f64 = 0.0;
    for (xs) |x| {
        sum += x;
    }
    const mean = sum / @as(f64, @floatFromInt(xs.len));
    
    // Count matches
    var count: usize = 0;
    for (xs) |x| {
        if (x > mean) count += 1;
    }
    
    // Allocate and fill result
    var result = try allocator.alloc(usize, count);
    var j: usize = 0;
    for (xs, 0..) |x, i| {
        if (x > mean) {
            result[j] = i;
            j += 1;
        }
    }
    
    return result;
}
```

Zig requires explicit allocators, making memory allocation visible and testable. The `!` in the return type indicates the function can return an error (for allocation failure). The `try` keyword propagates errors. This explicitness is similar to C but with better ergonomics and safety.

### Systems Language Comparison

Modern systems languages share the goal of safe, efficient code without garbage collection. Within this group, they make different trade-offs.

Rust has the most sophisticated safety system (ownership and borrowing) but the steepest learning curve. Go prioritizes simplicity and fast compilation, accepting more verbosity. Zig aims to replace C with minimal abstraction over the machine, explicit allocation, and compile-time execution.

Our algorithm shows these philosophies: Rust's iterators provide high-level composition with zero-cost abstraction. Go's explicit loops are simple but verbose. Zig's explicit allocation and two-pass approach is closest to C but with better error handling.

---

## The Wirth Family

Niklaus Wirth designed a series of languages emphasizing clarity, structure, and type safety, influencing language design for decades.

### Pascal

Pascal was designed for teaching structured programming and remains influential.

```pascal
program AboveMeanDemo;

type
  IntArray = array of Integer;
  FloatArray = array of Real;

function AboveMean(xs: FloatArray): IntArray;
var
  sum, mean: Real;
  count, i, j: Integer;
begin
  // Compute mean
  sum := 0.0;
  for i := 0 to High(xs) do
    sum := sum + xs[i];
  mean := sum / Length(xs);
  
  // Count matches
  count := 0;
  for i := 0 to High(xs) do
    if xs[i] > mean then
      Inc(count);
  
  // Allocate and fill result
  SetLength(Result, count);
  j := 0;
  for i := 0 to High(xs) do
    if xs[i] > mean then begin
      Result[j] := i;
      Inc(j);
    end;
end;
```

Pascal uses `begin`/`end` blocks, `:=` for assignment, and `High()` for the last valid index. Modern Pascal (Free Pascal, Delphi) supports dynamic arrays with `SetLength`. The structure is similar to C but more verbose and more strongly typed.

### Modula-2

Modula-2 is Wirth's successor to Pascal, adding modules and concurrency.

```modula2
MODULE Stats;

FROM Storage IMPORT ALLOCATE;
FROM SYSTEM IMPORT TSIZE;

TYPE
  RealArray = ARRAY OF REAL;
  IntArray = ARRAY OF INTEGER;

PROCEDURE AboveMean(xs: RealArray; VAR result: IntArray; VAR count: CARDINAL);
VAR
  sum, mean: REAL;
  i, j: CARDINAL;
BEGIN
  (* Compute mean *)
  sum := 0.0;
  FOR i := 0 TO HIGH(xs) DO
    sum := sum + xs[i];
  END;
  mean := sum / FLOAT(HIGH(xs) + 1);
  
  (* Count matches *)
  count := 0;
  FOR i := 0 TO HIGH(xs) DO
    IF xs[i] > mean THEN INC(count) END;
  END;
  
  (* Fill result - assuming result is pre-allocated *)
  j := 0;
  FOR i := 0 TO HIGH(xs) DO
    IF xs[i] > mean THEN
      result[j] := i;
      INC(j);
    END;
  END;
END AboveMean;

END Stats.
```

Modula-2 adds explicit modules, separating interface from implementation. The `VAR` keyword indicates pass-by-reference output parameters. The syntax is cleaner than Pascal's with explicit `END` for all constructs.

### Ada

Ada is a strongly typed language designed for safety-critical systems, influenced by Pascal.

```ada
with Ada.Containers.Vectors;

package Stats is
   package Int_Vectors is new Ada.Containers.Vectors
     (Index_Type => Natural, Element_Type => Natural);
   
   function Above_Mean (Xs : array of Float) return Int_Vectors.Vector;
end Stats;

package body Stats is
   function Above_Mean (Xs : array of Float) return Int_Vectors.Vector is
      Sum  : Float := 0.0;
      Mean : Float;
      Result : Int_Vectors.Vector;
   begin
      -- Compute mean
      for X of Xs loop
         Sum := Sum + X;
      end loop;
      Mean := Sum / Float(Xs'Length);
      
      -- Collect indices above mean
      for I in Xs'Range loop
         if Xs(I) > Mean then
            Result.Append(I);
         end if;
      end loop;
      
      return Result;
   end Above_Mean;
end Stats;
```

Ada separates specification (`.ads`) from body (`.adb`). It uses attributes like `'Length` and `'Range` to query array properties. The generics syntax instantiates the standard vector container. Ada's verbosity reflects its design for safety-critical systems where explicitness aids review and verification.

### Wirth Family Comparison

The Wirth family shares an emphasis on clarity, structure, and strong typing. Within the family, the main variations are in feature richness and application domain.

Pascal is the simplest, designed for teaching. Modula-2 adds modules for larger programs. Ada is the most feature-rich, designed for mission-critical software with contracts, tasks, and extensive type checking.

Our algorithm shows the family's characteristic verbosity: explicit variable declarations, BEGIN/END blocks, and separate passes for counting and filling. This explicitness aids understanding and review but requires more code than functional or array-oriented languages.

---

## Scripting Languages

Scripting languages prioritize developer productivity and rapid iteration, often with dynamic typing and interpreted execution.

### Python

Python emphasizes readability and simplicity, becoming one of the most popular languages worldwide.

```python
def above_mean(xs):
    mean = sum(xs) / len(xs)
    return [i for i, x in enumerate(xs) if x > mean]
```

Python's list comprehension with `enumerate` is remarkably concise. The intent is clear: for each index `i` and value `x` from the enumerated list, keep `i` if `x` exceeds the mean. This three-line solution rivals the brevity of functional languages while remaining readable to newcomers.

For numerical work, NumPy provides array operations:

```python
import numpy as np

def above_mean(xs):
    xs = np.array(xs)
    return np.where(xs > xs.mean())[0]
```

NumPy's approach is essentially APL-style: the comparison `xs > xs.mean()` broadcasts the scalar mean across the array, producing a boolean array. `np.where` returns indices where the array is true.

### Ruby

Ruby emphasizes programmer happiness with elegant syntax and powerful metaprogramming.

```ruby
def above_mean(xs)
  mean = xs.sum.to_f / xs.length
  xs.each_index.select { |i| xs[i] > mean }
end
```

Ruby's `each_index` returns an enumerator of indices, and `select` filters it. The block `{ |i| xs[i] > mean }` is a predicate. Ruby's method chaining makes the data flow clear.

Alternatively, using `each_with_index`:

```ruby
def above_mean(xs)
  mean = xs.sum.to_f / xs.length
  xs.each_with_index.filter_map { |x, i| i if x > mean }
end
```

### JavaScript

JavaScript is the language of the web, running in browsers and on servers via Node.js.

```javascript
function aboveMean(xs) {
    const mean = xs.reduce((a, b) => a + b, 0) / xs.length;
    return xs.flatMap((x, i) => x > mean ? [i] : []);
}
```

JavaScript lacks a built-in `sum`, so we use `reduce`. The `flatMap` with conditional array return is a filter-map idiom: returning `[i]` includes the index, returning `[]` excludes it.

More traditionally:

```javascript
function aboveMean(xs) {
    const mean = xs.reduce((a, b) => a + b, 0) / xs.length;
    return xs.map((x, i) => [x, i])
             .filter(([x, _]) => x > mean)
             .map(([_, i]) => i);
}
```

### TypeScript

TypeScript adds static typing to JavaScript.

```typescript
function aboveMean(xs: number[]): number[] {
    const mean = xs.reduce((a, b) => a + b, 0) / xs.length;
    return xs.flatMap((x, i) => x > mean ? [i] : []);
}
```

The implementation is identical to JavaScript, but with type annotations. TypeScript catches type errors at compile time while outputting plain JavaScript.

### Perl

Perl is known for its text processing power and "there's more than one way to do it" philosophy.

```perl
sub above_mean {
    my @xs = @_;
    my $mean = (sum @xs) / @xs;
    grep { $xs[$_] > $mean } 0 .. $#xs;
}

use List::Util qw(sum);
```

Perl's `grep` filters a list by a predicate. The `0 .. $#xs` generates indices from 0 to the last index. The sigils (`@` for array, `$` for scalar) indicate variable types. This is concise but requires familiarity with Perl's conventions.

### Lua

Lua is a lightweight language designed for embedding, popular in game development.

```lua
function above_mean(xs)
    -- Compute mean
    local sum = 0
    for _, x in ipairs(xs) do
        sum = sum + x
    end
    local mean = sum / #xs
    
    -- Collect indices (Lua uses 1-based indexing)
    local result = {}
    for i, x in ipairs(xs) do
        if x > mean then
            table.insert(result, i)
        end
    end
    return result
end
```

Lua uses tables for all data structures and has minimal built-ins. The `#` operator returns table length. Lua uses one-based indexing. The code is straightforward but requires explicit loops.

### Scripting Language Comparison

Scripting languages share a focus on developer productivity, dynamic typing (except TypeScript), and interpreted execution. Within the family, variations include syntax aesthetics, built-in features, and application domains.

Python prioritizes readability with significant whitespace and comprehensive standard library. Ruby emphasizes elegance and metaprogramming. JavaScript is constrained by browser compatibility but has evolved powerful functional features. Perl maximizes conciseness at the cost of readability. Lua minimizes language size for embedding.

Our algorithm shows Python and Ruby achieving near-functional brevity with comprehensions and method chaining. JavaScript requires more ceremony without built-in `sum` or `enumerate`. Perl's dense syntax is powerful but cryptic. Lua's minimalism requires explicit loops.

---

## The Apple Ecosystem

### Swift

Swift is Apple's modern language for iOS, macOS, and beyond, designed for safety and expressiveness.

```swift
func aboveMean(_ xs: [Double]) -> [Int] {
    let mean = xs.reduce(0, +) / Double(xs.count)
    return xs.enumerated()
             .filter { $0.element > mean }
             .map { $0.offset }
}
```

Swift's `enumerated()` returns tuples of `(offset: Int, element: Element)`. The `$0` syntax refers to the first closure parameter. Swift infers types and provides functional collection methods similar to modern JavaScript or Ruby.

Alternatively, using `compactMap`:

```swift
func aboveMean(_ xs: [Double]) -> [Int] {
    let mean = xs.reduce(0, +) / Double(xs.count)
    return xs.indices.filter { xs[$0] > mean }
}
```

---

## Historical Languages

These languages, while less commonly used today for new projects, shaped the development of programming and remain important for understanding the field's evolution.

### FORTRAN

FORTRAN (FORmula TRANslation) was the first widely used high-level language, created in the 1950s for scientific computing.

```fortran
function above_mean(xs, n, result, max_result) result(count)
    implicit none
    integer, intent(in) :: n, max_result
    real(8), intent(in) :: xs(n)
    integer, intent(out) :: result(max_result)
    integer :: count
    real(8) :: mean
    integer :: i
    
    ! Compute mean
    mean = sum(xs) / n
    
    ! Collect indices (FORTRAN uses 1-based indexing)
    count = 0
    do i = 1, n
        if (xs(i) > mean) then
            count = count + 1
            result(count) = i
        end if
    end do
end function
```

Modern Fortran (90 and later) has array operations like `sum()`, but dynamic allocation still requires care. FORTRAN uses one-based indexing and traditionally uses uppercase (though modern Fortran is case-insensitive). The `intent` declarations specify parameter directions.

### COBOL

COBOL (Common Business-Oriented Language) was designed for business data processing and remains in use in legacy financial systems.

```cobol
       IDENTIFICATION DIVISION.
       PROGRAM-ID. ABOVE-MEAN.
       
       DATA DIVISION.
       WORKING-STORAGE SECTION.
       01 WS-SUM           PIC 9(10)V99 VALUE 0.
       01 WS-MEAN          PIC 9(10)V99.
       01 WS-COUNT         PIC 9(5) VALUE 0.
       01 WS-INDEX         PIC 9(5).
       01 WS-ARRAY-SIZE    PIC 9(5) VALUE 8.
       01 WS-INPUT-ARRAY.
          05 WS-ELEMENT    PIC 9(5)V99 OCCURS 8 TIMES.
       01 WS-RESULT-ARRAY.
          05 WS-RESULT-IDX PIC 9(5) OCCURS 8 TIMES.
       
       PROCEDURE DIVISION.
           PERFORM VARYING WS-INDEX FROM 1 BY 1 
                   UNTIL WS-INDEX > WS-ARRAY-SIZE
               ADD WS-ELEMENT(WS-INDEX) TO WS-SUM
           END-PERFORM.
           
           DIVIDE WS-SUM BY WS-ARRAY-SIZE GIVING WS-MEAN.
           
           PERFORM VARYING WS-INDEX FROM 1 BY 1
                   UNTIL WS-INDEX > WS-ARRAY-SIZE
               IF WS-ELEMENT(WS-INDEX) > WS-MEAN
                   ADD 1 TO WS-COUNT
                   MOVE WS-INDEX TO WS-RESULT-IDX(WS-COUNT)
               END-IF
           END-PERFORM.
           
           STOP RUN.
```

COBOL's English-like verbosity reflects its design for business readability. The DATA DIVISION declares all variables with explicit picture clauses (PIC) specifying their format. Fixed-size arrays must be pre-declared. COBOL uses one-based indexing.

### BASIC

BASIC (Beginner's All-purpose Symbolic Instruction Code) was designed for teaching and became the entry point for a generation of programmers.

```basic
10 DIM XS(8), RESULT(8)
20 DATA 3, 1, 4, 1, 5, 9, 2, 6
30 FOR I = 1 TO 8: READ XS(I): NEXT I
40 REM Compute mean
50 SUM = 0
60 FOR I = 1 TO 8: SUM = SUM + XS(I): NEXT I
70 MEAN = SUM / 8
80 REM Collect indices above mean
90 COUNT = 0
100 FOR I = 1 TO 8
110   IF XS(I) > MEAN THEN COUNT = COUNT + 1: RESULT(COUNT) = I
120 NEXT I
130 REM Print results
140 FOR I = 1 TO COUNT: PRINT RESULT(I); : NEXT I
150 END
```

Classic BASIC uses line numbers, single-letter variables, and minimal structure. Modern BASIC dialects (Visual Basic, etc.) are much more capable, but this shows the original flavor. BASIC uses one-based indexing.

### Algol 60

Algol 60 was hugely influential in language design, introducing block structure, lexical scoping, and BNF notation for syntax specification.

```algol
begin
    real array xs[1:8];
    integer array result[1:8];
    real sum, mean;
    integer n, count, i;
    
    n := 8;
    comment Initialize array;
    xs[1] := 3; xs[2] := 1; xs[3] := 4; xs[4] := 1;
    xs[5] := 5; xs[6] := 9; xs[7] := 2; xs[8] := 6;
    
    comment Compute mean;
    sum := 0;
    for i := 1 step 1 until n do
        sum := sum + xs[i];
    mean := sum / n;
    
    comment Collect indices above mean;
    count := 0;
    for i := 1 step 1 until n do
        if xs[i] > mean then begin
            count := count + 1;
            result[count] := i
        end;
end
```

Algol 60 introduced `begin`/`end` blocks, the `for` loop with `step` and `until`, and `:=` for assignment. Its influence is visible in Pascal, C, and nearly all subsequent languages. Algol uses one-based indexing.

### Historical Language Comparison

Historical languages reflect the constraints and concerns of their eras. FORTRAN optimized for mathematical computation on limited hardware. COBOL prioritized business readability and data description. BASIC democratized programming for beginners. Algol advanced programming language theory.

All require explicit loops and pre-declared array sizes. Memory management is minimal (garbage collection wasn't practical). The verbosity varies from COBOL's extreme readability to BASIC's terseness, but all share a procedural, step-by-step character.

These languages' influence persists: FORTRAN's array notation appears in NumPy, COBOL's record structures influenced SQL, BASIC's accessibility inspired languages like Python, and Algol's block structure and syntax notation became universal.

---

## Other Notable Languages

Several languages don't fit neatly into families but offer interesting perspectives on our algorithm.

### Forth

Forth is a stack-based language known for its minimalism and extensibility, used in embedded systems and astronomy.

```forth
: above-mean ( addr n -- result-addr count )
  dup >r                     \ Save n for later
  0e                         \ Initialize sum on FP stack
  0 do                       \ Loop n times
    dup i floats + f@ f+     \ Add xs[i] to sum
  loop
  r@ s>f f/                  \ Divide by n for mean
  
  here r@                    \ Start result array, get n
  0 swap                     \ Initialize count
  0 do                       \ Loop through array
    over i floats + f@       \ Get xs[i]
    fdup fover f>            \ Compare with mean
    if
      here over floats + i swap !  \ Store index
      1+                     \ Increment count
    then
    fdrop
  loop
  fdrop fdrop                \ Clean up FP stack
  r> drop                    \ Clean up return stack
;
```

Forth uses postfix notation with an explicit stack. The backslash begins comments. Words like `dup`, `swap`, and `over` manipulate the stack. This code is more illustrative than idiomatic; real Forth programmers would factor it into smaller words.

### Prolog

Prolog is a logic programming language where you describe relationships and let the system find solutions.

```prolog
above_mean(Xs, Indices) :-
    sum_list(Xs, Sum),
    length(Xs, N),
    Mean is Sum / N,
    findall(I, (nth0(I, Xs, X), X > Mean), Indices).
```

Prolog describes *what* we want rather than *how* to compute it. `findall` collects all solutions: indices `I` where the `I`th element of `Xs` is greater than `Mean`. The `nth0` predicate uses zero-based indexing. Prolog backtracks to find all valid indices.

### Smalltalk

Smalltalk pioneered object-oriented programming with message passing and influenced countless languages.

```smalltalk
aboveMean: xs
    | mean |
    mean := xs sum / xs size.
    xs withIndexCollect: [:x :i | 
        x > mean ifTrue: [i] ifFalse: [nil]
    ] thenSelect: [:x | x notNil]
```

In Smalltalk, everything is an object receiving messages. The `withIndexCollect:` message iterates with indices, and `thenSelect:` filters. Smalltalk uses one-based indexing. The syntax of keyword messages like `ifTrue:ifFalse:` is distinctive.

A cleaner version:

```smalltalk
aboveMean: xs
    | mean |
    mean := xs average.
    xs withIndexSelect: [:x :i | x > mean] thenCollect: [:x :i | i]
```

### SQL

SQL handles this as a set operation with window functions.

```sql
WITH indexed AS (
    SELECT value, ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) - 1 AS idx
    FROM UNNEST(@xs) AS value
),
stats AS (
    SELECT AVG(value) AS mean FROM indexed
)
SELECT idx 
FROM indexed, stats 
WHERE value > mean
ORDER BY idx;
```

SQL is declarative: we describe the result we want, and the database engine determines how to compute it. `ROW_NUMBER()` assigns indices, `AVG()` computes the mean, and the `WHERE` clause filters. The exact syntax varies by database system.

### MATLAB

MATLAB is designed for numerical computing with built-in matrix operations.

```matlab
function indices = aboveMean(xs)
    indices = find(xs > mean(xs));
end
```

MATLAB's `mean` computes the average, `>` broadcasts the comparison, and `find` returns indices of true values (one-based). This APL-influenced brevity is why MATLAB remains popular in engineering and science despite its proprietary nature.

### R

R is a language for statistical computing, descended from S.

```r
above_mean <- function(xs) {
  which(xs > mean(xs))
}
```

R's `which` returns indices where a condition is true (one-based). Like MATLAB, R's design incorporates array programming ideas. The vectorized comparison `xs > mean(xs)` broadcasts the scalar mean across the vector.

### Julia

Julia aims to solve the "two language problem" by being both high-level and fast.

```julia
function above_mean(xs)
    findall(x -> x > mean(xs), xs)
end
```

Julia's `findall` returns indices where a predicate holds (one-based). The arrow syntax `x -> x > mean(xs)` creates an anonymous function. Julia compiles to efficient native code while maintaining high-level syntax.

More efficiently (computing mean once):

```julia
function above_mean(xs)
    m = mean(xs)
    findall(x -> x > m, xs)
end
```

---

## Cross-Family Comparison

Having seen the algorithm in dozens of languages, we can identify several axes of variation.

### Explicitness vs. Implicitness

The APL family makes iteration completely implicit: operations apply to entire arrays without mentioning loops or indices. Array languages like MATLAB, R, and Julia follow this approach partially. Traditional languages (C, Pascal, Go) require explicit loops. Functional languages (Haskell, OCaml) use explicit higher-order functions that abstract iteration patterns.

This spectrum reflects different beliefs about what programmers should think about. APL advocates argue that iteration is incidental detail; you should think in terms of array transformations. C advocates argue that control flow should be explicit; you should understand exactly what the machine does.

### Static vs. Dynamic Typing

Statically typed languages (Rust, Haskell, Java) catch type errors at compile time but require more annotation. Dynamically typed languages (Python, Ruby, JavaScript) allow faster iteration but defer errors to runtime.

Within statically typed languages, there's variation in how much inference the compiler provides. Haskell and Rust infer most types; Java requires explicit annotations; C requires both type annotations and manual memory management.

### Functional vs. Imperative Style

Functional languages emphasize expressions and immutability: the algorithm composes operations without modifying variables. Imperative languages emphasize statements and mutation: the algorithm executes steps that update state.

Many modern languages support both styles. Scala, Kotlin, and Swift have functional collection operations alongside imperative loops. Python's list comprehensions are functional, but Python equally supports imperative code.

### Verbosity and Ceremony

Languages vary enormously in how much boilerplate surrounds the essential logic. K expresses the algorithm in 18 characters. COBOL requires dozens of lines of declarations. This difference reflects different priorities: K optimizes for expert efficiency; COBOL optimizes for readability by business stakeholders.

Verbosity isn't always inversely correlated with readability. Python is moderately concise but highly readable. Perl can be concise but cryptic. The sweet spot depends on context and audience.

### Memory Management

The C family requires manual memory management. Rust provides safety through ownership. Garbage-collected languages (Java, Python, Go) handle memory automatically. APL languages typically have automatic memory management suited to their array-oriented nature.

Memory management affects the algorithm's structure. In C, we must pre-count results to allocate the right size, requiring two passes. In Python, we can build the result incrementally. In Rust, the ownership system ensures safety while allowing efficient single-pass construction.

### Indexing Convention

Most modern languages use zero-based indexing: the first element is at index 0. Some languages (APL, FORTRAN, Lua, MATLAB, R, Julia, Smalltalk) use one-based indexing: the first element is at index 1. This difference is historical but affects algorithm implementation when indices must be computed or communicated.

### Domain Orientation

Some languages are general-purpose; others target specific domains. MATLAB and R are designed for numerical and statistical computing, making our algorithm trivial. SQL is designed for relational data, making the algorithm possible but awkward. Domain fit matters: the right tool makes work easy, the wrong tool makes it painful.

---

## Conclusion

A single algorithm, expressed in dozens of languages, reveals the rich diversity of approaches to programming. From APL's radical terseness to COBOL's deliberate verbosity, from Haskell's pure functions to C's manual memory management, each language embodies a philosophy about what programming should be.

No single approach is universally best. APL's notation is powerful for array computations but has a steep learning curve. Python's readability aids collaboration but sacrifices some performance. Rust's safety guarantees prevent bugs but require understanding ownership. Go's simplicity aids maintenance but limits expressiveness.

The algorithm itself is simple enough that every language can express it, yet complex enough that different languages reveal their characters. We see how languages handle aggregation (sum), abstraction (computing mean), iteration (examining each element), filtering (keeping matches), and collection (gathering results). These operations recur throughout programming, and how a language handles them shapes what it's like to think and work in that language.

Understanding multiple paradigms makes one a better programmer even when working in a single language. The APL programmer thinks about data transformations. The Haskell programmer thinks about types and composition. The C programmer thinks about memory and performance. Each perspective illuminates problems differently, and the best solutions often combine insights from multiple traditions.

As programming languages continue to evolve, they borrow from each other. Python adds type hints. Java adds lambdas. Rust adds pattern matching. The boundaries between paradigms blur. Yet the diversity remains valuable: different problems call for different tools, and different minds resonate with different notations. The wealth of programming languages is not confusion but richness.
