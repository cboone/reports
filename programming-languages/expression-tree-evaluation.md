---
created: 2026-02-06
---

# Evaluating Expression Trees: A Cross-Language Study

_February 6, 2026_

> Part of a series: [overview](three-algorithms-overview.md) | [indices above mean](indices-above-mean.md) | **this document** | [concurrent pipelines](concurrent-producer-consumer.md)

This document explores a single algorithm implemented across dozens of programming languages, organized by language family. Where the companion document on [finding indices above the mean](indices-above-mean.md) showcased array languages, this algorithm — evaluating a recursive expression tree — rewards languages with algebraic data types, pattern matching, and recursive thinking.

## Table of Contents

1. [The Algorithm](#the-algorithm)
2. [The ML Family](#the-ml-family)
   - [Haskell](#haskell)
   - [Standard ML](#standard-ml)
   - [OCaml](#ocaml)
   - [F#](#f)
   - [Elm](#elm)
   - [PureScript](#purescript)
   - [ML Family Comparison](#ml-family-comparison)
3. [The Lisp Family](#the-lisp-family)
   - [Common Lisp](#common-lisp)
   - [Scheme](#scheme)
   - [Racket](#racket)
   - [Clojure](#clojure)
   - [Lisp Family Comparison](#lisp-family-comparison)
4. [Proof Assistants and Dependently Typed Languages](#proof-assistants-and-dependently-typed-languages)
   - [Lean](#lean)
   - [Agda](#agda)
   - [Idris](#idris)
   - [Coq (Rocq)](#coq-rocq)
   - [Proof Assistant Comparison](#proof-assistant-comparison)
5. [The BEAM Family](#the-beam-family)
   - [Erlang](#erlang)
   - [Elixir](#elixir)
   - [Gleam](#gleam)
   - [BEAM Family Comparison](#beam-family-comparison)
6. [Newer Functional Languages](#newer-functional-languages)
   - [Roc](#roc)
   - [Unison](#unison)
7. [Logic and Relational Languages](#logic-and-relational-languages)
   - [Prolog](#prolog)
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
14. [The APL Family](#the-apl-family)
    - [APL](#apl)
    - [J](#j)
    - [K](#k)
    - [BQN](#bqn)
    - [APL Family Comparison](#apl-family-comparison)
15. [Other Notable Languages](#other-notable-languages)
    - [Forth](#forth)
    - [Smalltalk](#smalltalk)
    - [SQL](#sql)
    - [MATLAB](#matlab)
    - [R](#r)
    - [Julia](#julia)
16. [Historical Languages](#historical-languages)
    - [FORTRAN](#fortran)
    - [COBOL](#cobol)
    - [BASIC](#basic)
    - [Algol 60](#algol-60)
    - [Historical Language Comparison](#historical-language-comparison)
17. [Cross-Family Comparison](#cross-family-comparison)
18. [Conclusion](#conclusion)

---

## The Algorithm

The task we will implement across all languages is this: given a tree representing an arithmetic expression, evaluate it to produce a numeric result.

Our expression language has four node types:

- **Lit(n)** — a literal number n
- **Neg(e)** — the negation of subexpression e
- **Add(e1, e2)** — the sum of two subexpressions
- **Mul(e1, e2)** — the product of two subexpressions

For example, the expression `Mul(Add(Lit(3), Lit(4)), Lit(5))` represents `(3 + 4) × 5`, which evaluates to `35`.

The algorithm is defined recursively:

1. `eval(Lit(n))` = n
2. `eval(Neg(e))` = −eval(e)
3. `eval(Add(e1, e2))` = eval(e1) + eval(e2)
4. `eval(Mul(e1, e2))` = eval(e1) × eval(e2)

This is trivially simple as mathematics. What makes it interesting as a programming exercise is that it requires two things: a way to define a recursive data type (a tree with four kinds of nodes), and a way to dispatch on the node type (deciding what to do for each variant). Different language families have radically different answers to these two requirements, and those differences reveal deep design philosophies.

Languages with algebraic data types (the ML family, Rust, Swift) can define the tree type directly and pattern-match on its variants. Lisp languages represent the tree as nested lists, making the data structure trivial but the dispatch manual. Object-oriented languages represent each node type as a class and use polymorphic dispatch. C-family languages use tagged unions or struct hierarchies. Array languages, designed for flat rectangular data, find recursive trees fundamentally awkward.

This algorithm also serves as a miniature interpreter, and interpreters are one of the most important programs in computer science — from compilers to web browsers to AI inference engines, much of computing is evaluating tree-shaped structures.

Throughout this document, we use the same example expression: `(3 + 4) × 5 = 35`.

---

## The ML Family

The ML family was essentially designed for this problem. Algebraic data types let you define a tree with labeled variants, and pattern matching lets you destructure it exhaustively. The evaluator reads like the mathematical definition, almost character for character.

### Haskell

Haskell's type system and pattern matching make the expression tree evaluator a textbook example — and not by coincidence. Much of Haskell's early development was motivated by programming language research where exactly this kind of tree manipulation is the central activity.

```haskell
data Expr
  = Lit Double
  | Neg Expr
  | Add Expr Expr
  | Mul Expr Expr

eval :: Expr -> Double
eval (Lit n)     = n
eval (Neg e)     = -(eval e)
eval (Add e1 e2) = eval e1 + eval e2
eval (Mul e1 e2) = eval e1 * eval e2
```

The `data` declaration defines four constructors. The `eval` function uses pattern matching to dispatch on each constructor, and the compiler will warn if we forget a case. Every line of the function corresponds directly to one line of the mathematical definition. The type signature `Expr -> Double` documents the function's contract.

Example usage:

```haskell
-- (3 + 4) * 5
example = Mul (Add (Lit 3) (Lit 4)) (Lit 5)
result  = eval example  -- 35.0
```

### Standard ML

Standard ML (SML) predates Haskell and was one of the first languages to formalize algebraic data types and pattern matching. The syntax is slightly different but the structure is identical.

```sml
datatype expr =
    Lit of real
  | Neg of expr
  | Add of expr * expr
  | Mul of expr * expr

fun eval (Lit n)       = n
  | eval (Neg e)       = ~(eval e)
  | eval (Add (e1,e2)) = eval e1 + eval e2
  | eval (Mul (e1,e2)) = eval e1 * eval e2
```

SML uses `datatype` instead of `data`, tuples `(e1, e2)` for multi-argument constructors, and `~` for numeric negation. The `fun` keyword introduces a function with pattern-matching clauses separated by `|`. SML's pattern matching, like Haskell's, is exhaustive — the compiler warns if cases are missing.

### OCaml

OCaml adds an object system to the ML core, but for this problem we use the algebraic data type side. OCaml is notable for being practical — it is used at Jane Street for financial trading and at Meta for the Hack type checker.

```ocaml
type expr =
  | Lit of float
  | Neg of expr
  | Add of expr * expr
  | Mul of expr * expr

let rec eval = function
  | Lit n       -> n
  | Neg e       -> -.(eval e)
  | Add (e1,e2) -> eval e1 +. eval e2
  | Mul (e1,e2) -> eval e1 *. eval e2
```

OCaml requires `let rec` for recursive functions and uses `function` to introduce an anonymous match. The dotted operators (`-.`, `+.`, `*.`) distinguish floating-point arithmetic from integer arithmetic — a small syntactic inconvenience that reflects OCaml's pragmatic choice to avoid type classes.

### F#

F# brings ML-family algebraic data types to the .NET ecosystem. It is the most natural of the .NET languages for this task.

```fsharp
type Expr =
    | Lit of float
    | Neg of Expr
    | Add of Expr * Expr
    | Mul of Expr * Expr

let rec eval = function
    | Lit n       -> n
    | Neg e       -> -(eval e)
    | Add (e1,e2) -> eval e1 + eval e2
    | Mul (e1,e2) -> eval e1 * eval e2
```

F# follows OCaml's syntax closely, with minor differences: `float` instead of `float`, unified arithmetic operators (no dotted variants), and indentation-sensitive syntax. F# compiles to .NET IL and can interoperate with C# libraries, but for this purely functional algorithm, it looks and feels like its ML ancestors.

### Elm

Elm targets front-end web development with a focus on beginner-friendliness and zero runtime exceptions. Its type system is ML-derived but simplified.

```elm
type Expr
    = Lit Float
    | Neg Expr
    | Add Expr Expr
    | Mul Expr Expr

eval : Expr -> Float
eval expr =
    case expr of
        Lit n       -> n
        Neg e       -> -(eval e)
        Add e1 e2   -> eval e1 + eval e2
        Mul e1 e2   -> eval e1 * eval e2
```

Elm requires an explicit `case` expression rather than allowing pattern matching directly in function definitions. Multi-argument constructors take arguments without tuples. The compiler enforces exhaustive matching. The Elm version reads clearly even to non-ML programmers, which reflects its design goal of accessibility.

### PureScript

PureScript is a strict, purely functional language targeting JavaScript. Its syntax is close to Haskell but with strict evaluation by default.

```purescript
data Expr
  = Lit Number
  | Neg Expr
  | Add Expr Expr
  | Mul Expr Expr

eval :: Expr -> Number
eval (Lit n)     = n
eval (Neg e)     = -(eval e)
eval (Add e1 e2) = eval e1 + eval e2
eval (Mul e1 e2) = eval e1 * eval e2
```

PureScript is nearly identical to Haskell here. The differences emerge in other areas: PureScript has strict evaluation, row polymorphism, and a different approach to effects. For a pure, recursive function over an algebraic data type, it is indistinguishable.

### ML Family Comparison

Across the ML family, the expression tree evaluator is remarkably uniform. Every language defines the type in about four lines and the evaluator in four more. The differences are syntactic details: `data` vs. `datatype` vs. `type`, `->` vs. `=` in match arms, tupled vs. curried constructors.

This uniformity is not a coincidence. The ML family was designed at the intersection of type theory and programming language implementation. Evaluating a recursive data structure by structural recursion is the canonical use case. If the "indices above mean" algorithm made the APL family look effortless and other families look labored, this algorithm does the reverse — the ML family is perfectly at home, while array languages must improvise.

---

## The Lisp Family

The Lisp family takes a fundamentally different approach to the same problem. Rather than defining a new type for expressions, Lisp represents them as nested lists — the same data structure that Lisp programs themselves are made of. This property, called homoiconicity, means that the tree is already a native data structure, but the dispatch on node types must be done manually with conditionals.

### Common Lisp

Common Lisp is a large, multi-paradigm language with deep metaprogramming facilities.

```lisp
;; Expression: (mul (add (lit 3) (lit 4)) (lit 5))
;; Represented as nested lists: (mul (add (lit 3) (lit 4)) (lit 5))

(defun eval-expr (expr)
  (cond
    ((eq (car expr) 'lit)
     (cadr expr))
    ((eq (car expr) 'neg)
     (- (eval-expr (cadr expr))))
    ((eq (car expr) 'add)
     (+ (eval-expr (cadr expr))
        (eval-expr (caddr expr))))
    ((eq (car expr) 'mul)
     (* (eval-expr (cadr expr))
        (eval-expr (caddr expr))))))
```

Expressions are lists where the first element (accessed with `car`) is a symbol naming the operation, and the remaining elements (accessed with `cadr`, `caddr`) are the operands. The `cond` form tests each case in sequence. There is no static type to enforce that all cases are covered — the programmer must be disciplined.

An alternative style uses pattern-matching via `destructuring-bind`:

```lisp
(defun eval-expr (expr)
  (ecase (car expr)
    (lit (cadr expr))
    (neg (- (eval-expr (cadr expr))))
    (add (+ (eval-expr (cadr expr))
            (eval-expr (caddr expr))))
    (mul (* (eval-expr (cadr expr))
            (eval-expr (caddr expr))))))
```

The `ecase` form signals an error if no case matches, providing a degree of exhaustiveness checking at runtime.

Because Lisp code is itself nested lists, one could also represent expressions as actual Lisp expressions `(+ (* 3 4) 5)` and evaluate them with the built-in `eval`. This blurring of code and data is Lisp's deepest idea.

### Scheme

Scheme is a minimalist Lisp dialect that emphasizes a small, clean core.

```scheme
(define (eval-expr expr)
  (case (car expr)
    ((lit) (cadr expr))
    ((neg) (- (eval-expr (cadr expr))))
    ((add) (+ (eval-expr (cadr expr))
              (eval-expr (caddr expr))))
    ((mul) (* (eval-expr (cadr expr))
              (eval-expr (caddr expr))))))
```

Scheme's `case` form matches the tag against literal symbols. The code is structurally identical to the Common Lisp version but slightly cleaner — Scheme's minimalism means fewer syntactic options and more consistency.

### Racket

Racket, evolved from Scheme, adds a `match` form that provides structural pattern matching closer to the ML style.

```racket
#lang racket

(define (eval-expr expr)
  (match expr
    [(list 'lit n)       n]
    [(list 'neg e)       (- (eval-expr e))]
    [(list 'add e1 e2)   (+ (eval-expr e1) (eval-expr e2))]
    [(list 'mul e1 e2)   (* (eval-expr e1) (eval-expr e2))]))
```

Racket's `match` gives the code a remarkably ML-like feel. The pattern `(list 'lit n)` matches a list whose first element is the symbol `lit` and binds the second element to `n`. This is pattern matching over untyped data — we get the syntactic clarity of ML without the static type guarantees.

### Clojure

Clojure brings Lisp to the JVM with an emphasis on immutable data and practical pragmatism.

```clojure
(defn eval-expr [[op & args]]
  (case op
    :lit (first args)
    :neg (- (eval-expr (first args)))
    :add (+ (eval-expr (first args)) (eval-expr (second args)))
    :mul (* (eval-expr (first args)) (eval-expr (second args)))))
```

Clojure destructures the expression directly in the parameter list: `[op & args]` binds `op` to the first element and `args` to the rest. Keywords (`:lit`, `:neg`) replace symbols as tags, which is idiomatic in Clojure. This version is compact and reads clearly.

Example usage:

```clojure
(eval-expr [:mul [:add [:lit 3] [:lit 4]] [:lit 5]])
;; => 35
```

Clojure uses vectors `[]` rather than lists `()` for data, since vectors support efficient random access and are the idiomatic Clojure data structure.

### Lisp Family Comparison

The Lisp family trades static type safety for representational simplicity. There is no type definition at all — expressions are just lists. The evaluator is a function that inspects the first element of a list and recurses. The code is concise and clear, though it lacks the compiler's guarantee that all cases are handled.

The deeper Lisp insight is homoiconicity. Because expressions are lists and Lisp code is lists, the boundary between data and program dissolves. One could represent arithmetic expressions as raw Lisp `(+ (* 3 4) 5)` and evaluate them with the built-in `eval`. Macros can transform expression trees at compile time. This flexibility is why Lisp has been the language of choice for AI research and language experimentation for decades.

Compared to the ML family: ML trades representational flexibility for static guarantees (exhaustive matching, type safety). Lisp trades static guarantees for representational flexibility (any list can be an expression, code is data). Both are elegant; they represent different philosophies about where safety should live.

---

## Proof Assistants and Dependently Typed Languages

The proof assistants take the ML approach further: not only can you define the expression type and write the evaluator, you can *prove* that the evaluator is correct with respect to a specification. For this algorithm, the type definitions and evaluators look similar to Haskell, but the real power emerges when we add properties.

### Lean

Lean 4 is both a theorem prover and a general-purpose programming language.

```lean
inductive Expr where
  | lit : Float → Expr
  | neg : Expr → Expr
  | add : Expr → Expr → Expr
  | mul : Expr → Expr → Expr

def eval : Expr → Float
  | .lit n     => n
  | .neg e     => -(eval e)
  | .add e1 e2 => eval e1 + eval e2
  | .mul e1 e2 => eval e1 * eval e2
```

The `inductive` keyword defines an algebraic data type, and the dot-notation `.lit` is shorthand for `Expr.lit`. Lean's termination checker verifies that the recursion is well-founded (each recursive call is on a structurally smaller argument). Beyond this evaluator, Lean could express and prove theorems like "for all expressions e, eval (neg (neg e)) = eval e."

### Agda

Agda is a dependently typed language used in programming language research. It uses Unicode extensively and has a distinctive syntax.

```agda
data Expr : Set where
  lit : Float → Expr
  neg : Expr → Expr
  add : Expr → Expr → Expr
  mul : Expr → Expr → Expr

eval : Expr → Float
eval (lit n)     = n
eval (neg e)     = - (eval e)
eval (add e1 e2) = eval e1 + eval e2
eval (mul e1 e2) = eval e1 * eval e2
```

Agda's syntax is close to Haskell's, with explicit universe annotations (`Set`) and a more principled approach to pattern matching. Agda enforces totality: every function must handle every case and must terminate. The evaluator passes both checks trivially, since it is structurally recursive and exhaustive.

### Idris

Idris is designed to bring dependent types to general-purpose programming, with a pragmatic focus.

```idris
data Expr
  = Lit Double
  | Neg Expr
  | Add Expr Expr
  | Mul Expr Expr

eval : Expr -> Double
eval (Lit n)     = n
eval (Neg e)     = -(eval e)
eval (Add e1 e2) = eval e1 + eval e2
eval (Mul e1 e2) = eval e1 * eval e2
```

Idris looks nearly identical to Haskell here. Its dependent type features become relevant when you want to prove properties of the evaluator — for instance, defining an expression type indexed by its value, so that the type system guarantees correct evaluation.

### Coq (Rocq)

Coq is one of the oldest and most established proof assistants. It uses a tactic-based approach to proofs.

```coq
Inductive Expr : Type :=
  | Lit : nat -> Expr
  | Neg : Expr -> Expr
  | Add : Expr -> Expr -> Expr
  | Mul : Expr -> Expr -> Expr.

Fixpoint eval (e : Expr) : nat :=
  match e with
  | Lit n     => n
  | Neg e     => eval e    (* simplified: negation on nat is not standard *)
  | Add e1 e2 => eval e1 + eval e2
  | Mul e1 e2 => eval e1 * eval e2
  end.
```

Coq uses `Inductive` and `Fixpoint` (for recursive functions) with explicit type annotations. The `match ... with ... end` syntax is more verbose than Haskell's pattern matching. In a real Coq development, we would use `Z` (integers) rather than `nat` for proper negation, and we could then prove properties like commutativity of addition under `eval`.

### Proof Assistant Comparison

For the basic evaluator, proof assistants look like slightly more verbose versions of Haskell. Their power becomes apparent in what they can do *beyond* evaluation: proving that optimizations preserve semantics, that two different evaluators agree, or that certain expression patterns never arise. The expression tree evaluator is the canonical introductory example in proof assistant tutorials for good reason — it is simple enough to be approachable but structured enough to support interesting proofs.

---

## The BEAM Family

The BEAM languages handle this problem with pattern matching on tagged tuples — a dynamically typed analogue of algebraic data types.

### Erlang

Erlang represents expression trees as tagged tuples, with atoms serving as the tags.

```erlang
eval({lit, N})      -> N;
eval({neg, E})      -> -eval(E);
eval({add, E1, E2}) -> eval(E1) + eval(E2);
eval({mul, E1, E2}) -> eval(E1) * eval(E2).
```

Erlang's function clauses with pattern matching make this remarkably clean. Each clause matches a tuple shape and binds variables. The syntax is close to the mathematical definition. Variables are capitalized (Erlang convention), and clauses are separated by semicolons with a final period.

Example:

```erlang
eval({mul, {add, {lit, 3}, {lit, 4}}, {lit, 5}}).
%% => 35
```

### Elixir

Elixir provides the same pattern matching with a more modern syntax.

```elixir
defmodule Expr do
  def eval({:lit, n}),      do: n
  def eval({:neg, e}),      do: -eval(e)
  def eval({:add, e1, e2}), do: eval(e1) + eval(e2)
  def eval({:mul, e1, e2}), do: eval(e1) * eval(e2)
end
```

Elixir wraps the function in a module (required in Elixir) and uses atoms prefixed with `:` rather than bare atoms. The `do:` shorthand keeps single-expression clauses on one line. The structure mirrors Erlang exactly — Elixir is syntactic sugar over the same BEAM runtime.

### Gleam

Gleam adds static types to the BEAM ecosystem, bringing ML-style type definitions.

```gleam
pub type Expr {
  Lit(Float)
  Neg(Expr)
  Add(Expr, Expr)
  Mul(Expr, Expr)
}

pub fn eval(expr: Expr) -> Float {
  case expr {
    Lit(n)       -> n
    Neg(e)       -> 0.0 -. eval(e)
    Add(e1, e2)  -> eval(e1) +. eval(e2)
    Mul(e1, e2)  -> eval(e1) *. eval(e2)
  }
}
```

Gleam looks much more like an ML language than like Erlang. It has proper algebraic data types with a `case` expression and exhaustive matching. The dotted operators (`-.`, `+.`, `*.`) distinguish float operations, as in OCaml. Gleam compiles to BEAM bytecode but catches type errors at compile time.

### BEAM Family Comparison

Erlang and Elixir handle this problem elegantly through pattern matching on dynamic data — there is no static type definition, just conventions about tuple shapes. Gleam bridges the gap, offering ML-style types on the BEAM runtime. All three benefit from pattern matching in function heads, making the evaluator concise and readable. The BEAM family is not as tailor-made for this problem as the ML family, but it handles it well — pattern matching is a deep tradition in functional programming, and all three BEAM languages embrace it.

---

## Newer Functional Languages

### Roc

Roc is a functional language focused on applications, with tag unions that make the tree definition lightweight.

```roc
Expr : [
    Lit F64,
    Neg Expr,
    Add Expr Expr,
    Mul Expr Expr,
]

eval : Expr -> F64
eval = \expr ->
    when expr is
        Lit n     -> n
        Neg e     -> -(eval e)
        Add e1 e2 -> eval e1 + eval e2
        Mul e1 e2 -> eval e1 * eval e2
```

Roc's tag unions are like anonymous algebraic data types — you do not need to name the type separately from its constructors. The `when ... is` expression provides pattern matching with exhaustiveness checking. Roc aims for a pragmatic, accessible functional language, and this algorithm shows its clarity.

### Unison

Unison is a content-addressed language where code is stored as a hash-identified AST rather than text files.

```unison
unique type Expr
  = Lit Float
  | Neg Expr
  | Add Expr Expr
  | Mul Expr Expr

eval : Expr -> Float
eval = cases
  Lit n     -> n
  Neg e     -> Float.negate (eval e)
  Add e1 e2 -> eval e1 + eval e2
  Mul e1 e2 -> eval e1 * eval e2
```

Unison's `cases` keyword introduces a pattern-matching lambda. The syntax is Haskell-flavored. Unison's unique contribution is that once this function is defined, it is stored by its content hash, making it permanently cacheable and shareable. But for this algorithm, it reads like another member of the ML family.

---

## Logic and Relational Languages

### Prolog

Prolog turns the evaluator into a logical relation between expressions and values. This is a fundamentally different approach — rather than computing a result, we declare what it means for a result to be correct.

```prolog
eval(lit(N), N).
eval(neg(E), V) :-
    eval(E, V1),
    V is -V1.
eval(add(E1, E2), V) :-
    eval(E1, V1),
    eval(E2, V2),
    V is V1 + V2.
eval(mul(E1, E2), V) :-
    eval(E1, V1),
    eval(E2, V2),
    V is V1 * V2.
```

Each clause states a fact or rule. `eval(lit(N), N)` declares that a literal evaluates to its value. The other clauses state that an addition evaluates to `V` *if* its subexpressions evaluate to `V1` and `V2` *and* `V is V1 + V2`.

Example query:

```prolog
?- eval(mul(add(lit(3), lit(4)), lit(5)), Result).
Result = 35.
```

The remarkable thing about Prolog's approach is that the relation can sometimes be run in reverse. While we cannot easily ask "what expression evaluates to 35?" with this definition (due to `is` requiring ground arithmetic), a pure relational version using constraint logic programming (CLP) could enumerate expressions that produce a given result. This bidirectionality has no equivalent in any other paradigm.

Prolog's representation of expressions uses compound terms like `add(lit(3), lit(4))`, which are tree-shaped by nature. Prolog was designed for symbolic reasoning over tree structures, making this problem a natural fit.

---

## The JVM Family

The JVM languages show an interesting evolution in handling this problem. Java uses the classic object-oriented approach; Scala bridges OOP and FP; Kotlin adds sealed types and modern syntax.

### Java

Java's traditional approach uses polymorphism: each expression type is a class with an `eval` method.

```java
sealed interface Expr permits Lit, Neg, Add, Mul {}

record Lit(double value) implements Expr {}
record Neg(Expr expr) implements Expr {}
record Add(Expr left, Expr right) implements Expr {}
record Mul(Expr left, Expr right) implements Expr {}

static double eval(Expr expr) {
    return switch (expr) {
        case Lit(var n)           -> n;
        case Neg(var e)           -> -eval(e);
        case Add(var e1, var e2)  -> eval(e1) + eval(e2);
        case Mul(var e1, var e2)  -> eval(e1) * eval(e2);
    };
}
```

Modern Java (21+) with sealed interfaces, records, and pattern matching in switch expressions reads remarkably close to the ML versions. The `sealed` keyword ensures exhaustiveness — the compiler knows all possible implementations of `Expr` and can verify that the switch covers every case. This is Java adopting ML-family ideas after decades of relying on the visitor pattern.

The classic pre-Java-21 approach would use the visitor pattern:

```java
interface ExprVisitor<T> {
    T visitLit(double value);
    T visitNeg(Expr expr);
    T visitAdd(Expr left, Expr right);
    T visitMul(Expr left, Expr right);
}

interface Expr {
    <T> T accept(ExprVisitor<T> visitor);
}
```

This requires far more ceremony to achieve what pattern matching provides directly. The evolution from visitor pattern to sealed interfaces and pattern matching represents Java catching up to ideas the ML family has had since the 1970s.

### Scala

Scala has had sealed traits and pattern matching since its inception.

```scala
enum Expr:
  case Lit(n: Double)
  case Neg(e: Expr)
  case Add(e1: Expr, e2: Expr)
  case Mul(e1: Expr, e2: Expr)

def eval(expr: Expr): Double = expr match
  case Expr.Lit(n)      => n
  case Expr.Neg(e)      => -eval(e)
  case Expr.Add(e1, e2) => eval(e1) + eval(e2)
  case Expr.Mul(e1, e2) => eval(e1) * eval(e2)
```

Scala 3's `enum` provides concise algebraic data types. The `match` expression is ML-style pattern matching with exhaustiveness checking. Scala was designed from the start as a fusion of object-oriented and functional programming, and this algorithm shows the functional side working exactly as in an ML language.

### Kotlin

Kotlin uses sealed classes and `when` expressions for the same pattern.

```kotlin
sealed class Expr
data class Lit(val n: Double) : Expr()
data class Neg(val e: Expr) : Expr()
data class Add(val e1: Expr, val e2: Expr) : Expr()
data class Mul(val e1: Expr, val e2: Expr) : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Lit -> expr.n
    is Neg -> -eval(expr.e)
    is Add -> eval(expr.e1) + eval(expr.e2)
    is Mul -> eval(expr.e1) * eval(expr.e2)
}
```

Kotlin's `when` with `is` checks performs type testing and smart casting. Unlike ML-style pattern matching, Kotlin does not destructure in the `when` clause — instead, it smart-casts `expr` to the matched type and accesses fields by name. This is a pragmatic approach that works well without full pattern matching support.

### JVM Family Comparison

The JVM family shows the gradual adoption of functional programming ideas by mainstream languages. Scala had ADTs from the start. Java added them decades later through sealed interfaces and records. Kotlin sits in between with sealed classes and smart casts. For this specific algorithm, all three produce clean, readable code — but the path they took to get there reflects very different design philosophies about how quickly to adopt new paradigms.

---

## The C Family

The C family must represent the expression tree using manual memory allocation and tagged unions or struct hierarchies. This is where the algorithm's recursive nature creates real engineering work.

### C

C represents the tree using a tagged union (sometimes called a discriminated union).

```c
typedef enum { LIT, NEG, ADD, MUL } ExprTag;

typedef struct Expr {
    ExprTag tag;
    union {
        double lit;
        struct Expr *neg;
        struct { struct Expr *left; struct Expr *right; } binop;
    };
} Expr;

double eval(Expr *e) {
    switch (e->tag) {
        case LIT: return e->lit;
        case NEG: return -eval(e->neg);
        case ADD: return eval(e->binop.left) + eval(e->binop.right);
        case MUL: return eval(e->binop.left) * eval(e->binop.right);
    }
    return 0; /* unreachable, but silences compiler warning */
}
```

The `Expr` struct contains a tag field and a union of possible payloads. The evaluator switches on the tag and accesses the appropriate union member. C provides no guarantee that the tag and the union member are consistent — accessing `e->lit` when the tag is `ADD` would read garbage without any compiler error. The programmer must maintain this invariant manually.

Constructing expressions requires heap allocation:

```c
Expr *lit(double n) {
    Expr *e = malloc(sizeof(Expr));
    e->tag = LIT;
    e->lit = n;
    return e;
}
```

Each constructor function allocates memory, which must eventually be freed. A real implementation needs a corresponding `free_expr` function that recursively frees the tree — the kind of bookkeeping that higher-level languages handle automatically.

### C++

Modern C++ provides `std::variant` and `std::visit` for type-safe tagged unions.

```cpp
#include <variant>
#include <memory>

struct Expr;
using ExprPtr = std::shared_ptr<Expr>;

struct Lit { double value; };
struct Neg { ExprPtr expr; };
struct Add { ExprPtr left; ExprPtr right; };
struct Mul { ExprPtr left; ExprPtr right; };

struct Expr : std::variant<Lit, Neg, Add, Mul> {
    using variant::variant;
};

double eval(const ExprPtr& e) {
    return std::visit([](const auto& node) -> double {
        using T = std::decay_t<decltype(node)>;
        if constexpr (std::is_same_v<T, Lit>) return node.value;
        else if constexpr (std::is_same_v<T, Neg>) return -eval(node.expr);
        else if constexpr (std::is_same_v<T, Add>) return eval(node.left) + eval(node.right);
        else if constexpr (std::is_same_v<T, Mul>) return eval(node.left) * eval(node.right);
    }, static_cast<const std::variant<Lit, Neg, Add, Mul>&>(*e));
}
```

This is type-safe — `std::variant` ensures only valid access — but the template metaprogramming required for `std::visit` produces dense, hard-to-read code. `shared_ptr` handles memory management. The `if constexpr` chain performs compile-time dispatch on the variant alternative. This approach is safe and efficient, but a far cry from the ML family's clarity.

An alternative C++ approach uses traditional OOP:

```cpp
struct Expr {
    virtual double eval() const = 0;
    virtual ~Expr() = default;
};

struct Lit : Expr {
    double value;
    Lit(double v) : value(v) {}
    double eval() const override { return value; }
};

struct Add : Expr {
    std::unique_ptr<Expr> left, right;
    Add(std::unique_ptr<Expr> l, std::unique_ptr<Expr> r)
        : left(std::move(l)), right(std::move(r)) {}
    double eval() const override { return left->eval() + right->eval(); }
};
// ... Neg and Mul similarly
```

The virtual dispatch approach distributes the `eval` logic across classes rather than centralizing it in a single function. This is the classic "expression problem" tension: pattern matching centralizes operations, virtual dispatch centralizes types.

### Objective-C

Objective-C uses message-passing polymorphism.

```objc
@interface Expr : NSObject
- (double)eval;
@end

@interface Lit : Expr
@property double value;
+ (instancetype)withValue:(double)value;
@end

@implementation Lit
+ (instancetype)withValue:(double)value {
    Lit *e = [[Lit alloc] init];
    e.value = value;
    return e;
}
- (double)eval { return self.value; }
@end

@interface Add : Expr
@property Expr *left;
@property Expr *right;
+ (instancetype)withLeft:(Expr *)left right:(Expr *)right;
@end

@implementation Add
+ (instancetype)withLeft:(Expr *)left right:(Expr *)right {
    Add *e = [[Add alloc] init];
    e.left = left;
    e.right = right;
    return e;
}
- (double)eval { return [self.left eval] + [self.right eval]; }
@end
// ... Neg and Mul similarly
```

Objective-C's message-passing syntax (`[self.left eval]`) is distinctive. Each class implements `eval` as a method, providing polymorphic dispatch through the Objective-C runtime. The verbosity is substantial — each node type requires an interface and implementation with property declarations and factory methods. Reference counting (ARC) handles memory.

### C Family Comparison

The C family reveals what the ML family's pattern matching gives you for free. In C, you manually manage the tagged union, the memory, and the dispatch. In C++, `std::variant` and templates provide safety at the cost of syntactic complexity. Objective-C uses OOP dispatch, which is clean but verbose. The essential algorithm — four cases of structural recursion — is buried under engineering concerns in all three languages.

---

## Modern Systems Languages

### Rust

Rust provides algebraic data types (enums) with pattern matching, combining ML-family expressiveness with systems-level control.

```rust
enum Expr {
    Lit(f64),
    Neg(Box<Expr>),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
}

fn eval(expr: &Expr) -> f64 {
    match expr {
        Expr::Lit(n)      => *n,
        Expr::Neg(e)      => -eval(e),
        Expr::Add(e1, e2) => eval(e1) + eval(e2),
        Expr::Mul(e1, e2) => eval(e1) * eval(e2),
    }
}
```

Rust's `enum` is a proper sum type, and `match` is exhaustive pattern matching — the compiler refuses to compile if a case is missing. The `Box<Expr>` is required because recursive types must have indirection (the compiler needs to know the size of `Expr`, and without `Box` it would be infinite). This is a systems-level concern that ML languages hide behind garbage collection.

The `Box` is the only visible systems programming artifact. Otherwise, this reads like Haskell with different syntax. Rust demonstrates that systems programming and algebraic data types are not mutually exclusive.

### Go

Go has no algebraic data types and no pattern matching. The idiomatic approach uses interfaces.

```go
type Expr interface {
    eval() float64
}

type Lit struct{ value float64 }
type Neg struct{ expr Expr }
type Add struct{ left, right Expr }
type Mul struct{ left, right Expr }

func (l Lit) eval() float64 { return l.value }
func (n Neg) eval() float64 { return -n.expr.eval() }
func (a Add) eval() float64 { return a.left.eval() + a.right.eval() }
func (m Mul) eval() float64 { return m.left.eval() * m.right.eval() }
```

Go uses structural subtyping: any type with an `eval() float64` method satisfies the `Expr` interface. Each struct type implements this method. The code is clear and straightforward, but there is no way to ensure the interface is closed — anyone can add a new `Expr` implementation, and there is no exhaustiveness checking.

Go's philosophy is visible here: simple, explicit, no magic. The code does exactly what it says, with no abstractions that might surprise. For this particular problem, the lack of algebraic data types means Go cannot express the "closed set of four variants" constraint that ML-family languages capture in the type system.

### Zig

Zig uses tagged unions with manual memory control, representing a middle ground between C and Rust.

```zig
const std = @import("std");

const Expr = union(enum) {
    lit: f64,
    neg: *const Expr,
    add: struct { left: *const Expr, right: *const Expr },
    mul: struct { left: *const Expr, right: *const Expr },
};

fn eval(expr: *const Expr) f64 {
    return switch (expr.*) {
        .lit => |n| n,
        .neg => |e| -eval(e),
        .add => |a| eval(a.left) + eval(a.right),
        .mul => |m| eval(m.left) * eval(m.right),
    };
}
```

Zig's tagged unions are similar to Rust's enums but with C-level control over memory layout. The `switch` on a tagged union is exhaustive. Pointers are explicit (no `Box` abstraction), and memory management is manual. Zig sits between C's raw approach and Rust's safe abstractions.

### Systems Language Comparison

Rust is the clear winner for this problem among systems languages, providing ML-family ergonomics with the `Box` indirection as the only visible cost. Go's interface approach is clean but loses the closed-world guarantee. Zig's tagged unions are powerful but more verbose. All three are practical; the differences reflect their positions on the safety-control spectrum.

---

## The Wirth Family

The Wirth family languages use variant records or discriminated types, which are the historical precursors of algebraic data types.

### Pascal

Pascal uses variant records, which are untagged unions with a discriminant field.

```pascal
type
  ExprTag = (etLit, etNeg, etAdd, etMul);
  PExpr = ^Expr;
  Expr = record
    case tag: ExprTag of
      etLit: (value: Real);
      etNeg: (sub: PExpr);
      etAdd: (addLeft, addRight: PExpr);
      etMul: (mulLeft, mulRight: PExpr);
  end;

function Eval(e: PExpr): Real;
begin
  case e^.tag of
    etLit: Eval := e^.value;
    etNeg: Eval := -Eval(e^.sub);
    etAdd: Eval := Eval(e^.addLeft) + Eval(e^.addRight);
    etMul: Eval := Eval(e^.mulLeft) + Eval(e^.mulRight);
  end;
end;
```

Pascal's variant records are essentially C's tagged unions with slightly more structure. The `^` operator dereferences pointers, and `PExpr = ^Expr` declares a pointer type. Field names must be unique across variants, leading to names like `addLeft` and `mulLeft`. Pascal was one of the first widely-taught languages to have variant records, influencing the later development of proper algebraic data types.

### Modula-2

Modula-2's approach is similar to Pascal's, with minor syntactic differences.

```modula2
TYPE
  ExprTag = (lit, neg, add, mul);
  ExprPtr = POINTER TO Expr;
  Expr = RECORD
    CASE tag: ExprTag OF
      lit: value: REAL |
      neg: sub: ExprPtr |
      add: addLeft, addRight: ExprPtr |
      mul: mulLeft, mulRight: ExprPtr
    END;
  END;

PROCEDURE Eval(e: ExprPtr): REAL;
BEGIN
  CASE e^.tag OF
    lit: RETURN e^.value |
    neg: RETURN -Eval(e^.sub) |
    add: RETURN Eval(e^.addLeft) + Eval(e^.addRight) |
    mul: RETURN Eval(e^.mulLeft) * Eval(e^.mulRight)
  END;
END Eval;
```

Modula-2 capitalizes keywords and uses `|` to separate cases. The structure is identical to Pascal. Wirth's languages were explicitly designed for teaching structured programming, and the variant record approach makes the data structure explicit even if it lacks the safety of ML-style types.

### Ada

Ada provides discriminated records with stronger safety guarantees than Pascal.

```ada
type Expr_Tag is (Lit_Tag, Neg_Tag, Add_Tag, Mul_Tag);
type Expr;
type Expr_Ptr is access Expr;

type Expr (Tag : Expr_Tag) is record
   case Tag is
      when Lit_Tag => Value : Float;
      when Neg_Tag => Sub   : Expr_Ptr;
      when Add_Tag => Add_Left, Add_Right : Expr_Ptr;
      when Mul_Tag => Mul_Left, Mul_Right : Expr_Ptr;
   end case;
end record;

function Eval (E : Expr_Ptr) return Float is
begin
   case E.Tag is
      when Lit_Tag => return E.Value;
      when Neg_Tag => return -Eval (E.Sub);
      when Add_Tag => return Eval (E.Add_Left) + Eval (E.Add_Right);
      when Mul_Tag => return Eval (E.Mul_Left) * Eval (E.Mul_Right);
   end case;
end Eval;
```

Ada's discriminated records are safer than Pascal's variant records — accessing a field that does not belong to the current discriminant raises `Constraint_Error` at runtime. The `case` in the type definition specifies which fields exist for each discriminant value. This is the closest any of the Wirth-family languages comes to the safety of algebraic data types, though it is still checked at runtime rather than compile time.

### Wirth Family Comparison

The Wirth family languages are the historical bridge between unstructured tagged unions (C) and algebraic data types (ML). Pascal introduced variant records; Ada added runtime safety checks. None provides compile-time exhaustiveness checking, and all require manual memory management. The code is verbose but explicit, reflecting Wirth's pedagogical emphasis on clarity over brevity.

---

## Scripting Languages

Scripting languages typically represent expression trees using classes, dictionaries, or nested structures. Without static types, the "tree definition" is implicit in how the code constructs and inspects nodes.

### Python

Python's `match` statement (3.10+) provides structural pattern matching.

```python
from dataclasses import dataclass
from typing import Union

@dataclass
class Lit:
    value: float

@dataclass
class Neg:
    expr: 'Expr'

@dataclass
class Add:
    left: 'Expr'
    right: 'Expr'

@dataclass
class Mul:
    left: 'Expr'
    right: 'Expr'

Expr = Union[Lit, Neg, Add, Mul]

def eval_expr(expr: Expr) -> float:
    match expr:
        case Lit(n):
            return n
        case Neg(e):
            return -eval_expr(e)
        case Add(e1, e2):
            return eval_expr(e1) + eval_expr(e2)
        case Mul(e1, e2):
            return eval_expr(e1) * eval_expr(e2)
```

Modern Python with dataclasses and match statements looks surprisingly close to the ML versions. The type annotations are optional (Python does not enforce them at runtime), and the `match` is not exhaustive — missing a case produces no error until runtime. But the readability is excellent, and the structural pattern matching is a direct import from the ML tradition.

A pre-3.10 version would use `isinstance` checks:

```python
def eval_expr(expr):
    if isinstance(expr, Lit):
        return expr.value
    elif isinstance(expr, Neg):
        return -eval_expr(expr.expr)
    elif isinstance(expr, Add):
        return eval_expr(expr.left) + eval_expr(expr.right)
    elif isinstance(expr, Mul):
        return eval_expr(expr.left) * eval_expr(expr.right)
```

### Ruby

Ruby uses classes and case expressions.

```ruby
Lit = Struct.new(:value)
Neg = Struct.new(:expr)
Add = Struct.new(:left, :right)
Mul = Struct.new(:left, :right)

def eval_expr(expr)
  case expr
  when Lit then expr.value
  when Neg then -eval_expr(expr.expr)
  when Add then eval_expr(expr.left) + eval_expr(expr.right)
  when Mul then eval_expr(expr.left) * eval_expr(expr.right)
  end
end
```

Ruby's `Struct` creates simple data classes, and `case/when` dispatches on class. This is concise and idiomatic. Ruby's `case` uses the `===` operator, which for classes performs an `is_a?` check. Ruby also supports a more object-oriented style where each struct defines its own `eval` method.

### JavaScript

JavaScript uses plain objects or classes.

```javascript
const Lit = (n) => ({ type: "lit", value: n });
const Neg = (e) => ({ type: "neg", expr: e });
const Add = (e1, e2) => ({ type: "add", left: e1, right: e2 });
const Mul = (e1, e2) => ({ type: "mul", left: e1, right: e2 });

function evalExpr(expr) {
    switch (expr.type) {
        case "lit": return expr.value;
        case "neg": return -evalExpr(expr.expr);
        case "add": return evalExpr(expr.left) + evalExpr(expr.right);
        case "mul": return evalExpr(expr.left) * evalExpr(expr.right);
    }
}
```

JavaScript represents nodes as plain objects with a `type` string tag, which is the dynamic-language equivalent of a tagged union. Factory functions create nodes. The `switch` on the type string dispatches to each case. There is no structural guarantee that nodes have the right fields — this is entirely convention.

### TypeScript

TypeScript adds discriminated unions with type narrowing.

```typescript
type Expr =
    | { type: "lit"; value: number }
    | { type: "neg"; expr: Expr }
    | { type: "add"; left: Expr; right: Expr }
    | { type: "mul"; left: Expr; right: Expr };

function evalExpr(expr: Expr): number {
    switch (expr.type) {
        case "lit": return expr.value;
        case "neg": return -evalExpr(expr.expr);
        case "add": return evalExpr(expr.left) + evalExpr(expr.right);
        case "mul": return evalExpr(expr.left) * evalExpr(expr.right);
    }
}
```

TypeScript's discriminated union type provides real compile-time safety. The `type` field serves as the discriminant, and TypeScript narrows the type in each `case` branch — accessing `expr.value` is only valid in the `"lit"` case. The compiler can also check exhaustiveness if strict checks are enabled. This is the closest JavaScript gets to algebraic data types.

### Perl

Perl uses hash references with a type tag.

```perl
sub lit { { type => 'lit', value => $_[0] } }
sub neg { { type => 'neg', expr => $_[0] } }
sub add { { type => 'add', left => $_[0], right => $_[1] } }
sub mul { { type => 'mul', left => $_[0], right => $_[1] } }

sub eval_expr {
    my ($expr) = @_;
    my $type = $expr->{type};
    if    ($type eq 'lit') { return $expr->{value} }
    elsif ($type eq 'neg') { return -eval_expr($expr->{expr}) }
    elsif ($type eq 'add') { return eval_expr($expr->{left}) + eval_expr($expr->{right}) }
    elsif ($type eq 'mul') { return eval_expr($expr->{left}) * eval_expr($expr->{right}) }
}
```

Perl uses hash references (`{}`) as lightweight structs with string keys. The arrow operator `->` dereferences. This is structurally identical to the JavaScript approach — tagged objects with manual dispatch — but with Perl's distinctive sigils and syntax.

### Lua

Lua uses tables, its single compound data type.

```lua
function lit(n)    return {type="lit", value=n} end
function neg(e)    return {type="neg", expr=e} end
function add(a, b) return {type="add", left=a, right=b} end
function mul(a, b) return {type="mul", left=a, right=b} end

function eval_expr(expr)
    if     expr.type == "lit" then return expr.value
    elseif expr.type == "neg" then return -eval_expr(expr.expr)
    elseif expr.type == "add" then return eval_expr(expr.left) + eval_expr(expr.right)
    elseif expr.type == "mul" then return eval_expr(expr.left) * eval_expr(expr.right)
    end
end
```

Lua tables are the universal data structure. The code is clean and minimal, reflecting Lua's design as an embeddable extension language. There is no type system, no pattern matching, and no class hierarchy — just tables with string keys and conditional chains.

### Scripting Language Comparison

The scripting languages converge on the same basic approach: objects or dictionaries with a type tag, dispatched via conditionals or switch statements. Python's match statement and TypeScript's discriminated unions bring ML-family ideas into dynamically-flavored ecosystems. Ruby's case/when is clean and idiomatic. JavaScript, Perl, and Lua use the simplest possible approach: tagged dictionaries.

The core difference from the ML family is the absence of compile-time guarantees. No scripting language (except TypeScript) verifies that all cases are handled or that node fields are consistent with node types. This is the trade-off: less ceremony in exchange for less safety.

---

## The Apple Ecosystem

### Swift

Swift provides enums with associated values, which are full algebraic data types.

```swift
indirect enum Expr {
    case lit(Double)
    case neg(Expr)
    case add(Expr, Expr)
    case mul(Expr, Expr)
}

func eval(_ expr: Expr) -> Double {
    switch expr {
    case .lit(let n):
        return n
    case .neg(let e):
        return -eval(e)
    case .add(let e1, let e2):
        return eval(e1) + eval(e2)
    case .mul(let e1, let e2):
        return eval(e1) * eval(e2)
    }
}
```

The `indirect` keyword tells Swift to heap-allocate the recursive cases (like Rust's `Box`, but applied to the entire enum). The `switch` is exhaustive — omitting a case is a compiler error. Swift's approach is nearly identical to Rust's, reflecting their shared lineage from ML-family ideas.

---

## The APL Family

Array languages are designed for flat, rectangular data. Recursive tree structures are fundamentally alien to the array paradigm. Representing and evaluating expression trees in APL-family languages requires encoding the tree as flat arrays — possible, but working against the grain.

### APL

One approach encodes the tree as parallel arrays: one for operations, one for values, and one for child indices.

```apl
⍝ Encoding: ops='L' for lit, 'N' for neg, 'A' for add, 'M' for mul
⍝ vals: value for lit nodes, 0 otherwise
⍝ left, right: child indices (1-based), 0 if none
⍝
⍝ Example: (3+4)×5
⍝ Node 1: Mul, children 2,3
⍝ Node 2: Add, children 4,5
⍝ Node 3: Lit 5
⍝ Node 4: Lit 3
⍝ Node 5: Lit 4

∇ R←EVAL ops vals left right;i;op
  R←vals
  :For i :In ⌽⍳≢ops  ⍝ Process leaves first (reverse order)
    op←i⊃ops
    :Select op
    :Case 'N' ⋄ R[i]←-R[left[i]]
    :Case 'A' ⋄ R[i]←R[left[i]]+R[right[i]]
    :Case 'M' ⋄ R[i]←R[left[i]]×R[right[i]]
    :EndSelect
  :EndFor
  R←R[1]
∇
```

This is deeply unidiomatic APL. The algorithm requires an explicit loop (`:For`) processing nodes in reverse topological order, filling in computed values. The tree has been manually flattened into parallel arrays, losing the structural clarity of the recursive definition. APL's array operations have nothing to contribute here — there is no vectorization opportunity because each node's value depends on its children.

### J

J can use boxed arrays to represent trees, but evaluation still requires recursion.

```j
NB. Represent tree as boxed structures
lit =: 'lit'&;
neg =: 'neg'&; @ <
add =: 'add'&; @ (< @ ,)
mul =: 'mul'&; @ (< @ ,)

eval =: monad define
  tag =. > {. y
  if. tag -: 'lit' do. > {: y
  elseif. tag -: 'neg' do. - eval > {: y
  elseif. tag -: 'add' do. (eval > 0 { > {: y) + (eval > 1 { > {: y)
  elseif. tag -: 'mul' do. (eval > 0 { > {: y) * (eval > 1 { > {: y)
  end.
)
```

J's boxing mechanism (`<` to box, `>` to unbox) creates nested structures, but manipulating them requires constant boxing and unboxing. The explicit `if/elseif` chain is rarely seen in J, which prefers tacit definitions. Trees push J out of its element.

### K

K takes a minimalist approach, using nested lists.

```k
eval:{$[x 0;"lit";x 1
        x 0;"neg";-eval x 1
        x 0;"add";(eval x 1)+eval x 2
        x 0;"mul";(eval x 1)*eval x 2]}
```

K's `$[...]` is a conditional, testing `x 0` (the tag) against each string. The tree is a list where the first element is the operation name. This works, but it is the same tagged-object pattern used by JavaScript and Lua — K's array superpowers are irrelevant here.

### BQN

BQN, like the other array languages, must fall back to recursion and conditionals.

```bqn
Eval ← {
  tag‿payload ← 𝕩
  tag◶⟨
    {𝕩}            # lit: return value
    {-Eval 𝕩}      # neg: negate
    {(Eval⊑𝕩)+Eval 1⊑𝕩}  # add
    {(Eval⊑𝕩)×Eval 1⊑𝕩}  # mul
  ⟩ payload
}
```

BQN uses a list of functions selected by the tag (using `◶`, the "choose" combinator). The tree is a pair of `⟨tag, payload⟩`. This is more idiomatic than the other array languages — BQN's first-class functions and combinators allow a table-dispatch approach — but it is still fundamentally non-array code.

### APL Family Comparison

The APL family's struggle with expression trees is deeply informative. These languages are built around the idea that data is rectangular and operations apply uniformly to every element. Trees are neither rectangular nor uniform — each node has a different number of children, and evaluation must respect parent-child dependencies. The flat-array encoding destroys structural clarity. The recursive approach abandons array operations entirely.

This is the inverse of the "indices above mean" algorithm, where APL was effortless and other languages labored. Different problems genuinely require different paradigms, and forcing array notation onto tree structures is as awkward as forcing explicit loops onto array transformations.

---

## Other Notable Languages

### Forth

Forth's stack-based approach encodes the tree implicitly in the execution order.

```forth
: lit    ( n -- result ) ;   \ literal: value is already on stack
: neg    ( n -- result ) negate ;
: add-e  ( n1 n2 -- result ) + ;
: mul-e  ( n1 n2 -- result ) * ;

\ (3 + 4) * 5 becomes:
\ 3 lit 4 lit add-e 5 lit mul-e
```

In Forth, the expression tree is encoded in the order of operations and the stack implicitly manages the tree structure. Each "node" is a word that operates on the stack: `lit` is a no-op (the number is already pushed), `add-e` pops two values and pushes their sum. The expression `3 lit 4 lit add-e 5 lit mul-e` evaluates to 35. This is reverse Polish notation — the tree is linearized into a post-order traversal and the stack reconstructs the dependencies.

This is both elegant and limiting. There is no explicit tree data structure, so you cannot inspect or transform expressions — only evaluate them. Forth excels at this "immediate evaluation" mode but cannot easily represent deferred computation.

### Smalltalk

Smalltalk uses polymorphic message dispatch — each node type is a class that knows how to evaluate itself.

```smalltalk
Object subclass: #Lit instanceVariableNames: 'value'.
Lit >> eval [ ^ value ]
Lit class >> value: n [ ^ self new value: n ]

Object subclass: #Neg instanceVariableNames: 'expr'.
Neg >> eval [ ^ expr eval negated ]

Object subclass: #Add instanceVariableNames: 'left right'.
Add >> eval [ ^ left eval + right eval ]

Object subclass: #Mul instanceVariableNames: 'left right'.
Mul >> eval [ ^ left eval * right eval ]
```

Each class defines an `eval` method. Sending the `eval` message to any expression dispatches to the correct implementation. Construction and evaluation are both natural message sends: `Add left: (Lit value: 3) right: (Lit value: 4)`. This is the OOP ideal — behavior is distributed among objects, and new operations are added by defining new methods.

### SQL

SQL can evaluate expression trees stored as rows in a table using recursive CTEs.

```sql
-- Nodes stored as rows: id, type, value (for lit), left_id, right_id
WITH RECURSIVE eval_tree AS (
    -- Base case: literals
    SELECT id, value AS result
    FROM nodes WHERE type = 'lit'
    
    UNION ALL
    
    -- Recursive case: nodes whose children are already evaluated
    SELECT n.id,
        CASE n.type
            WHEN 'neg' THEN -c1.result
            WHEN 'add' THEN c1.result + c2.result
            WHEN 'mul' THEN c1.result * c2.result
        END AS result
    FROM nodes n
    JOIN eval_tree c1 ON n.left_id = c1.id
    LEFT JOIN eval_tree c2 ON n.right_id = c2.id
    WHERE n.type != 'lit'
)
SELECT result FROM eval_tree WHERE id = 1;
```

This is a bottom-up evaluation using a recursive common table expression. Literals are evaluated first, then nodes whose children have results, and so on up the tree. It works, but SQL was designed for flat relational data, not recursive tree evaluation. The recursive CTE is powerful but obscures the simple recursive structure of the algorithm.

### MATLAB

MATLAB uses structs and switch statements.

```matlab
function result = eval_expr(expr)
    switch expr.type
        case 'lit'
            result = expr.value;
        case 'neg'
            result = -eval_expr(expr.expr);
        case 'add'
            result = eval_expr(expr.left) + eval_expr(expr.right);
        case 'mul'
            result = eval_expr(expr.left) * eval_expr(expr.right);
    end
end
```

MATLAB structs with a type field and a switch statement — essentially the same approach as JavaScript and Lua. MATLAB's matrix operations are irrelevant here, and the code is straightforward but unexceptional. This is far from MATLAB's sweet spot.

### R

R uses lists as nodes and recursive function application.

```r
eval_expr <- function(expr) {
  switch(expr$type,
    lit = expr$value,
    neg = -eval_expr(expr$expr),
    add = eval_expr(expr$left) + eval_expr(expr$right),
    mul = eval_expr(expr$left) * eval_expr(expr$right)
  )
}
```

R's `switch` on `expr$type` dispatches to each case. R lists serve as structs (`expr$value` accesses the `value` field). This is clean and concise but, like MATLAB, far from R's strength in statistical computing.

### Julia

Julia uses multiple dispatch, which provides an elegant solution.

```julia
abstract type Expr end

struct Lit <: Expr
    value::Float64
end

struct Neg <: Expr
    expr::Expr
end

struct Add <: Expr
    left::Expr
    right::Expr
end

struct Mul <: Expr
    left::Expr
    right::Expr
end

eval_expr(e::Lit) = e.value
eval_expr(e::Neg) = -eval_expr(e.expr)
eval_expr(e::Add) = eval_expr(e.left) + eval_expr(e.right)
eval_expr(e::Mul) = eval_expr(e.left) * eval_expr(e.right)
```

Julia's multiple dispatch selects the function method based on the argument type, similar to pattern matching on constructors. Each `eval_expr` method handles one case. The `abstract type Expr end` creates a type hierarchy, and `<:` declares subtyping. This is arguably the cleanest approach among the non-ML languages — Julia's dispatch system makes tree evaluation natural without requiring explicit match expressions.

---

## Historical Languages

### FORTRAN

FORTRAN was designed for numerical computation, not symbolic tree manipulation.

```fortran
MODULE ExprModule
  IMPLICIT NONE
  
  INTEGER, PARAMETER :: LIT_TAG=1, NEG_TAG=2, ADD_TAG=3, MUL_TAG=4
  
  TYPE :: Expr
    INTEGER :: tag
    REAL :: value
    TYPE(Expr), POINTER :: left => NULL()
    TYPE(Expr), POINTER :: right => NULL()
  END TYPE
  
CONTAINS
  RECURSIVE FUNCTION Eval(e) RESULT(res)
    TYPE(Expr), INTENT(IN) :: e
    REAL :: res
    
    SELECT CASE (e%tag)
      CASE (LIT_TAG)
        res = e%value
      CASE (NEG_TAG)
        res = -Eval(e%left)
      CASE (ADD_TAG)
        res = Eval(e%left) + Eval(e%right)
      CASE (MUL_TAG)
        res = Eval(e%left) * Eval(e%right)
    END SELECT
  END FUNCTION
END MODULE
```

Modern Fortran (90+) supports derived types and pointers, making tree representation possible. The `SELECT CASE` dispatches on the tag. The `RECURSIVE` keyword is required to declare recursive functions. This works but is verbose and alien to Fortran's intended use in scientific computing.

### COBOL

COBOL's data description facilities can define a node structure, but recursive data types require workarounds.

```cobol
IDENTIFICATION DIVISION.
PROGRAM-ID. EVAL-EXPR.

DATA DIVISION.
WORKING-STORAGE SECTION.
01 NODE-TABLE.
   05 NODE-ENTRY OCCURS 100 TIMES.
      10 NODE-TYPE     PIC X(3).
      10 NODE-VALUE    PIC 9(10)V99.
      10 NODE-LEFT     PIC 99.
      10 NODE-RIGHT    PIC 99.
01 WS-RESULT PIC 9(10)V99.

PROCEDURE DIVISION.
EVAL-NODE.
    EVALUATE NODE-TYPE(1)
        WHEN "LIT" MOVE NODE-VALUE(1) TO WS-RESULT
        WHEN "NEG" PERFORM EVAL-NEG
        WHEN "ADD" PERFORM EVAL-ADD
        WHEN "MUL" PERFORM EVAL-MUL
    END-EVALUATE.
    STOP RUN.
```

COBOL uses a flat table of nodes (since it lacks pointers and recursive types) and `EVALUATE` for case dispatch. The tree is stored as a table with index-based references to children. Recursive evaluation requires paragraph-level subroutines. This is an extremely poor fit — COBOL was designed for business record processing, and recursive tree evaluation is about as far from its intended domain as possible.

### BASIC

Classic BASIC lacks user-defined types and recursion entirely.

```basic
10 REM Expression tree in parallel arrays
20 DIM T$(20), V(20), L(20), R(20)
30 REM Node 1: MUL, children 2,3
40 T$(1) = "MUL": L(1) = 2: R(1) = 3
50 T$(2) = "ADD": L(2) = 4: R(2) = 5
60 T$(3) = "LIT": V(3) = 5
70 T$(4) = "LIT": V(4) = 3
80 T$(5) = "LIT": V(5) = 4
90 REM Evaluate bottom-up
100 FOR I = 5 TO 1 STEP -1
110   IF T$(I) = "LIT" THEN V(I) = V(I)
120   IF T$(I) = "NEG" THEN V(I) = -V(L(I))
130   IF T$(I) = "ADD" THEN V(I) = V(L(I)) + V(R(I))
140   IF T$(I) = "MUL" THEN V(I) = V(L(I)) * V(R(I))
150 NEXT I
160 PRINT "Result: "; V(1)
```

Without recursion, BASIC evaluates bottom-up using a loop over a flat node table. This is the most manual encoding in the document — parallel arrays for tags, values, and child indices, processed in reverse order. It works for this specific tree but would need careful ordering for arbitrary trees.

### Algol 60

Algol 60 supports recursion and has some ability to represent structured data, though without modern type systems.

```algol60
real procedure eval(tag, value, left, right, n);
  integer n;
  integer array tag[1:n];
  real array value[1:n];
  integer array left, right[1:n];
begin
  if tag[n] = 0 then eval := value[n]
  else if tag[n] = 1 then eval := -eval(tag, value, left, right, left[n])
  else if tag[n] = 2 then eval := eval(tag, value, left, right, left[n])
                           + eval(tag, value, left, right, right[n])
  else eval := eval(tag, value, left, right, left[n])
               * eval(tag, value, left, right, right[n])
end;
```

Algol 60 uses parallel arrays (like BASIC) but can recurse. The procedure takes the entire tree representation as parameters and an index for which node to evaluate. This is verbose and error-prone but demonstrates that Algol 60 — the ancestor of most modern languages — supported the essential computational pattern.

### Historical Language Comparison

The historical languages reveal how much the ML family's innovations — algebraic data types and pattern matching — transformed the expressibility of programs that work with tree-shaped data. FORTRAN, COBOL, and BASIC were designed before these ideas existed, and their attempts at tree manipulation are labored and fragile. Algol 60 provides recursion but not structured data types. The gap between these languages and Haskell for this particular problem is enormous — far larger than the gap for the "indices above mean" algorithm, where the historical languages were merely verbose rather than fundamentally uncomfortable.

---

## Cross-Family Comparison

### The Expression Problem

This algorithm directly touches the *expression problem*: the tension between adding new data variants and adding new operations. Pattern matching (ML, Rust, Swift) makes it easy to add operations — just write a new function with a case for each variant — but adding a new variant requires modifying every existing function. Polymorphic dispatch (Java, Smalltalk, Go) makes it easy to add variants — just write a new class — but adding a new operation requires modifying every existing class.

This is not a flaw in either approach but a fundamental trade-off in the design of extensible software. Languages that combine both (Scala, Kotlin) use various mechanisms (type classes, extension methods) to mitigate the tension.

### Data Definition

The deepest variation is in how languages define the expression type. ML-family languages have a single, concise declaration that names all variants. Lisp languages have no definition at all — expressions are just lists. OOP languages distribute the definition across class hierarchies. C uses tagged unions. The choice of data definition mechanism shapes everything that follows.

### Dispatch Mechanism

Four fundamentally different dispatch mechanisms appear across language families: pattern matching (ML, Rust, Erlang), conditional chains (Lisp, scripting languages), polymorphic method dispatch (OOP languages), and logical unification (Prolog). Each corresponds to a different programming philosophy. Pattern matching is declarative and centralized. Conditionals are imperative and centralized. Method dispatch is declarative and distributed. Logical unification is relational and bidirectional.

### Recursion vs. Iteration

Most implementations use structural recursion — the evaluator calls itself on subexpressions. A few (BASIC, the APL flat-array approach) use iteration, processing nodes bottom-up. Recursion is more natural for tree-shaped data, which is why recursive data types and recursive functions are such a powerful combination.

### Type Safety Spectrum

The spectrum ranges from full compile-time safety (Haskell, Rust, Lean — which guarantee exhaustiveness and correct field access) through partial safety (TypeScript, Kotlin — which check some things statically) to no static checking (Python, JavaScript, Lua — where all guarantees are at runtime). The proof assistants go further, allowing proofs that the evaluator satisfies a specification.

### Where Each Family Excels

The ML family excels here because this is *their* canonical problem — recursive data with pattern matching. The Lisp family excels because homoiconicity makes the tree a first-class citizen. Prolog excels because relational definitions allow bidirectional reasoning. The OOP languages produce clean, extensible designs through polymorphic dispatch.

The APL family, historical languages, and SQL are uncomfortable — trees are not their native data shape. The C family works but requires significant engineering to manage memory and maintain type safety. The scripting languages converge on a reasonable middle ground with tagged objects and conditionals.

---

## Conclusion

The expression tree evaluator inverts the results of the "indices above mean" algorithm. Where array languages were effortless and ML languages were merely adequate for array transformation, here the ML family is perfectly at home and array languages must improvise. This inversion demonstrates that no single paradigm is universally superior — each is optimized for a class of problems.

The algorithm also reveals a progression in language design. The historical languages lacked the tools to express tree-structured data cleanly. The Wirth family introduced variant records, a partial solution. The ML family perfected algebraic data types and pattern matching, making recursive data manipulation concise and safe. Modern mainstream languages (Java 21, Python 3.10, TypeScript) are now importing these ideas, decades after their invention.

Perhaps the deepest insight is the contrast between data definition and dispatch. ML languages define data centrally and dispatch by shape. Lisp languages skip formal definitions and dispatch by convention. OOP languages distribute definitions and dispatch by method lookup. Prolog defines relations and lets unification handle dispatch. Each choice carries consequences for extensibility, safety, and clarity. The expression tree evaluator, simple as it is, illuminates these fundamental architectural decisions that shape every program we write.
