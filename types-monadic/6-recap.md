# 📜 Recap

## Union types: `Option` and `Result`

- What they are used for:
  - Model absence of value and business errors
  - Partial operations made total `tryXxx`
    - *Smart constructor* `tryCreate`
- How to use them:
  - Chaining: `map`, `bind`, `filter` → *ROP*
  - Pattern matching
- Their benefits:
  - `null` free, `Exception` free → no guard clauses Cluttering the code
  - Makes business logic and *happy path* more readable

## *Computation expression (CE)*

- Syntactic sugar: inner syntax standard or "banged" (`let!`)
- *Separation of Concerns*: business logic *vs* "machinery"
- Compiler translates to *builder* calls
  - Object storing a state
  - Builds an output value of a specific type
- Can be nested but not easy to combine!
- Underlying theoretical concepts
  - Monoid → `seq` *(of composable elements and with a "zero "*)
  - Monad → `async`, `option`, `result`
  - Applicative → `validation`/`Result<'T, 'Err list>`
- Libraries: FSharpPlus, FsToolkit, Expecto, Farmer, Saturn, Bolero...

## 🔗 Additional ressources

- Compositional IT *(Isaac Abraham)*
  - [*Writing more succinct C# – in F#! (Part 2)*](https://kutt.it/gpIgfD) • 2020
- F# for Fun and Profit *(Scott Wlaschin)*
  - [*The Option type*](https://kutt.it/e78rNj) • 2012
  - [*Making illegal states unrepresentable*](https://kutt.it/7J5Krc) • 2013
  - [*The "Map and Bind and Apply, Oh my!" series*](https://kutt.it/ebfGNA) • 2015
  - [*The "Computation Expressions" series*](https://kutt.it/drchkQ) • 2013
- Extending F# through Computation Expressions
  - 📹 [Video](https://youtu.be/bYor0oBgvws)
  - 📜 [Article](https://panesofglass.github.io/computation-expressions/#/)
- [Computation Expressions Workshop](https://github.com/panesofglass/computation-expressions-workshop)
- [Applicatives IRL](https://thinkbeforecoding.com/post/2020/10/03/applicatives-irl) by Jeremie Chassaing
