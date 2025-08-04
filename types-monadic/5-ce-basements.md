# 🚀 Functional patterns

## CE: the Swiss army knife ✨

The _computation expressions_ serve different purposes:

* C♯ `yield return` → F♯ `seq {}`
* C♯ `async/await` → F♯ `async {}`
* C♯ LINQ expressions `from... select` → F♯ `query {}`
* ...

Underlying theoretical foundations :

* Monoid
* Monad
* Applicative

## Monoid

≃ Type `T` defining a set with:

1. Operation `(+): T -> T -> T`
   * To combine sets and keep the same "type"
   * Associative: `a + (b + c)` ≡ `(a + b) + c`
2. Neutral element _(aka identity)_ ≃ empty set
   * Combinable with any set without effect
   * `a + e` ≡ `e + a` ≡ `a`

### CE monoidal

The builder of a monoidal CE _(such as `seq`)_ has _at least_ :

* `Yield` to build the set element by element
* `Combine` ≡ `(+)` (`Seq.append`)
* Zero `≡ neutral element (`Seq.empty\`)

Generally added (among others):

* `For` to support `for x in xs do ...`
* `YieldFrom` to support `yield!`

## Monad

≃ Generic type `M<'T>` with:

1. `return` construction function
   * Signature : `(value: 'T) -> M<'T>`
   * ≃ Wrap a value
2. Link function `bind` _(aka `>>=` operator)_
   * Signature : `(f: 'T -> M<'U>) -> M<'T> -> M<'U>`
   * Use wrapped value, map with `f` function to a value of another type and re-wrap the result

### Monad laws

`return` ≡ neutral element for `bind`

* Left: `return x |> bind f` ≡ `f x`
* Right: `m |> bind return` ≡ `m`

`bind` is associative

* `m |> bind f |> bind g` ≡ `m |> bind (fun x -> f x |> bind g)`

### Monads and languages

**Haskell**

* Monads used a lot. Common ones: `IO`, `Maybe`, `State`, `Reader`.
* `Monad` is a _type class_ for easily creating your own monads.

**F♯**

* Some CEs allow monadic operations.
* More rarely used directly _(except by Haskellers, OCamlers...)_

**C♯**

* Monad implicit in LINQ
* [LanguageExt](https://github.com/louthy/language-ext) library for functional programming

### Monadic CE

The builder of a monadic CE has `Return` and `Bind` methods.

The `Option` and `Result` types are monadic.\
→ We can create their own CE :

```fsharp
type OptionBuilder() =
    member _.Bind(x, f) = x |> Option.bind f
    member _.Return(x) = Some x

type ResultBuilder() =
    member _.Bind(x, f) = x |> Result.bind f
    member _.Return(x) = Ok x
```

### Monadic and generic CE

[FSharpPlus](http://fsprojects.github.io/FSharpPlus/computation-expressions.html) provides a `monad` CE\
→ Works for all monadic types: `Option`, `Result`, ... and even `Lazy`!

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let lazyValue = monad {
    let! a = lazy (printfn "I'm lazy"; 2)
    let! b = lazy (printfn "I'm lazy too"; 10)
    return a + b
} // System.Lazy<int>

let result = lazyValue.Value
// I'm lazy
// I'm lazy too
// val result : int = 12
```

### Example with `Option` type:

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let addOptions x' y' = monad {
    let! x = x'
    let! y = y'
    return x + y
}

let v1 = addOptions (Some 1) (Some 2) // Some 3
let v2 = addOptions (Some 1) None     // None
```

### Limits

⚠️ Several monadic types cannot be mixed!

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let v1 = monad {
    let! a = Ok 2
    let! b = Some 10
    return a + b
} // 💥 Error FS0043...

let v2 = monad {
    let! a = Ok 2
    let! b = Some 10 |> Option.toResult
    return a + b
} // val v2 : Result<int,unit> = Ok 12
```

### Specific monadic CE

[FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling/) library provides:\
• CE `option {}` specific to type `Option<'T>` _(example below)_\
• CE `result {}` specific to type `Result<'Ok, 'Err>`

☝ Recommended as it is more explicit than `monad` CE.

```fsharp
#r "nuget: FSToolkit.ErrorHandling"
open FsToolkit.ErrorHandling

let addOptions x' y' = option {
    let! x = x'
    let! y = y'
    return x + y
}

let v1 = addOptions (Some 1) (Some 2) // Some 3
let v2 = addOptions (Some 1) None     // None
```

## Applicative _(a.k.a Applicative Functor)_

≃ Generic type `M<'T>` -- 3 styles:

**Style A:** Applicative with `apply`/`<*>` and `pure`/`return`\
• ❌ Not easy to understand\
• ☝ Not recommended by Don Syme in the [Nov. 2020 note](https://github.com/dsyme/fsharp-presentations/blob/master/design-notes/rethinking-applicatives.md)

**Style B:** Applications with `mapN`\
• `map2`, `map3`... `map5` combines 2 to 5 wrapped values

**Style C:** Applicatives with `let! ... and! ...` in a CE\
• Same principle: combine several wrapped values\
• Available from F♯ 5 _(_[_announcement Nov. 2020_](https://devblogs.microsoft.com/dotnet/announcing-f-5/#applicative-computation-expressions)_)_

☝ **Tip:** Styles B and C are equally recommended.

### Applicative CE

Library [FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling/) offers:

* Type `Validation<'Ok, 'Err>` ≡ `Result<'Ok, 'Err list>`
* CE `validation {}` supporting `let!...and!...` syntax.

Allows errors to be accumulated → Uses:

* Parsing external inputs
* _Smart constructor_ _(Example code slide next...)_

**Example:**

```fsharp
#r "nuget: FSToolkit.ErrorHandling"
open FsToolkit.ErrorHandling

type [<Measure>] cm
type Customer = { Name: string; Height: int<cm> }

let validateHeight height =
    if height <= 0<cm>
    then Error "Height must me positive"
    else Ok height

let validateName name =
    if System.String.IsNullOrWhiteSpace name
    then Error "Name can't be empty"
    else Ok name

module Customer =
    let tryCreate name height : Result<Customer, string list> =
        validation {
            let! validName = validateName name
            and! validHeight = validateHeight height
            return { Name = validName; Height = validHeight }
        }

let c1 = Customer.tryCreate "Bob" 180<cm>  // Ok { Name = "Bob"; Height = 180 }
let c2 = Customer.tryCreate "Bob" 0<cm> // Error ["Height must me positive"]
let c3 = Customer.tryCreate "" 0<cm>    // Error ["Name can't be empty"; "Height must me positive"]
```

## Applicative _vs_ Monad

The `Result` type is "monadic": on the 1st error, we "unplug".

There is another type called `Validation` that is "applicative": it allows to accumulate errors.

* ≃ `Result<'ok, 'error list>`\\
* Handy for validating user input and reporting all errors

Example: `Validation.map2` to combine 2 results and get the list of their eventual errors.

```fsharp
module Validation =
    // f : 'T -> 'U -> Result<'V, 'Err list>
    // x': Result<'T, 'Err list>
    // y': Result<'U, 'Err list>
    //  -> Result<'V, 'Err list>
    let map2 f x' y' =
        match x', y' with
        | Ok x, Ok y -> f x y
        | Ok _, Error errors | Error errors, Ok _ -> Error errors
        | Error errors1, Error errors2 -> Error (errors1 @ errors2) // 👈 ②
```

🔗 **Ressources**

* [FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling)
* [Validation with F# 5 and FsToolkit](https://www.compositional-it.com/news-blog/validation-with-f-5-and-fstoolkit/)

## Other CE

We've seen 2 libraries that extend F♯ and offer their CEs:

* FSharpPlus → `monad`
* FsToolkit.ErrorHandling → `option`, `result`, `validation`

Many libraries have their own DSL _(Domain Specific Language.)_\
Some are based on CE:

* [Expecto](https://github.com/haf/expecto): Testing library (`test "..." {...}`)
* [Farmer](https://github.com/compositionalit/farmer): Infra as code for Azure (`storageAccount {...}`)
* [Saturn](https://saturnframework.org/): Web framework on top of ASP.NET Core (`application {...}`)
