# 📜 Recap

## Pattern matching

- F♯ core building block
- Combines "data structure matching" and "deconstruction"
- Used almost everywhere:
  - `match` and `function` expressions
  - `try/with` block
  - `let` binding, including function parameter
- Can be abstracted into a `fold` function associated with a union type

## Patterns

| Pattern                            | Example                           |
|------------------------------------|-----------------------------------|
| Constant • Identifier • Wilcard    | `1`, `Color.Red` • `Some 1` • `_` |
| *Collection* : Cons • List • Array | `head :: tail` • `[1; 2]`         |
| *Product type* : Record • Tuple    | `{ A = a }` • `a, b`              |
| Type Test                          | `:? Subtype`                      |
| *Logical* : OR, AND                | `1 \| 2`, `P1 & P2`               |
| Variables • Alias                  | `head :: _` • `(0, 0) as origin`  |

\+ The `when` guards in `match` expressions

## Active Patterns

- Extending pattern matching
- Based on function + metadata → 1st-class citizens
- 4 types: total simple/multiple, partial (simple), parametric
- At 1st little tricky to understand, but we get used to it quickly
- Use for:
  - Add semantics without relying on union types
  - Simplify / factorize guards
  - Wrapping BCL methods
  - Extract a data set from an object
  - ...

## Addendum

📜 [Match expressions](https://fsharpforfunandprofit.com/posts/match-expression/), F# for fun and profit, Juin 2012

📜 [Domain modelling et pattern matching : Roman numeral](https://fsharpforfunandprofit.com/posts/roman-numerals/), F# for fun and profit, Juin 2012

📜 [Recursive types and folds](https://fsharpforfunandprofit.com/series/recursive-types-and-folds/) *(6 articles),* F# for fun and profit, Août 2015

📹 [A Deep Dive into Active Patterns](https://www.youtube.com/watch?v=Q5KO-UDx5eA) - [Repo github](https://github.com/pblasucci/DeepDiveAP)
