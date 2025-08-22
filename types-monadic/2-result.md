# Result type

A.k.a `Either` _(Haskell)_

Models a _double-track_ Success/Failure

```fsharp
type Result<'Success, 'Error> = // 2 generic parameters
    | Ok of 'Success  // Success Track
    | Error of 'Error // Failure Track
```

Functional way of dealing with business errors _(expected errors)_

* Allows exceptions to be used only for exceptional errors
* As soon as an operation fails, the remaining operations are not launched

🔗 [_Railway-oriented programming (ROP)_](https://fsharpforfunandprofit.com/rop/)

## Result module

Main functions:

`map f result` : to map the success\
• `('T -> 'U) -> Result<'T, 'Error> -> Result<'U, 'Error>`

`mapError f result` : to map the error\
• `('Err1 -> 'Err2) -> Result<'T, 'Err1> -> Result<'T, 'Err2>`

`bind f result` : same as `map` with `f` returning a `Result`\
• `('T -> Result<'U, 'Error>) -> Result<'T, 'Error> -> Result<'U, 'Error>`\
• 💡 The result is flattened, like the `flatMap` function on JS arrays\
• ⚠️ Same type of `'Error` for `f` and the input `result`.

💡 **Since F♯ 7.0**, more functions have been added to the `Result` module, but some functions are missing compared to the `Option` module:

| Result                           | Option                            | List                     |
| -------------------------------- | --------------------------------- | ------------------------ |
| `Ok value`                       | `Some value`                      | `[ value ]`              |
| `Error err`                      | `None`                            | `[ ]`                    |
| `Result.isOk` / `Result.isError` | `Option.isSome` / `Option.isNone` | × / `List.isEmpty`       |
| `Result.contains okValue`        | `Option.contains someValue`       | `List.contains value`    |
| `Result.count`                   | `Option.count`                    | `List.length` ❗ _(O(N))_ |
| `Result.defaultValue value`      | `Option.defaultValue value`       | ×                        |
| `Result.defaultWith defThunk`    | `Option.defaultWith defThunk`     | ×                        |
| ×                                | `Option.filter predicate`         | `List.filter predicate`  |
| `Result.iter action`             | `Option.iter action`              | `List.iter action`       |
| `Result.map f`                   | `Option.map f`                    | `List.map f`             |
| `Result.mapError f`              | ×                                 | ×                        |
| `Result.bind f`                  | `Option.bind f`                   | `List.bind f`            |
| `Result.fold f state`            | `Option.fold f state`             | `List.fold f state`      |
| `Result.toArray` / `toList`      | `Option.toArray` / `toList`       | ×                        |
| `Result.toOption`                | ×                                 | ×                        |

## Quiz 🎲

Implement `Result.map` and `Result.bind`

💡 **Tips:**

* _Map_ the _Success_ track
* Access the _Success_ value using pattern matching

<details>

<summary>Solution</summary>

```fsharp
// ('T -> 'U) -> Result<'T, 'Error> -> Result<'U, 'Error>
let map f result =
    match result with
    | Ok x    -> Ok (f x)  // ☝ Ok -> Ok
    | Error e -> Error e   // ⚠️ The 2 `Error e` don't have the same type!

// ('T -> Result<'U, 'Error>) -> Result<'T, 'Error>
//                            -> Result<'U, 'Error>
let bind f result =
    match result with
    | Ok x    -> f x       // ☝ `f x` already returns a `Result`
    | Error e -> Error e
```

</details>

## Success/Failure tracks

`map`: no track change

```txt
Track      Input          Operation      Output
Success ─ Ok x    ───► map( x -> y ) ───► Ok y
Failure ─ Error e ───► map(  ....  ) ───► Error e
```

`bind`: eventual routing to Failure track, but never vice versa

```txt
Track     Input              Operation           Output
Success ─ Ok x    ─┬─► bind( x -> Ok y     ) ───► Ok y
                   └─► bind( x -> Error e2 ) ─┐
Failure ─ Error e ───► bind(     ....      ) ─┴─► Error ~
```

☝ The _mapping/binding_ operation is never executed in track Failure.

## `Result` _vs_ `Option`

`Option` can represent the result of an operation that may fail\
☝ But if it fails, the option doesn't contain the error, just `None`

`Option<'T>` ≃ `Result<'T, unit>`\
→ `Some x` ≃ `Ok x`\
→ `None` ≃ `Error ()`\
→ See `Result.toOption` _(built-in)_ and `Result.ofOption` _(below)_

```fsharp
[<RequireQualifiedAccess>]
module Result =
    let ofOption error option =
        match option with
        | Some x -> Ok x
        | None -> Error error
```

#### 📅 Dates

* The `Option` type has been part of F# from the beginning
* The `Result` type is more recent: introduced in F# 4.1 (2016)

#### 📝 Memory

* The `Option` type (alias: `option`) is a regular union: a reference type
* The `Result` type is a _struct_ union: a value type
* The `ValueOption` type (alias: `voption`) is a _struct_ union: `ValueNone | ValueSome of 't`

### Example

Let's change our previous `checkAnswer` to indicate the `Error`:

```fsharp
type Answer = A | B | C | D
type Error = InvalidInput of string | WrongAnswer of Answer

let tryParseAnswer =
    function
    | "A" -> Ok A
    | "B" -> Ok B
    | "C" -> Ok C
    | "D" -> Ok D
    | s   -> Error(InvalidInput s)

let checkAnswerIs expected actual =
    if actual = expected then Ok actual else Error(WrongAnswer actual)

let printAnswerCheck (givenAnswer: string) =
    tryParseAnswer givenAnswer
    |> Result.bind (checkAnswerIs B)
    |> function
       | Ok x                  -> printfn $"%A{x}: ✅ Correct"
       | Error(WrongAnswer x)  -> printfn $"%A{x}: ❌ Wrong Answer"
       | Error(InvalidInput s) -> printfn $"%s{s}: ❌ Invalid Input"

printAnswerCheck "X";;  // X: ❌ Invalid Input
printAnswerCheck "A";;  // A: ❌ Wrong Answer
printAnswerCheck "B";;  // B: ✅ Correct
```
