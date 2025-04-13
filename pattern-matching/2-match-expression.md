# Match expression

Similar to a `switch` expression in C♯ 8.0 but more powerful thanks to patterns

Syntax:

```fsharp
match test-expression with
| pattern1 [ when condition ] -> result-expression1
| pattern2 [ when condition ] -> result-expression2
| ...
```

Returns the result of the 1st branch whose pattern "matches" `test-expression`

☝ **Note:** all branches must return the same type!

## Exhaustivity

A C# `switch` must always define a default case. \
Otherwise: compile warning, 💥 `MatchFailureException` at runtime

Not necessary in a F# *match expression* if branches cover all cases because the compiler checks for completeness and "dead" branches

```fsharp
let fn x =
    match x with
    | Some true  -> "ok"
    | Some false -> "ko"
    | None       -> ""
    | _          -> "?"
//    ~  ⚠️ Warning FS0026: his rule will never be matched
```

☝ **Tip:** the more branches are exhaustive, the more code is explicit and safe

Example: checking all the cases of a union type allows you to manage the addition of a case by a warning at compile time: \
`Warning FS0025: Special criteria incomplete in this expression`

- Detection of accidental addition
- Identification of the code to change to handle the new case

## Guard

Syntax: `pattern1 when condition` \
Usage: to refine a pattern, using constraints on variables

```fsharp
let classifyBetween low top value =
    match value with
    | x when x < low -> "Inf"
    | x when x = low -> "Low"
    | x when x = top -> "Top"
    | x when x > top -> "Sup"
    | _ -> "Between"

let test1 = 1 |> classifyBetween 1 5  // "Low"
let test2 = 6 |> classifyBetween 1 5  // "Sup"
```

💡 The *guard* is only evaluated if the pattern is satisfied.

## Guard *vs* OR Pattern

The OR pattern has a higher *precedence/priority* than the *Guard* :

```fsharp
type Parity = Even of int | Odd of int

let parityOf value =
    if value % 2 = 0 then Even value else Odd value

let hasSquare square value =
    match parityOf square, parityOf value with
    | Even x2, Even x
    | Odd  x2, Odd  x
        when x2 = x*x -> true  // 👈 The guard is covering the 2 previous patterns
    | _ -> false

let test1 = 2 |> hasSquare 4  // true
let test2 = 3 |> hasSquare 9  // true
```

## Match function

Syntax:

```fsharp
function
| pattern1 [ when condition ] -> result-expression1
| pattern2 [ when condition ] -> result-expression2
| ...
```

Equivalent to a lambda taking an implicit parameter which is "matched":

```fsharp
fun value ->
    match value with
    | pattern1 [ when condition ] -> result-expression1
    | pattern2 [ when condition ] -> result-expression2
    | ...
```

### Intérêts

Interest

1. In pipeline

```fsharp
value
|> is123
|> function
    | true  -> "ok"
    | false -> "ko"
```

2. Terser function

```fsharp
let is123 = function
    | 1 | 2 | 3 -> true
    | _ -> false
```

### Limites

Limitations

⚠️ Implicit parameter => can make the code more difficult to understand!

Example: function declared with other explicit parameters \
→ The number of parameters and their order can be wrong:

```fsharp
let classifyBetween low high = function  // 👈 3 parameters : `low`, `high`, and another one implicit
    | x when x < low  -> "Inf"
    | x when x = low  -> "Low"
    | x when x = high -> "High"
    | x when x > high -> "Sup"
    | _ -> "Between"

let test1 = 1 |> classifyBetween 1 5  // "Low"
let test2 = 6 |> classifyBetween 1 5  // "Sup"
```

## Exhaustivity in OOP

The equivalent of the pattern matching exhaustivity in FP is \
... the [Visitor design pattern](https://refactoring.guru/design-patterns/visitor) in OOP

> Visitor is a behavioral design pattern that lets you **separate algorithms from the objects** on which they operate.

→ It's FP in OOP, much very convoluted: see *double-dispatch technique*

## `fold` function 🚀

Function associated with a union type and hiding the *matching* logic \
Takes N+1 parameters for a union type with N *cases*

```fsharp
type [<Measure>] C
type [<Measure>] F

type Temperature =
    | Celsius     of float<C>
    | Fahrenheint of float<F>

module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius    : float<C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float<F> -> 'T
```

### Usage

```fsharp
module Temperature =
    // ...
    let [<Literal>] FactorC2F = 1.8<F/C>
    let [<Literal>] DeltaC2F = 32.0<F>

    let celsiusToFahrenheint x = (x * FactorC2F) + DeltaC2F  // float<C> -> float<F>
    let fahrenheintToCelsius x = (x - DeltaC2F) / FactorC2F  // float<F> -> float<C>

    let toggleUnit temperature =
        temperature |> fold
            (celsiusToFahrenheint >> Fahrenheint)
            (fahrenheintToCelsius >> Celsius)

let t1 = Celsius 100.0<C>
let t2 = t1 |> Temperature.toggleUnit  // Fahrenheint 212.0
```

### Interest

`fold` hides the implementation details of the type

For example, we could add a `Kelvin` *case* and only impact `fold`, not the functions that call it, such as `toggleUnit` in the previous example

```fsharp
type [<Measure>] C
type [<Measure>] F
type [<Measure>] K  // 🌟

type Temperature =
    | Celsius     of float<C>
    | Fahrenheint of float<F>
    | Kelvin      of float<K>  // 🌟

module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius: float<C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float<F> -> 'T
        | Kelvin x      -> mapCelsius (x * 1.0<C/K> + 273.15<C>)  // 🌟

Kelvin 273.15<K>
|> Temperature.toggleUnit
|> Temperature.toggleUnit
// Celsius 0.0<C>
```
