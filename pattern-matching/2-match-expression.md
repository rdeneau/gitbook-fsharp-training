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

A C# `switch` must always define a default case.\
Otherwise: compile warning, 💥 `MatchFailureException` at runtime

Not necessary in a F# _match expression_ if branches cover all cases because the compiler checks for completeness and "dead" branches

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

Example: checking all the cases of a union type allows you to manage the addition of a case by a warning at compile time:\
`Warning FS0025: Special criteria incomplete in this expression`

* Detection of accidental addition
* Identification of the code to change to handle the new case

## Guard

Syntax: `pattern1 when condition`\
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

💡 The _guard_ is only evaluated if the pattern is satisfied.

## Guard _vs_ OR Pattern

The OR pattern has a higher _precedence/priority_ than the _Guard_ :

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

## Pattern composition

Pattern matching is powerful not only due to its exhaustivity but also because of the pattern composition.

→ **Example:**

```fsharp
match feature with
| { Value = FeatureValue.Boolean(Some true) } -> "Enabled"
| _ -> "Disabled"
```

{% hint style="info" %}
### Notes

This example demonstrates how we match the data on 4 levels:

1. `feature` is a record with the `Value` field  (possibly amongst other fields)
2. `Value` has the case `Boolean` of the union type `FeatureValue`.
3. This union case contains an optional boolean (`bool option` type).
4. We detect when the provided value is `true`.
{% endhint %}

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

**Benefits:**

1. In pipelines

```fsharp
value
|> is123
|> function
    | true  -> "ok"
    | false -> "ko"
```

2. More concise function

```fsharp
let is123 = function
    | 1 | 2 | 3 -> true
    | _ -> false
```

{% hint style="warning" %}
### Limitations

As the parameter is implicit, it can make the code more difficult to understand!\
→ See [#point-free-style](../functions/5-operators.md#point-free-style "mention")
{% endhint %}

**Example:** a function with both explicit parameters and the `function` keyword\
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

The equivalent of the pattern matching exhaustivity in FP is\
... the [Visitor design pattern](https://refactoring.guru/design-patterns/visitor) in OOP

> Visitor is a behavioral design pattern that lets you **separate algorithms from the objects** on which they operate.

→ It's FP in OOP, much very convoluted.
