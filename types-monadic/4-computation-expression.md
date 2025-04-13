# 🚀 Computation expression (CE)

Syntactic sugar hiding a "machinery"

- Applies the *Separation of Concerns* principle
- Code should be more readable inside the *computation expression*

Syntax: `builder { expr }`

- `builder` instance of a "Builder" 📍
- `expr` can contain `let`, `let!`, `do!`, `yield`, `yield!`, `return`, `return!`

💡 **Note:** `seq`, `async` and `task` are computation expressions baked into the language.

## Builder

A *computation expression* relies on an object called *Builder*. This object can be used to store a background state.

For each supported keyword (`let!`, `return`...), the *Builder* implements one or more related methods. Examples:

- `builder { return expr }` → `builder.Return(expr)`
- `builder { let! x = expr; cexpr }` → `builder.Bind(expr, (fun x -> {| cexpr |}))`

The *builder* can also wrap the result in a type of its own:

- `async { return x }` returns an `Async<'X>`
- `seq { yield x }` returns a `Seq<'X>`

## Builder desugaring

The compiler translates to the *builder* methods. \
→ The CE hides the complexity of these calls, which are often nested:

```fsharp
seq {
    for n in list do
        yield n
        yield n * 10 }

// Desugared as:
seq.For(list, fun () ->
    seq.Combine(seq.Yield(n),
                seq.Delay(fun () -> seq.Yield(n * 10)) ) )
```

## Examples

### `logger`

Need: log the intermediate values of a calculation

```fsharp
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
2. *Error prone*: forget a `log`, log wrong value...

```fsharp
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

💡 **Solutions**

Make logs implicit in a CE when `let!` / `Bind` :

```fsharp
type LoggingBuilder() =
    let log value = printfn $"{value}"; value
    member _.Bind(x, f) = x |> log |> f
    member _.Return(x) = x

let logger = LoggingBuilder()

//---

let loggedCalc = logger {
    let! x = 42
    let! y = 43
    let! z = x + y
    return z
}
```

☝️ Each time we do a `let! var = value` in the `logger` CE, the `value` is printed in the console and is bound to `var`.

### `maybe`

Need: simplify the sequence of "trySomething" returning an `Option`

```fsharp
let tryDivideBy bottom top = // (bottom: int) -> (top: int) -> int option
    if (bottom = 0) or (top % bottom <> 0)
    then None
    else Some (top / bottom)

// W/o CE
let division =
    36
    |> tryDivideBy 2                // Some 18
    |> Option.bind (tryDivideBy 3)  // Some 6
    |> Option.bind (tryDivideBy 2)  // Some 3

// With CE
type MaybeBuilder() =
    member _.Bind(x, f) = x |> Option.bind f
    member _.Return(x) = Some x

let maybe = MaybeBuilder()

let division' = maybe {
    let! v1 = 36 |> tryDivideBy 2
    let! v2 = v1 |> tryDivideBy 3
    let! v3 = v2 |> tryDivideBy 2
    return v3
}
```

**Result:** ✅ Symmetry, ❌ Intermediate values

## Limits

### nested CEs

✅ Different CEs can be nested
❌ But code becomes difficult to understand

Example: combining `logger` and `maybe` ❓

Alternative solution 🚀🚀:

```fsharp
// Define an operator for `bind`
let inline (>>=) x f = x |> Option.bind f

let logM value = printfn $"{value}"; Some value  // 'a -> 'a option

let division' =
    36 |> tryDivideBy 2 >>= logM
      >>= tryDivideBy 3 >>= logM
      >>= tryDivideBy 2 >>= logM
```

### Combining CEs

How to combine `Async` + `Option`/`Result` ? \
→ `asyncResult` CE + helpers in [FsToolkit](https://demystifyfp.gitbook.io/fstoolkit-errorhandling/#a-motivating-example)

```fsharp
type LoginError =
    | InvalidUser | InvalidPassword
    | Unauthorized of AuthError | TokenErr of TokenError

let login username password =
    asyncResult {
        // tryGetUser: string -> Async<User option>
        let! user = username |> tryGetUser |> AsyncResult.requireSome InvalidUser

        // isPasswordValid: string -> User -> bool
        do! user |> isPasswordValid password |> Result.requireTrue InvalidPassword

        // authorize: User -> Async<Result<unit, AuthError>>
        do! user |> authorize |> AsyncResult.mapError Unauthorized

        // createAuthToken: User -> Result<AuthToken, TokenError>
        return! user |> createAuthToken |> Result.mapError TokenErr
    } // Async<Result<AuthToken, LoginError>>
```
