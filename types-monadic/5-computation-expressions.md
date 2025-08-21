# 🚀 Computation expression (CE)

## Intro

### Presentation

🔗 [Learn F# - Computation Expressions](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions), by Microsoft:

> 1. Computation expressions in F# provide a convenient **syntax** for writing computations that can be sequenced and combined using control flow constructs and bindings.
> 2. Depending on the kind of computation expression, they can be thought of as a way to express monads, monoids, monad transformers, and applicatives.

{% hint style="info" %}
### Note

> monads, monoids, monad transformers, and applicatives

These are [4-functional-pattern.md](4-functional-pattern.md "mention"), _except monad transformers_ 📍
{% endhint %}

Built-in CEs: `async` and `task`, `seq`, `query`\
→ Easy to use, once we know the syntax and its keywords

We can write our own CE too.\
→ More challenging!

### Syntax

CE = block like `myCE { body }` where `body` looks like **imperative** F# code with:

* regular keywords: `let`, `do`, `if`/`then`/`else`, `match`, `for`...
* dedicated keywords: `yield`, `return`
* "banged" keywords: `let!`, `do!`, `match!`, `yield!`, `return!`

These keywords hide a ❝ **machinery** ❞ to perform background **specific** effects:

* Asynchronous computations like with `async` and `task`
* State management: e.g. a sequence with `seq`
* Absence of value with `option` CE
* ...

## Builder

A _computation expression_ relies on an object called _Builder_.

{% hint style="warning" %}
#### Warning

This is not exactly the _Builder_ OO design pattern.
{% endhint %}

For each supported **keyword** (`let!`, `return`...), the _Builder_ implements one or more related **methods**.

☝️ Compiler accepts **flexibility** in the builder **method signature**, as long as the methods can be **chained together** properly when the compiler evaluates the CE on the **caller side.**\
→ ✅ Versatile, ⚠️ Difficult to design and to test\
→ Given method signatures illustrate only typical situations.

### Builder example: `logger {}`

Need: log the intermediate values of a calculation

```fsharp
// First version
let log value = printfn $"{value}"

let loggedCalc =
    let x = 42
    log x  // ❶
    let y = 43
    log y  // ❶
    let z = x + y
    log z  // ❶
    z
```

⚠️ **Issues**

1. Verbose: the `log x` interfere with reading
2. _Error prone_: forget a `log`, log wrong value...

💡 **Solutions**

Make logs implicit in a CE by implementing a custom `let!` / `Bind()` :

```fsharp
type LoggerBuilder() =
    let log value = printfn $"{value}"; value
    member _.Bind(x, f) = x |> log |> f
    member _.Return(x) = x

let logger = LoggerBuilder()

//---

let loggedCalc = logger {
    let! x = 42     // 👈 Implicitly perform `log x`
    let! y = 43     // 👈                    `log y`
    let! z = x + y  // 👈                    `log z`
    return z
}
```

The 3 consecutive `let!` are desugared into 3 **nested** calls to `Bind` with:

* 1st argument: the right side of the `let!` (e.g. `42` with `let! x = 42`)
* 2nd argument: a lambda taking the variable defined at the left side of the `let!` (e.g. `x`) and returning the whole expression below the `let!` until the `}`

```fsharp
// let! x = 42
logger.Bind(42, (fun x ->
    // let! y = 43
    logger.Bind(43, (fun y ->
        // let! z = x + y
        logger.Bind(x + y, (fun z ->
            logger.Return z)
        ))
    ))
)
```

### `Bind` _vs_ `let!`

`logger { let! var = expr in cexpr }` is desugared as:\
`logger.Bind(expr, fun var -> cexpr)`

👉 **Key points:**

* `var` and `expr` appear in reverse order
* `var` is used in the rest of the computation `cexpr`\
  → highlighted using the `in` keyword of the verbose syntax
* the lambda `fun var -> cexpr` is a **continuation** function

### CE desugaring: tips 💡

I found a simple way to desugar a computation expression:\
→ Write a failing unit test and use [Unquote](https://github.com/SwensenSoftware/unquote) - 🔗 [Example](https://github.com/rdeneau/formation-fsharp/blob/main/src/FSharpTraining/04-Monad/LoggerTests.fs#L42)

<figure><img src="../.gitbook/assets/desugar-ce-with-unquote.png" alt=""><figcaption></figcaption></figure>

### Constructor parameters

The builder can be constructed with additional parameters.\
→ The CE syntax allows us to pass these arguments when using the CE:

```fsharp
type LoggerBuilder(prefix: string) =
    let log value = printfn $"{prefix}{value}"; value
    member _.Bind(x, f) = x |> log |> f
    member _.Return(x) = x

let logger prefix = LoggerBuilder(prefix)

//---

let loggedCalc = logger "[Debug] " {
    let! x = 42     // 👈 Output "[Debug] 42"
    let! y = 43     // 👈 Output "[Debug] 43"
    let! z = x + y  // 👈 Output "[Debug] 85"
    return z
}
```

### Builder example: `option {}`

Need: successively try to find in maps by identifiers\
→ Steps:

1. `roomRateId` in `policyCodesByRoomRate` map → find `policyCode`
2. `policyCode` in `policyTypesByCode` map → find `policyType`
3. `policyCode` and `policyType` → build `result`

#### Implementation #1: based on match expressions

{% code lineNumbers="true" %}
```fsharp
match policyCodesByRoomRate.TryFind(roomRateId) with
| None -> None
| Some policyCode ->
    match policyTypesByCode.TryFind(policyCode) with  // ⚠️ Nesting
    | None -> None                                    // ⚠️ Duplicates line 2
    | Some policyType -> Some(buildResult policyCode policyType)
```
{% endcode %}

#### Implementation #2: based on `Option` module helpers

```fsharp
policyCodesByRoomRate.TryFind(roomRateId)
|> Option.bind (fun policyCode ->
    policyTypesByCode.TryFind(policyCode)
    |> Option.map (fun policyType -> buildResult policyCode policyType)
)
```

👉 Issues ⚠️:

* Nesting too
* Even more difficult to read because of parentheses

#### Implementation #3: based on the `option {}` CE

```fsharp
type OptionBuilder() =
    member _.Bind(x, f) = x |> Option.bind f
    member _.Return(x) = Some x

let option = OptionBuilder()

option {
    let! policyCode = policyCodesByRoomRate.TryFind(roomRateId)
    let! policyType = policyTypesByCode.TryFind(policyCode)
    return buildResult policyCode policyType
}
```

👉 Both terse and readable ✅🎉

## CE monoidal

A monoidal CE can be identified by the usage of `yield` and `yield!` keywords.

**Relationship with the monoid:**\
→ Hidden in the builder methods:

* `+` operation → `Combine` method
* `e` neutral element → `Zero` method

### CE monoidal builder method signatures

Like we did for functional patterns, we use the generic type notation: \\

* `M<T>`: type returned by the CE
* `Delayed<T>`: presented later 📍

```fsharp
// Method     | Signature                           | CE syntax supported
    Yield     : T -> M<T>                           ; yield x
    YieldFrom : M<T> -> M<T>                        ; yield! xs
    Zero      : unit -> M<T>                        ; if // without `else`  // Monoid neutral element
    Combine   : M<T> * Delayed<T> -> M<T>                                   // Monoid + operation
    Delay     : (unit -> M<T>) -> Delayed<T>        ; // always required with Combine

// Other additional methods
    Run       : Delayed<T> -> M<T>
    For       : seq<T> * (T -> M<U>) -> M<U>        ; for i in seq do yield ... ; for i = 0 to n do yield ...
                              (* or *)  seq<M<U>>
    While     : (unit -> bool) * Delayed<T> -> M<T> ; while cond do yield...
    TryWith   : M<T> -> (exn -> M<T>) -> M<T>       ; try/with
    TryFinally: Delayed<T> * (unit -> unit) -> M<T> ; try/finally
```

### CE monoidal _vs_ comprehension

#### Comprehension definition

> It is the concise and declarative syntax to build collections with control flow keywords `if`, `for`, `while`... and ranges `start..end`.

#### Comparison

* Similar syntax from caller perspective
* Distinct overlapping concepts

#### Minimal set of methods expected for each

* Monoidal CE: `Yield`, `Combine`, `Zero`
* Comprehension: `For`, `Yield`

### CE monoidal example: `multiplication {}`

Let's build a CE that multiplies the integers yielded in the computation body:\
→ CE type: `M<T> = int` • Monoid operation = `*` • Neutral element = `1`

```fsharp
type MultiplicationBuilder() =
    member _.Zero() = 1
    member _.Yield(x) = x
    member _.Combine(x, y) = x * y
    member _.Delay(f) = f () // eager evaluation

    member m.For(xs, f) =
        (m.Zero(), xs)
        ||> Seq.fold (fun res x -> m.Combine(res, f x))

let multiplication = MultiplicationBuilder()

let shouldBe10 = multiplication { yield 5; yield 2 }
let factorialOf5 = multiplication { for i in 2..5 -> i } // 2 * 3 * 4 * 5
```

{% hint style="success" %}
### Exercise

* Copy this snippet in vs code
* Comment out builder methods
  * To start with an empty builder, add this line `let _ = ()` in the body.
  * After adding the first method, this line can be removed.
* Let the compiler errors in `shouldBe10` and `factorialOf5` guides you to ad the relevant methods.
{% endhint %}

Desugared `multiplication { yield 5; yield 2 }`:

```fsharp
// Original
let shouldBe10 =
    multiplication.Delay(fun () ->
        multiplication.Combine(
            multiplication.Yield(5),
            multiplication.Delay(fun () ->
                multiplication.Yield(2)
            )
        )
    )

// Simplified (without Delay)
let shouldBe10 =
    multiplication.Combine(
        multiplication.Yield(5),
        multiplication.Yield(2)
    )
```

Desugared `multiplication { for i in 2..5 -> i }`:

```fsharp
// Original
let factorialOf5 =
    multiplication.Delay (fun () ->
        multiplication.For({2..5}, (fun _arg2 ->
            let i = _arg2 in multiplication.Yield(i))
        )
    )

// Simplified
let factorialOf5 =
    multiplication.For({2..5}, (fun i -> multiplication.Yield(i)))
```

### CE monoidal `Delayed<T>` type

`Delayed<T>` represents a delayed computation and is used in these methods:

* `Delay` returns this type, hence defines it for the CE
* `Combine`, `Run`, `While` and `TryFinally` used it as input parameter

```fsharp
 Delay      : thunk: (unit -> M<T>) -> Delayed<T>
 Combine    : M<T> * Delayed<T> -> M<T>
 Run        : Delayed<T> -> M<T>
 While      : predicate: (unit -> bool) * Delayed<T> -> M<T>
 TryFinally : Delayed<T> * finalizer: (unit -> unit) -> M<T>
```

* `Delay` is called each time converting from `M<T>` to `Delayed<T>` is needed
* `Delayed<T>` is internal to the CE
  * `Run` is required at the end to get back the `M<T>`...
  * ... **only** when `Delayed<T>` ≠ `M<T>`, otherwise it can be omitted

👉 Enables to implement **laziness and short-circuiting** at the CE level.

**Example:** lazy `multiplication {}` with `Combine` optimized when `x = 0`

```fsharp
type MultiplicationBuilder() =
    member _.Zero() = 1
    member _.Yield(x) = x
    member _.Delay(thunk: unit -> int) = thunk // Lazy evaluation
    member _.Run(delayedX: unit -> int) = delayedX () // Required to get a final `int`

    member _.Combine(x: int, delayedY: unit -> int) : int =
        match x with
        | 0 -> 0 // 👈 Short-circuit for multiplication by zero
        | _ -> x * delayedY ()

    member m.For(xs, f) =
        (m.Zero(), xs) ||> Seq.fold (fun res x -> m.Combine(res, m.Delay(fun () -> f x)))
```

| Difference              | Eager   | Lazy                           |
| ----------------------- | ------- | ------------------------------ |
| `Delay` return type     | `int`   | `unit -> int`                  |
| `Run`                   | Omitted | Required to get back an `int`  |
| `Combine` 2nd parameter | `int`   | `unit -> int`                  |
| `For` calling `Delay`   | Omitted | Explicit but not required here |

<figure><img src="../.gitbook/assets/diff-ce-multiplication-eager-vs-lazy.png" alt=""><figcaption></figcaption></figure>

### CE monoidal kinds

With `multiplication {}`, we've seen a first kind of monoidal CE:\
→ To reduce multiple yielded values into 1.

Second kind of monoidal CE:\
→ To aggregate multiple yielded values into a collection.\
→ Example: `seq {}` returns a `'t seq`.

### CE monoidal to generate a collection

Let's build a `list {}` monoidal CE!

```fsharp
type ListBuilder() =
    member _.Zero() = [] // List.empty
    member _.Yield(x) = [x] // List.singleton
    member _.YieldFrom(xs) = xs
    member _.Delay(thunk: unit -> 't list) = thunk () // eager evaluation
    member _.Combine(xs, ys) = xs @ ys // List.append
    member _.For(xs, f: _ seq) = xs |> Seq.collect f |> Seq.toList

let list = ListBuilder()
```

{% hint style="info" %}
### Notes 💡

* `M<T>` is `'t list` → type returned by `Yield` and `Zero`
* `For` uses an intermediary sequence to collect the values returned by `f`.
{% endhint %}

Let's test the CE to generate the list `[begin; 16; 9; 4; 1; 2; 4; 6; 8; end]`\
\&#xNAN;_(Desugared code simplified)_

<figure><img src="../.gitbook/assets/list-ce-desugared.png" alt=""><figcaption></figcaption></figure>

Comparison with the same expression in a list comprehension:

<figure><img src="../.gitbook/assets/list-real-desugared.png" alt=""><figcaption></figcaption></figure>

`list { expr }` _vs_ `[ expr ]`:

* `[ expr ]` uses a hidden `seq` all through the computation and ends with a `toList`
* All methods are inlined:

| Method    | `list { expr }`              | `[ expr ]`      |
| --------- | ---------------------------- | --------------- |
| `Combine` | `xs @ ys => List.append`     | `Seq.append`    |
| `Yield`   | `[x] => List.singleton`      | `Seq.singleton` |
| `Zero`    | `[] => List.empty`           | `Seq.empty`     |
| `For`     | `Seq.collect` & `Seq.toList` | `Seq.collect`   |

## CE monadic

A monadic CE can be identified by the usage of `let!` and `return` keywords, revealing the monadic `bind` and `return` operations.

Behind the scene, builders of these CE should/can implement these methods:

```fsharp
// Method     | Signature                                     | CE syntax supported
    Bind      : M<T> * (T -> M<U>) -> M<U>                    ; let! x = xs in ...
                (* when T = unit *)                           ; do! command
    Return    : T -> M<T>                                     ; return x
    ReturnFrom: M<T> -> M<T>                                  ; return!

// Additional methods
    Zero      : unit -> M<T>                                  ; if // without `else` // Typically `unit -> M<unit>`
    Combine   : M<unit> * M<T> -> M<T>                        ; e1; e2  // e.g. one loop followed by another one
    TryWith   : M<T> -> (exn -> M<T>) -> M<T>                 ; try/with
    TryFinally: M<T> * (unit -> M<unit>) -> M<T>              ; try/finally
    While     : (unit -> bool) * (unit -> M<unit>) -> M<unit> ; while cond do command ()
    For       : seq<T> * (T -> M<unit>) -> M<unit>            ; for i in xs do command i ; for i = 0 to n do command i
    Using     : T * (T -> M<U>) -> M<U> when T :> IDisposable ; use! x = xs in ...
```

### CE monadic _vs_ CE monoidal

#### `Return` (monad) _vs_ `Yield` (monoid)

* Same signature: `T -> M<T>`
* A series of `return` is not expected\
  → Monadic `Combine` takes only a monadic command `M<unit>` as 1st param
* CE enforces appropriate syntax by implementing 1! of these methods:
  * `seq {}` allows `yield` but not `return`
  * `async {}`: vice versa

#### `For` and `While`

<table><thead><tr><th width="99">Method</th><th width="147">CE</th><th>Signature</th></tr></thead><tbody><tr><td><code>For</code></td><td>Monoidal</td><td><code>seq&#x3C;T> * (T -> M&#x3C;U>) -> M&#x3C;U> or seq&#x3C;M&#x3C;U>></code></td></tr><tr><td></td><td>Monadic</td><td><code>seq&#x3C;T> * (T -> M&#x3C;unit>) -> M&#x3C;unit></code></td></tr><tr><td><code>While</code></td><td>Monoidal</td><td><code>(unit -> bool) * Delayed&#x3C;T> -> M&#x3C;T></code></td></tr><tr><td></td><td>Monadic</td><td><code>(unit -> bool) * (unit -> M&#x3C;unit>) -> M&#x3C;unit></code></td></tr></tbody></table>

👉 Different use cases:

* Monoidal: Comprehension syntax
* Monadic: Series of effectful commands

### CE monadic and delayed

Like monoidal CE, monadic CE can use a `Delayed<'t>` type.\
→ Impacts on the method signatures:

```fsharp
 Delay      : thunk: (unit -> M<T>) -> Delayed<T>
 Run        : Delayed<T> -> M<T>
 Combine    : M<unit> * Delayed<T> -> M<T>
 While      : predicate: (unit -> bool) * Delayed<unit> -> M<unit>
 TryFinally : Delayed<T> * finalizer: (unit -> unit) -> M<T>
 TryWith    : Delayed<T> * handler: (exn -> unit) -> M<T>
```

### CE monadic examples

☝️ The initial CE studied—`logger {}` and `option {}`—was monadic.

Let's play with a `result {}` CE!

```fsharp
type ResultBuilder() =
    member _.Bind(rx, f) = rx |> Result.bind f
    member _.Return(x) = Ok x
    member _.ReturnFrom(rx) = rx

let result = ResultBuilder()

// ---

let rollDice =
    let random = Random(Guid.NewGuid().GetHashCode())
    fun () -> random.Next(1, 7)

let tryGetDice dice =
    result {
        if rollDice() <> dice then
            return! Error $"Not the expected dice {dice}."
    }

let tryGetAPairOf6 =
    result {
        let n = 6
        do! tryGetDice n
        do! tryGetDice n
        return true
    }
```

Desugaring:

```fsharp
let tryGetAPairOf6 =
    result {                ;
        let n = 6           ;   let n = 6
        do! tryGetDice n    ;   result.Bind(tryGetDice n, (fun () ->
        do! tryGetDice n    ;        result.Bind(tryGetDice n, (fun () ->
        return true         ;            result.Return(true)
    }                       ;        ))
                            ;   ))
```

### CE monadic: FSharpPlus `monad` CE

[FSharpPlus](http://fsprojects.github.io/FSharpPlus/computation-expressions.html) provides a `monad` CE

* Works for all monadic types: `Option`, `Result`, ... and even `Lazy` 🎉
* Supports monad stacks with monad transformers 📍

⚠️ **Limits:**

* Confusing: the `monad` CE has 4 flavours to cover all cases: delayed or strict, embedded side-effects or not
* Based on SRTP: can be very long to compile!
* Documentation not exhaustive, relying on Haskell knowledges
* Very Haskell-oriented: not idiomatic F♯

### Monad stack, monad transformers

A monad stack is a composition of different monads.\
→ Example: `Async`+`Option`.

How to handle it?\
→ Academic style _vs_ idiomatic F♯

#### 1. Academic style (with FSharpPlus)

Monad transformer (here `MaybeT`)\
→ Extends `Async` to handle both effects\
→ Resulting type: `MaybeT<Async<'t>>`

✅ reusable with other inner monad ❌ less easy to evaluate the resulting value ❌ not idiomatic

#### 2. Idiomatic style

Custom CE `asyncOption`, based on the `async` CE, handling `Async<Option<'t>>` type

```fsharp
type AsyncOption<'T> = Async<Option<'T>> // Convenient alias, not required

type AsyncOptionBuilder() =
    member _.Bind(aoX: AsyncOption<'a>, f: 'a -> AsyncOption<'b>) : AsyncOption<'b> =
        async {
            match! aoX with
            | Some x -> return! f x
            | None -> return None
        }

    member _.Return(x: 'a) : AsyncOption<'a> =
        async { return Some x }
```

⚠️ Limits: not reusable, just copiable for `asyncResult` for instance

## CE Applicative

An applicative CE is revealed through the usage of the `and!` keyword _(F♯ 5)._

An applicative CE builder should define these methods:

```fsharp
// Method        | Signature                        | Equivalence
    MergeSources : mx: M<X> * my: M<Y> -> M<X * Y>  ; map2 (fun x y -> x, y) mx my
    BindReturn   : m: M<T> * f: (T -> U) -> M<U>    ; map f m
```

### CE Applicative example - `validation {}`

```fsharp
type Validation<'t, 'e> = Result<'t, 'e list>

type ValidationBuilder() =
    member _.BindReturn(x: Validation<'t, 'e>, f: 't -> 'u) =
        Result.map f x

    member _.MergeSources(x: Validation<'t, 'e>, y: Validation<'u, 'e>) =
        match (x, y) with
        | Ok v1,    Ok v2    -> Ok(v1, v2)     // Merge both values in a pair
        | Error e1, Error e2 -> Error(e1 @ e2) // Merge errors in a single list
        | Error e, _ | _, Error e -> Error e   // Short-circuit single error source

let validation = ValidationBuilder()
```

**Usage:** validate a customer

* Name not null or empty
* Height strictly positive

```fsharp
type [<Measure>] cm
type Customer = { Name: string; Height: int<cm> }

let validateHeight height =
    if height <= 0<cm>
    then Error "Height must be positive"
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
let c2 = Customer.tryCreate "Bob" 0<cm> // Error ["Height must be positive"]
let c3 = Customer.tryCreate "" 0<cm>    // Error ["Name can't be empty"; "Height must be positive"]
```

Desugaring:

```fsharp
validation {                                ; validation.BindReturn(
                                            ;     validation.MergeSources(
    let! name = validateName "Bob"          ;         validateName "Bob",
    and! height = validateHeight 0<cm>      ;         validateHeight 0<cm>
                                            ;     ),
    return { Name = name; Height = height } ;     (fun (name, height) -> { Name = name; Height = height })
}                                           ; )
```

### CE Applicative trap

⚠️ The compiler accepts that we define `ValidationBuilder` without `BindReturn` but with `Bind` and `Return`. But in this case, we can loose the applicative behavior and it enables monadic CE bodies!

### CE Applicative - FsToolkit `validation {}`

[FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling/) offers a similar `validation {}`.

The desugaring reveals the definition of more methods: `Delay`, `Run`, `Source`📍

```fsharp
validation {                                ;  validation.Run(
    let! name = validateName "Bob"          ;      validation.Delay(fun () ->
    and! height = validateHeight 0<cm>      ;          validation.BindReturn(
    return { Name = name; Height = height } ;              validation.MergeSources(
}                                           ;                  validation.Source(validateName "Bob"),
                                            ;                  validation.Source(validateHeight 0<cm>)
                                            ;              ),
                                            ;              (fun (name, height) -> { Name = name; Height = height })
                                            ;          )
                                            ;      )
                                            ;  )
```

### `Source` methods

In FsToolkit `validation {}`, there are a couple of `Source` defined:

* The main definition is the `id` function.
* Another overload is interesting: it converts a `Result<'a, 'e>` into a `Validation<'a, 'e>`. As it's defined as an extension method, it has a lower priority for the compiler, leading to a better type inference. Otherwise, we would need to add type annotations.

☝️ **Note:** `Source` documentation is scarce. The most valuable information comes from a [question on stackoverflow](https://stackoverflow.com/a/35301315/8634147) mentioned in FsToolkit source code!

## Creating CEs

### Types

The CE builder methods definition can involve not 2 but 3 types:

* The wrapper type `M<T>`
* The `Delayed<T>` type
* An `Internal<T>` type

### `M<T>` wrapper type

☝️ We use the generic type notation `M<T>` to indicate any of these aspects: generic or container.

Examples of candidate types:

* Any generic type
* Any monoidal, monadic, or applicative type
* `string` as it contains `char`s
* Any type itself as `type Identity<'t> = 't` – see previous `logger {}` CE

### `Delayed<T>` type

* Return type of `Delay`
* Parameter to `Run`, `Combine`, `While`, `TryWith`, `TryFinally`
* Default type when `Delay` is not defined: `M<T>`
* Common type for a real delay: `unit -> M<T>` - see `member _.Delay f = f`

#### `Delayed<T>` type example: `eventually {}`

Union type used for both wrapper and delayed types:

```fsharp
// Code adapted from https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions
type Eventually<'t> =
    | Done of 't
    | NotYetDone of (unit -> Eventually<'t>)

type EventuallyBuilder() =
    member _.Return x = Done x
    member _.ReturnFrom expr = expr
    member _.Zero() = Done()
    member _.Delay f = NotYetDone f

    member m.Bind(expr, f) =
        match expr with
        | Done x -> f x
        | NotYetDone work -> NotYetDone(fun () -> m.Bind(work (), f))

    member m.Combine(command, expr) = m.Bind(command, (fun () -> expr))

let eventually = EventuallyBuilder()
```

The output values are maint to be evaluated interactively, step by step:

```fsharp
let step = function
    | Done x -> Done x
    | NotYetDone func -> func ()

let delayPrintMessage i =
    NotYetDone(fun () -> printfn "Message %d" i; Done ())

let test = eventually {
    do! delayPrintMessage 1
    do! delayPrintMessage 2
    return 3 + 4
}

let step1 = test |> step   // val step1: Eventually<int> = NotYetDone <fun:Bind@14-1>
let step2 = step1 |> step  // Message 1 ↩ val step2: Eventually<int> = NotYetDone <fun:Bind@14-1>
let step3 = step2 |> step  // Message 2 ↩ val step3: Eventually<int> = Done 7
```

### `Internal<T>` type

`Return`, `ReturnFrom`, `Yield`, `YieldFrom`, `Zero` can return a type internal to the CE. `Combine`, `Delay`, and `Run` handle this type.

```fsharp
// Example: list builder using sequences internally, like the list comprehension does.
type ListSeqBuilder() =
    member inline _.Zero() = Seq.empty
    member inline _.Yield(x) = Seq.singleton x
    member inline _.YieldFrom(xs) = Seq.ofList xs
    member inline _.Delay([<InlineIfLambda>] thunk) = Seq.delay thunk
    member inline _.Combine(xs, ys) = Seq.append xs ys
    member inline _.For(xs, [<InlineIfLambda>] f) = xs |> Seq.collect f
    member inline _.Run(xs) = xs |> Seq.toList

let listSeq = ListSeqBuilder()
```

💡 Highlights the usefulness of `ReturnFrom`, `YieldFrom`, implemented as an _identity_ function until now.

### Builder methods without type

{% hint style="success" %}
#### Another trick regarding types

As a F# developer in a .NET team using only C#, feel free to use these materials (both git-book and slides) to train your team.

💡 Any type can be turned into a CE by adding builder methods as extensions.
{% endhint %}

Example: `activity {}` CE to configure an `Activity` without passing the instance

* Type with builder extension methods: `System.Diagnostics.Activity`
* Return type: `unit` (no value returned)
* Internal type involved: `type ActivityAction = delegate of Activity -> unit`
* CE behaviour:
  * monoidal internally: composition of `ActivityAction`
  * like a `State` monad externally, with only the setter(s) part

```fsharp
type ActivityAction = delegate of Activity -> unit

// Helpers
let inline private action ([<InlineIfLambda>] f: Activity -> _) =
    ActivityAction(fun ac -> f ac |> ignore)

let inline addLink link = action _.AddLink(link)
let inline setTag name value = action _.SetTag(name, value)
let inline setStartTime time = action _.SetStartTime(time)

// CE Builder Methods
type ActivityExtensions =
    [<Extension; EditorBrowsable(EditorBrowsableState.Never)>]
    static member inline Zero(_: Activity | null) = ActivityAction(fun _ -> ())

    [<Extension; EditorBrowsable(EditorBrowsableState.Never)>]
    static member inline Yield(_: Activity | null, [<InlineIfLambda>] a: ActivityAction) = a

    [<Extension; EditorBrowsable(EditorBrowsableState.Never)>]
    static member inline Combine(_: Activity | null, [<InlineIfLambda>] a1: ActivityAction, [<InlineIfLambda>] a2: ActivityAction) =
        ActivityAction(fun ac -> a1.Invoke(ac); a2.Invoke(ac))

    [<Extension; EditorBrowsable(EditorBrowsableState.Never)>]
    static member inline Delay(_: Activity | null, [<InlineIfLambda>] f: unit -> ActivityAction) = f() // ActivityAction is already delayed

    [<Extension; EditorBrowsable(EditorBrowsableState.Never)>]
    static member inline Run(ac: Activity | null, [<InlineIfLambda>] f: ActivityAction) =
        match ac with
        | null -> ()
        | ac -> f.Invoke(ac)

// ---

let activity = new Activity("Tests")

activity {
    setStartTime DateTime.UtcNow
    setTag "count" 2
}
```

* The `activity` instance supports the CE syntax thanks to its extensions.
* The extension methods are marked as not `EditorBrowsable` for proper DevExp.
* Externally, the `activity` is implicit in the CE body, like a `State` monad.
* Internally, the state is handled as a composition of `ActivityAction`.
* The final `Run` enables us to evaluate the built `ActivityAction`,\
  resulting in the change (mutation) of the `activity` (the side effect).

### Custom operations 🚀

What: builder methods annotated with `[<CustomOperation("myOperation")>]`

Use cases: add new keywords, build a custom DSL → Example: the `query` core CE supports `where` and `select` keywords like LINQ

⚠️ **Warning:** you may need additional things that are not well documented:

* Additional properties for the `CustomOperation` attribute:
  * `AllowIntoPattern`, `MaintainsVariableSpace`
  * `IsLikeJoin`, `IsLikeGroupJoin`, `JoinConditionWord`
  * `IsLikeZip`...
* Additional attributes on the method parameters, like `[<ProjectionParameter>]`

🔗 [Computation Expressions Workshop: 7 - Query Expressions | GitHub](https://github.com/panesofglass/computation-expressions-workshop/blob/master/exercises/07_Queries.pdf)

### CE creation guidelines 📃

* Choose the main **behaviour**: monoidal? monadic? applicative?
  * Prefer a single behaviour unless it's a generic/multi-purpose CE
* Create a **builder** class
* Implement the main **methods** to get the selected behaviour
* Use/Test your CE to verify it compiles _(see typical compilation errors below),_ produces the expected result, and performs well.

```txt
1. This control construct may only be used if the computation expression builder defines a 'Delay' method
   => Just implement the missing method in the builder.
2. Type constraint mismatch. The type ''b seq' is not compatible with type ''a list'
   => Inspect the builder methods and track an inconsistency.
```

### CE creation tips 💡

* Get inspired by existing codebases that provide CEs - examples:
  * FSharpPlus → `monad`
  * FsToolkit.ErrorHandling → `option`, `result`, `validation`
  * [Expecto](https://github.com/haf/expecto): Testing library (`test "..." {...}`)
  * [Farmer](https://github.com/compositionalit/farmer): Infra as code for Azure (`storageAccount {...}`)
  * [Saturn](https://saturnframework.org/): Web framework on top of ASP.NET Core (`application {...}`)
* Overload methods to support more use cases like different input types
  * `Async<Result<_,_>>` + `Async<_>` + `Result<_,_>`
  * `Option<_>` and `Nullable<_>`

### CE benefits ✅

* **Increased Readability**: imperative-like code, DSL _(Domain Specific Language)_
* **Reduced Boilerplate**: hides a "machinery"
* **Extensibility**: we can write our own "builder" for specific logic

### CE limits ⚠️

* **Compiler error messages** within a CE body can be cryptic
* **Nesting different CEs** can make the code more cumbersome
  * E.g. `async` + `result`
  * Alternative: custom combining CE - see `asyncResult` in [FsToolkit](https://demystifyfp.gitbook.io/fstoolkit-errorhandling/#a-motivating-example)
* Writing our own CE can be **challenging**
  * Implementing the right methods, each the right way
  * Understanding the underlying concepts

## 🍔 Quiz

#### Question 1: **What is the primary purpose of computation expressions in F#?**

**A.** To replace all functional programming patterns

**B.** To provide imperative-like syntax for sequencing and combining computations

**C.** To eliminate the need for type annotations

**D.** To make F# code compatible with C#

<details>

<summary>Answer</summary>

**B.** To provide imperative-like syntax for sequencing and combining computations ✅

</details>

#### Question 2: **Which keywords identify a monadic computation expression?**

**A.** `yield` and `yield!`

**B.** `let!` and `return`

**C.** `let!` and `and!`

**D.** `do!` and `while`

<details>

<summary>Answer</summary>

**A.** `yield` and `yield!` keywords identify a monoidal CE ✖️

**B.** `let!` and `return` keywords identify a monadic CE ✅

**C.** `let!` and `and!` keywords identify a applicative CE ✖️

**D.** `do!` and `while` keywords can be used with any kind of CE ✖️

</details>

#### Question 3: **In a computation expression builder, what does the `Bind` method correspond to?**

**A.** The `yield` keyword

**B.** The `return` keyword

**C.** The `let!` keyword

**D.** The `else` keyword when omitted

<details>

<summary>Answer</summary>

**A.** The `yield` keyword corresponds to the `Yield` method ✖️

**B.** The `return` keyword corresponds to the `Return` method ✖️

**C.** The `let!` keyword corresponds to the `Bind` method ✅

**D.** The `else` keyword, when omitted, corresponds to the `Zero` method ✖️

</details>

#### Question 4: **What is the signature of a typical monadic `Bind` method?**

**A.** `M<T> -> M<T>`

**B.** `T -> M<T>`

**C.** `M<T> * (T -> M<U>) -> M<U>`

**D.** `M<T> * M<U> -> M<T * U>`

<details>

<summary>Answer</summary>

**A.** `M<T> -> M<T>` is the typical signature of `ReturnFrom` and `YieldFrom` methods ✖️

**B.** `T -> M<T>` is the typical signature of `Return` and `Yield` methods ✖️

**C.** `M<T> * (T -> M<U>) -> M<U>` is the typical signature of the `Bind` method ✅

**D.** `M<T> * M<U> -> M<T * U>` is the typical signature of `MergeSources` method ✖️

</details>

## 📜 CE wrap up

* Syntactic sugar: inner syntax: standard or "banged" (`let!`)\
  → Imperative-like • Easy to use
* CE is based on a _builder_
  * instance of a class with standard methods like `Bind` and `Return`
* _Separation of Concerns_
  * Business logic in the CE body
  * Machinery behind the scene in the CE builder
* Little issues for nesting or combining CEs
* Underlying functional patterns: monoid, monad, applicative
* Libraries: FSharpPlus, FsToolkit, Expecto, Farmer, Saturn...

## 🔗 Additional resources

* [Code examples in FSharpTraining.sln](https://github.com/rdeneau/formation-fsharp/tree/main/src/FSharpTraining) —Romain Deneau
* [_The "Computation Expressions" series_](https://fsharpforfunandprofit.com/series/computation-expressions/) —F# for Fun and Profit
* [All CE methods | Learn F#](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/computation-expressions#creating-a-new-type-of-computation-expression) —Microsoft
* [Computation Expressions Workshop](https://github.com/panesofglass/computation-expressions-workshop)
* [The F# Computation Expression Zoo](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/computation-zoo.pdf) —Tomas Petricek and Don Syme
  * [Documentation | Try Joinads](http://tryjoinads.org/docs/computations/home.html) —Tomas Petricek
* Extending F# through Computation Expressions: 📹 [Video](https://youtu.be/bYor0oBgvws) • 📜 [Article](https://panesofglass.github.io/computation-expressions/#/)
