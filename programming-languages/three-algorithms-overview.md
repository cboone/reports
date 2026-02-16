---
created: 2026-02-06
---

# Three Algorithms, Dozens of Languages: An Overview

_February 6, 2026_

This document introduces and connects three companion studies, each implementing a single algorithm across dozens of programming languages. Together, they reveal how different language families are optimized for fundamentally different kinds of problems — and why no single paradigm is universally best.

---

## The Three Algorithms

### 1. Finding Indices of Elements Above the Mean

**The task**: Given an array of numbers, return the indices of all elements whose value exceeds the arithmetic mean of the array.

**What it tests**: Array transformation, implicit iteration, aggregate computation, and filtering. This is a data-parallel problem that operates on flat, rectangular data — the natural habitat of array languages.

**Example**: Given `[3, 1, 4, 1, 5, 9, 2, 6]`, the mean is 3.875, and the indices of elements above the mean are `[2, 4, 5, 7]` (zero-based).

### 2. Evaluating an Expression Tree

**The task**: Given a tree representing an arithmetic expression (with literal numbers, negation, addition, and multiplication), evaluate it recursively to produce a numeric result.

**What it tests**: Recursive data type definition, structural dispatch (pattern matching or polymorphism), and tree traversal. This is a symbolic computation problem that operates on recursive, tree-shaped data — the natural habitat of ML-family and Lisp-family languages.

**Example**: `Mul(Add(Lit(3), Lit(4)), Lit(5))` evaluates to `35`.

### 3. A Concurrent Producer-Consumer Pipeline

**The task**: Multiple producers generate items into a shared bounded buffer; multiple consumers pull items, process them, and collect results. The pipeline must handle backpressure, safe concurrent access, and clean shutdown.

**What it tests**: Concurrency primitives, coordination between independent execution units, shared-state safety, and shutdown signaling. This is a coordination problem — the natural habitat of actor-model languages and channel-based concurrency systems.

**Example**: Three producers each emitting `[1..5]`, two consumers squaring each item, yielding 15 squared values collected and sorted.

---

## The Design of the Triptych

These three algorithms were chosen because each one makes a different language family look effortless while making others look labored. The combination ensures that every major paradigm gets its moment in the spotlight — and its moment of awkwardness.

The key insight is that "expressiveness" is not a single axis. A language can be supremely expressive for one class of problems and clumsy for another. APL expresses array transformations in a handful of characters but struggles with trees. Haskell expresses tree evaluation as a direct transliteration of the mathematical definition but requires more ceremony for concurrent coordination. Erlang makes concurrent pipelines trivial but has no special notation for array operations. Each language optimizes for the problems its designers considered most important.

---

## Results by Language Family

### The APL Family (APL, J, K, BQN, Uiua)

**Array transformation**: ★★★★★ — This is what they were born for. APL expresses the complete algorithm in 8 characters. The implicit iteration over arrays, the boolean masking, the index-extraction primitives — everything aligns perfectly. The APL family does not merely solve this problem; it makes the problem *disappear* into notation.

**Expression trees**: ★☆☆☆☆ — Trees are the anti-array. They are recursive, non-rectangular, and have varying numbers of children per node. Representing them in array languages requires flattening into parallel arrays or using boxing mechanisms that abandon array semantics. The evaluator becomes a loop with conditionals — exactly the kind of explicit, scalar code that array languages were designed to eliminate.

**Concurrent pipeline**: ★★☆☆☆ — Array languages approach concurrency through implicit data parallelism (parallel evaluation of array operations) rather than explicit task coordination. Some implementations offer parallel primitives, but the producer-consumer pattern — which requires coordination between independent tasks — is outside the paradigm.

### The ML Family (Haskell, SML, OCaml, F#, Elm, PureScript)

**Array transformation**: ★★★☆☆ — Clean and readable using higher-order functions (`filter`, `zip`, `map`), but more verbose than array languages. The algorithm requires explicit function composition where APL uses implicit array operations. Competent but not inspired.

**Expression trees**: ★★★★★ — This is the ML family's canonical example. Algebraic data types define the tree in four lines, and pattern matching defines the evaluator in four more. The code reads like the mathematical definition. The compiler verifies exhaustiveness. This is the problem the ML family was designed to solve, and it shows.

**Concurrent pipeline**: ★★★☆☆ — Varies by language. Haskell offers STM (Software Transactional Memory), a unique and powerful composable concurrency primitive. OCaml 5 has effect-based concurrency. F# uses .NET's async workflows. All are workable but none makes concurrency feel as natural as Erlang or Go.

### The Lisp Family (Common Lisp, Scheme, Racket, Clojure)

**Array transformation**: ★★★☆☆ — Similar to the ML family: higher-order functions over sequences. Clojure's threading macros add elegance. The Lisp family does not have array-specific notation, so the solution is functional but not terse.

**Expression trees**: ★★★★☆ — Homoiconicity means the tree is just a nested list — the same data structure that Lisp programs are. There is no type definition needed. Dispatch is manual (via `cond` or `case`), but Racket's `match` approaches ML-style pattern matching. The deeper insight is that in Lisp, evaluating an expression tree is essentially a subset of what the built-in `eval` already does, since code is data.

**Concurrent pipeline**: ★★★☆☆ — Clojure's `core.async` provides Go-style channels in a Lisp. Other Lisps have varying thread support. Clojure's immutable data structures provide natural thread safety for shared data. The Lisp family is adequate but not distinguished here.

### The BEAM Family (Erlang, Elixir, Gleam)

**Array transformation**: ★★★☆☆ — Standard functional programming with list comprehensions and higher-order functions. Competent, not remarkable. Erlang's syntax is functional but not optimized for data transformation pipelines.

**Expression trees**: ★★★★☆ — Pattern matching on tagged tuples is clean and expressive. Erlang and Elixir handle this well, and Gleam adds ML-style algebraic data types with exhaustiveness checking. Not quite as perfect a fit as the ML family, but close.

**Concurrent pipeline**: ★★★★★ — This is what the BEAM was built for. Lightweight processes (millions per VM), message passing, no shared state, built-in fault tolerance through supervision trees. The producer-consumer pipeline is a toy version of problems Erlang solves in production at WhatsApp-scale. Every component — producers, consumers, buffer, coordinator — is naturally a separate process communicating via messages.

### The JVM Family (Java, Scala, Kotlin)

**Array transformation**: ★★★☆☆ — Java's streams, Scala's collections, and Kotlin's extension functions all provide clean functional APIs. Verbose by APL standards but readable and well-optimized.

**Expression trees**: ★★★★☆ — Scala has had algebraic data types since its inception. Java 21 adds sealed interfaces, records, and pattern matching. Kotlin uses sealed classes and smart casts. All three produce clean tree evaluators, reflecting the gradual adoption of ML-family ideas by mainstream languages.

**Concurrent pipeline**: ★★★★☆ — The JVM has decades of investment in concurrency: `java.util.concurrent`, Akka/Pekko actors, Kotlin coroutines with channels, Clojure's `core.async`. Java 21's virtual threads make lightweight concurrency available. Kotlin's approach is particularly clean, resembling Go's goroutine-channel model.

### Modern Systems Languages (Rust, Go, Zig)

**Array transformation**: ★★★☆☆ — All three are explicit, iterative languages. Go is the most verbose (explicit loops, no generics for a long time). Rust's iterator chains are elegant. Zig is minimal and manual. None approaches APL's brevity.

**Expression trees**: ★★★★☆ for Rust, ★★☆☆☆ for Go, ★★★☆☆ for Zig — Rust's `enum` and `match` are proper algebraic data types, nearly as clean as Haskell (with `Box` for indirection as the only systems-level artifact). Go has no sum types and uses interfaces, losing the closed-world guarantee. Zig's tagged unions work but are more verbose.

**Concurrent pipeline**: ★★★★★ for Go, ★★★★☆ for Rust, ★★★☆☆ for Zig — Go's goroutines and channels were designed for exactly this pattern. The bounded channel handles backpressure, close handles shutdown, and WaitGroups coordinate completion. Rust provides stronger safety (data races are compile errors) but requires more ceremony. Zig requires building concurrent primitives from mutexes and condition variables.

### The C Family (C, C++, Objective-C)

**Array transformation**: ★★☆☆☆ — Manual loops, manual memory allocation for the result, manual bounds management. The algorithm is clear but buried under bookkeeping. C requires two passes (count results, allocate, fill).

**Expression trees**: ★★☆☆☆ — Tagged unions in C, `std::variant` templates in C++, class hierarchies in Objective-C. All work but require significant engineering to manage memory and maintain type safety. The essential four-case recursion is buried under structural concerns.

**Concurrent pipeline**: ★★☆☆☆ — Mutexes, condition variables, thread creation, manual signal/broadcast — every synchronization decision is explicit. The C version is the longest in the study, roughly 100 lines for what Go does in 30. Correct but error-prone.

### The Wirth Family (Pascal, Modula-2, Ada)

**Array transformation**: ★★☆☆☆ — Explicit loops with manual result collection. Verbose but clear, reflecting Wirth's pedagogical emphasis.

**Expression trees**: ★★★☆☆ — Variant records (Pascal, Modula-2) and discriminated records (Ada) are the historical precursors of algebraic data types. They work but lack compile-time exhaustiveness checking. Ada's runtime discriminant checks add some safety.

**Concurrent pipeline**: ★★★★☆ for Ada, ★★☆☆☆ for others — Ada's built-in tasking and protected objects with entry guards are the earliest form of language-level structured concurrency. Protected objects are monitors with condition-based entry, handling synchronization with minimal boilerplate. Pascal and Modula-2 lack concurrency primitives.

### Scripting Languages (Python, Ruby, JavaScript, TypeScript, Perl, Lua)

**Array transformation**: ★★★★☆ — List comprehensions (Python), `select` with index (Ruby), `map`/`filter` (JavaScript). Readable, concise, and idiomatic. Python's version is particularly clean.

**Expression trees**: ★★★☆☆ — Dataclasses or Structs with switch/match dispatch. Python 3.10's `match` and TypeScript's discriminated unions approach ML-style pattern matching. Others use tagged-object patterns with conditionals. Readable but without static safety.

**Concurrent pipeline**: ★★☆☆☆ — Fundamentally limited by runtime architectures: Python's GIL, Ruby's GVL, JavaScript's single-threaded event loop, and Lua's cooperative coroutines all prevent true CPU parallelism. The pattern can be simulated with queues and threads, but the languages cannot deliver actual concurrent execution without external mechanisms.

### Proof Assistants (Lean, Agda, Idris, Coq)

**Array transformation**: ★★★☆☆ — Similar to the ML family, with additional proof capabilities that are not exercised by this problem.

**Expression trees**: ★★★★★ — Not only can they define and evaluate the tree as cleanly as Haskell, they can *prove* the evaluator correct: that negating twice is the identity, that evaluation commutes with certain transformations, that an optimizer preserves semantics. This is the algorithm that best demonstrates what dependent types add beyond the ML family.

**Concurrent pipeline**: ★☆☆☆☆ — Proof assistants focus on verified sequential computation. Concurrency verification is an active research area but not practical for everyday programming.

### Logic Languages (Prolog)

**Array transformation**: ★★★☆☆ — Prolog can express the algorithm using `findall` to collect indices satisfying a predicate. Declarative but not as natural as array or functional languages.

**Expression trees**: ★★★★☆ — Prolog's compound terms are tree-shaped, making expression representation natural. The evaluator becomes a set of logical relations. Most remarkably, the relation can sometimes be run in reverse — asking "what expression evaluates to 35?" — which no other paradigm supports.

**Concurrent pipeline**: ★★☆☆☆ — Some Prolog implementations (SWI-Prolog) provide threads with message queues, but concurrency is peripheral to the logic programming paradigm.

---

## Cross-Cutting Themes

### Data Shape Determines Paradigm Fit

The single most important factor in how well a language handles each algorithm is whether the language's native data shape matches the problem's data shape. Array languages operate on flat, rectangular data — perfect for the first algorithm, terrible for trees. ML languages operate on recursive algebraic structures — perfect for trees, adequate for arrays. Actor languages operate on independent, communicating processes — perfect for concurrent pipelines, adequate for everything else.

This is not a flaw in any language but a fundamental property of paradigm design. A paradigm that is good at everything is good at nothing in particular. The power of APL's notation comes precisely from its commitment to arrays; generalize it to trees and you lose what made it special.

### The Abstraction Spectrum

Each algorithm reveals a different slice of the abstraction spectrum. For array transformation, the spectrum runs from APL's implicit iteration to C's explicit loops. For tree evaluation, it runs from Haskell's mathematical definition to C's tagged unions with manual memory. For concurrent pipelines, it runs from Erlang's message passing to C's mutexes and condition variables.

In every case, higher abstraction means less code, fewer bugs, and less control over execution details. Lower abstraction means more code, more potential for bugs, and more control. The right level of abstraction depends on the context: a financial trading system may need C's control; a web application may need Erlang's safety; a compiler may need Haskell's type guarantees.

### Language Evolution as Paradigm Borrowing

A recurring theme across all three studies is that mainstream languages are borrowing from specialized paradigms. Python 3.10 adds ML-style pattern matching. Java 21 adds sealed types, records, and virtual threads. TypeScript adds discriminated unions. Kotlin adds coroutines with channels. Each adoption makes the mainstream language better at the problems that the specialized paradigm excels at, while retaining the mainstream language's existing strengths.

This convergence does not make the specialized languages obsolete. Haskell's type system remains far more powerful than Java's sealed interfaces. APL's array notation remains far more concise than NumPy. Erlang's fault-tolerance model remains far more mature than Kotlin's coroutines. But the gap narrows with each generation, and polyglot ideas spread further.

### What Each Algorithm Fails to Test

Each algorithm has blind spots. The array transformation does not test recursion, type definition, or concurrency. The expression tree evaluator does not test I/O, error handling, or performance optimization. The concurrent pipeline does not test data transformation or static type reasoning.

Important language properties that none of the three algorithms adequately tests include: metaprogramming and macro systems (where Lisp excels), type-level programming (where Haskell and Idris excel), interop with existing ecosystems (where JVM and .NET languages excel), startup time and deployment simplicity (where Go excels), and embeddability (where Lua excels). A complete assessment of any language would require many more algorithms chosen to probe these additional dimensions.

---

## Reading the Companion Documents

The three companion documents can be read in any order, but reading them as a sequence creates a narrative arc.

**Start with ["Finding Indices of Elements Above the Mean"](indices-above-mean.md)** if you want to see every language on relatively equal footing. The algorithm is simple enough that no language is truly awkward (except perhaps COBOL), and the differences are primarily in conciseness, style, and the degree of explicit iteration. This document establishes the baseline: what does each language look like when doing straightforward data processing?

**Continue with ["Evaluating Expression Trees"](expression-tree-evaluation.md)** to see the first major divergence. The ML family suddenly looks inspired where it previously looked merely competent. The APL family suddenly looks uncomfortable where it previously looked effortless. The Lisp family reveals its deep insight about code-as-data. This document introduces the crucial idea that language strengths are problem-dependent.

**Finish with ["Concurrent Producer-Consumer Pipeline"](concurrent-producer-consumer.md)** to see the second major divergence. The BEAM family and Go, which were unremarkable for arrays and trees, suddenly excel. The scripting languages, which were adequate for everything else, reveal fundamental architectural limitations. This document completes the picture: different problems do not just favor different syntaxes; they favor different runtime models and concurrency philosophies.

Together, the three documents demonstrate that programming language design involves genuine trade-offs, not a simple ranking from worse to better. Every design decision — implicit vs. explicit iteration, algebraic data types vs. class hierarchies, message passing vs. shared memory, garbage collection vs. ownership — optimizes for some problems at the expense of others. Understanding these trade-offs is what allows programmers to choose the right tool for each job and, more importantly, to think about problems from multiple paradigmatic perspectives.

---

## Summary Table

| Family | Arrays | Trees | Concurrency | Sweet Spot |
|---|---|---|---|---|
| APL | ★★★★★ | ★☆☆☆☆ | ★★☆☆☆ | Data transformation |
| ML | ★★★☆☆ | ★★★★★ | ★★★☆☆ | Symbolic computation |
| Lisp | ★★★☆☆ | ★★★★☆ | ★★★☆☆ | Metaprogramming |
| BEAM | ★★★☆☆ | ★★★★☆ | ★★★★★ | Concurrent systems |
| JVM | ★★★☆☆ | ★★★★☆ | ★★★★☆ | Enterprise applications |
| Systems (Rust) | ★★★☆☆ | ★★★★☆ | ★★★★☆ | Performance-critical safety |
| Systems (Go) | ★★★☆☆ | ★★☆☆☆ | ★★★★★ | Concurrent services |
| C Family | ★★☆☆☆ | ★★☆☆☆ | ★★☆☆☆ | Hardware-level control |
| Scripting | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ | Rapid development |
| Proof Assistants | ★★★☆☆ | ★★★★★ | ★☆☆☆☆ | Verified correctness |
| Logic (Prolog) | ★★★☆☆ | ★★★★☆ | ★★☆☆☆ | Relational reasoning |
| Wirth | ★★☆☆☆ | ★★★☆☆ | ★★★★☆ (Ada) | Teaching & safety-critical |

The empty cells and middle ratings are as informative as the extremes. A ★★★☆☆ means "competent but unremarkable" — the language can do it, but the code does not illuminate anything distinctive about the language's design philosophy. The extreme ratings — both high and low — are where the interesting lessons lie. A ★★★★★ means "this is what the language was designed for." A ★☆☆☆☆ means "this problem is fundamentally alien to the language's worldview." Both extremes teach us something important about how paradigms shape thinking.
