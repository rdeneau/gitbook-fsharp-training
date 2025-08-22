# Active patterns

## Pattern Matching Limits

Limited number of patterns

Impossibility of factoring the action of patterns with their own guard

* `Pattern1 when Guard1 | Pattern2 when Guard2 -> do` 💥
* `Pattern1 when Guard1 -> do | Pattern2 when Guard2 -> do` 😕

Patterns are not first-class citizens\
&#xNAN;_&#x45;x: a function can't return a pattern_\
→ Just a kind of syntactic sugar

Patterns interact badly with an OOP style

## Origin of _Active Patterns_

> 🔗 [_Extensible pattern matching via a lightweight language extension_](https://www.microsoft.com/en-us/research/publication/extensible-pattern-matching-via-a-lightweight-language-extension/)\
> ℹ️ 2007 publication by Don Syme, Gregory Neverov, James Margetson

Integrated into F♯ 2.0 (2010)

💡 **Ideas**

* Enable _pattern matching_ on other data structures
* Make these new patterns first-class citizens

## Syntax

General syntax : `let (|Cases|) [arguments] valueToMatch = expression`

1. **Function** with a special name defined in "bananas" `(|...|)`
2. Set of 1..N **cases** in which to store the `valueToMatch` parameter

💡 Kind of _factory_ function of an "anonymous" **union** type, defined _inline_

## Types

There are 4 types of active patterns:

<table><thead><tr><th width="277">Name</th><th width="136.09088134765625">Cases</th><th>Exhaustive</th><th>Parameters</th></tr></thead><tbody><tr><td><code>1.  Simple    Total</code></td><td>1</td><td>✅ Yes</td><td>❔ 0+</td></tr><tr><td><code>2.  Multiple  Total</code></td><td>2+</td><td>✅ Yes</td><td>❌ 0</td></tr><tr><td><code>3.            Partial</code></td><td>1</td><td>❌ No</td><td>❌ 0</td></tr><tr><td><code>4.            Parametric</code></td><td>1</td><td>❌ No</td><td>✅ 1+</td></tr></tbody></table>

💡 _Partial_ and _total_ indicate the feasibility of "placing the input value in the box(es)"

* **Partial**: there is not always a corresponding box
* **Total**: there is always a corresponding box → exhaustive pattern

### Simple total active pattern

_A.k.a Single-case Total Pattern_

Syntax: `let (|Case|) [...parameters] value = Case [data]`\
Usage: on-site value adjustment

```fsharp
/// Ensure the given string is never null
let (|NotNullOrEmpty|) (s: string) = // string -> string
    if s |> isNull then System.String.Empty else s

// Usages:
let (NotNullOrEmpty a) = "abc" // val a: string = "abc"
let (NotNullOrEmpty b) = null  // val b: string = ""
```

Can accept **parameters** \
⚠️ Usually more difficult to understand

```fsharp
/// Get the value in the given option if there is some, otherwise the specified default value
let (|Default|) defaultValue option = option |> Option.defaultValue defaultValue
//              'T     -> 'T option -> 'T

// Usages:
let (Default "unknown" john) = Some "John"  // val john: string = "John"
let (Default 0 count) = None                // val count: int = 0

// Template function
let (|ValueOrUnknown|) = (|Default|) "unknown" // string option -> string

let (ValueOrUnknown person) = None  // val person: string = "unknown"
```

Another example: extracting the polar form of a complex number

```fsharp
/// Extracts the polar form (Magnitude, Phase) of the given complex number
let (|Polar|) (x: System.Numerics.Complex) =
    x.Magnitude, x.Phase

/// Multiply the 2 complex numbers by adding their phases and multiplying their magnitudes
let multiply (Polar(m1, p1)) (Polar(m2, p2)) =  // Complex -> Complex -> Complex
    System.Numerics.Complex.FromPolarCoordinates(magnitude = m1 * m2, phase = p1 + p2)

// Without the active pattern: we need to add type annotations
let multiply' (x: System.Numerics.Complex) (y: System.Numerics.Complex) =
    System.Numerics.Complex.FromPolarCoordinates(x.Magnitude * y.Magnitude, x.Phase + y.Phase)
```

### Active pattern total multiple

_A.k.a Multiple-case Total Pattern_

Syntax: `let (|Case1|...|CaseN|) value = CaseI [dataI]`\
☝ No parameters❗

```fsharp
// Using an ad-hoc union type                       ┆  // Using a total active pattern
type Parity = Even of int | Odd of int with         ┆  let (|Even|Odd|) x =  // int -> Choice<int, int>
    static member Of(x) =                           ┆      if x % 2 = 0 then Even x else Odd x
        if x % 2 = 0 then Even x else Odd x         ┆
                                                    ┆
let hasSquare square value =                        ┆  let hasSquare' square value =
    match Parity.Of(square), Parity.Of(value) with  ┆      match square, value with
    | Even sq, Even v                               ┆      | Even sq, Even v
    | Odd  sq, Odd  v when sq = v*v -> true         ┆      | Odd  sq, Odd  v when sq = v*v -> true
    | _ -> false                                    ┆      | _ -> false
```

### Partial active pattern

Syntax: `let (|Case|_|) value = Some Case | Some data | None`\
→ Returns the type `'T option` if _Case_ includes data, otherwise `unit option`\
→ Pattern matching is non-exhaustive → a default case is required

```fsharp
let (|Integer|_|) (x: string) = // (x: string) -> int option
    match System.Int32.TryParse x with
    | true, i -> Some i
    | false, _ -> None

let (|Float|_|) (x: string) = // (x: string) -> float option
    match System.Double.TryParse x with
    | true, f -> Some f
    | false, _ -> None

let detectNumber = function
    | Integer i -> $"Integer {i}"   // detectNumber "10"
    | Float f -> $"Float {f}"       // detectNumber "1.1" = "Float 1.1" (US locale)
    | s -> $"NaN {s}"               // detectNumber "abc" = "NaN abc"
```

Similar example, where active patterns are written with the `Option.ofTuple` function:

```fsharp
module Option =
    let ofTuple =
        function
        | true, value -> Some value
        | false, _ -> None

module Parsing =
    open System

    let (|AsBoolean|_|) (value: string) =
        Boolean.TryParse value |> Option.ofTuple

    let (|AsInteger|_|) (value: string) =
        Int32.TryParse value |> Option.ofTuple

    let tryParseBoolean =
        function
        | AsBoolean b -> Ok b
        | AsInteger 0 -> Ok false
        | AsInteger 1 -> Ok true
        | value -> Error $"{value} is not a valid boolean"
```

💡 To see how much more readable the code is, let's write a more low-level version of `tryParseBoolean` where we see:

* Nesting `match` expressions
* Difficulty reading lines 6 and 7 due to double Booleans (`true..false`, `true..true`)

{% code lineNumbers="true" %}
```fsharp
    let tryParseBoolean' (value: string) =
        match Boolean.TryParse value with
        | true, b -> Ok b
        | false, _ ->
            match Int32.TryParse value with
            | true, 0 -> Ok false
            | true, 1 -> Ok true
            | _ -> Error $"{value} is not a valid boolean"
```
{% endcode %}

### Parametric partial active pattern

Syntax: `let (|Case|_|) ...arguments value = Some Case | Some data | None`

**Example 1: leap year**\
→ Year divisible by 4 but not by 100, except if divisible by 400

```fsharp
let (|DivisibleBy|_|) factor x =  // (factor: int) -> (x: int) -> unit option
    match x % factor with
    | 0 -> Some DivisibleBy
    | _ -> None

let isLeapYear year =  // (year: int) -> bool
    match year with
    | DivisibleBy 400 -> true
    | DivisibleBy 100 -> false
    | DivisibleBy   4 -> true
    | _               -> false
```

**Exemple 2 :** Regular expression

```fsharp
let (|Regexp|_|) pattern value =  // string -> string -> string list option
    let m = System.Text.RegularExpressions.Regex.Match(value, pattern)
    if not m.Success || m.Groups.Count < 1 then
        None
    else
        [ for g in m.Groups -> g.Value ]
        |> List.tail // drop "root" match
        |> Some
```

💡 Usages seen with the next example...

**Exemple :** Hexadecimal color

```fsharp
let hexToInt hex =  // string -> int // E.g. "FF" -> 255
    System.Int32.Parse(hex, System.Globalization.NumberStyles.HexNumber)

let (|HexaColor|_|) = function  // string -> (int * int * int) option
    // 👇 Uses the previous active pattern
    // 💡 The Regex searches for 3 groups of 2 chars being a number or a letter A..F
    | Regexp "#([0-9A-F]{2})([0-9A-F]{2})([0-9A-F]{2})" [ r; g; b ] ->
        Some <| HexaColor ((hexToInt r), (hexToInt g), (hexToInt b))
    | _ -> None

match "#0099FF" with
| HexaColor (r, g, b) -> $"RGB: {r}, {g}, {b}"
| otherwise -> $"'{otherwise}' is not a hex-color"
// "RGB: 0, 153, 255"
```

### Recap

```txt
Active pattern | Syntax                          | Signature
---------------|---------------------------------|-------------------------------------------------
Total multiple | let (|Case1|..|CaseN|)        x |                     'T -> Choice<'U1, .., 'Un>
... parametric | let (|Case1|..|CaseN|) p1..pp x | 'P1 -> .. -> 'Pp -> 'T -> Choice<'U1, .., 'Un>
Total simple   | let (|Case|)                  x |                     'T -> 'U
Partial simple | let (|Case|_|)                x |                     'T -> 'U option
... parametric | let (|Case|_|)         p1..pp x | 'P1 -> .. -> 'Pp -> 'T -> 'U option
```

## Understanding an active pattern

> Understanding how to use an active pattern...\
> ...can be a real **intellectual challenge**! 😵

👉 Explanations using the previous examples...

### Understanding a total active pattern

≃ _factory_ function of an "anonymous" union type

```fsharp
// -- Single-case ----
let (|Cartesian|) (x: System.Numerics.Complex) = Cartesian(x.Real, x.Imaginary)

let (Cartesian(r, i)) = System.Numerics.Complex(1.0, 2.0)
// val r: float = 1.0
// val i: float = 2.0

// -- Double-case ----
let (|Even|Odd|) x = if x % 2 = 0 then Even else Odd

let printParity = function
    | Even as n -> printfn $"%i{n} is even"
    | Odd  as n -> printfn $"%i{n} is odd"

printParity 1;;  // 1 is odd
printParity 10;; // 10 is even
```

### Understanding a partial active pattern

☝ Distinguish parameters _(input)_ from data _(output)_

Examine the active pattern signature: `[...params ->] value -> 'U option`

* **N-1 parameters:** active pattern parameters
* **Last parameter:** `value` to match
* **Return type:** `'U option` → data of type `'U`
  * when `unit option` → no data

→ Examples

1. `let (|Integer|_|) (s: string) : int option`
   * Usage `match s with Integer i` → `i: int` is the output data
2. `let (|DivisibleBy|_|) (factor: int) (x: int) : unit option`
   * Usage `match year with DivisibleBy 400` → `400` is the `factor` parameter
3. `let (|Regexp|_|) (pattern: string) (value: string) : string list option`
   * Usage `match s with Regexp "#([0-9...)" [ r; g; b ]`
     * `"#([0-9...)"` is the `pattern` parameter
     * `[ r; g; b ]` is the output data • It's a nested pattern: a list of 3 strings

## Exercice : fizz buzz with active pattern

Rewrite this fizz buzz using an active pattern `DivisibleBy`.

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let fizzBuzz = function
    | i when i |> isDivisibleBy 15 -> "FizzBuzz"
    | i when i |> isDivisibleBy  3 -> "Fizz"
    | i when i |> isDivisibleBy  5 -> "Buzz"
    | other -> string other

[1..15] |> List.map fizzBuzz
// ["1"; "2"; "Fizz"; "4"; "Buzz"; "Fizz";
//  "7"; "8"; "Fizz"; "Buzz"; "11";
//  "Fizz"; "13"; "14"; "FizzBuzz"]
```

### Solution

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let (|DivisibleBy|_|) factor number =
    if number |> isDivisibleBy factor then Some () else None
    // 👆 In F# 9, just `number |> isDivisibleBy factor` is enough 👍

let fizzBuzz = function
    | DivisibleBy 15 -> "FizzBuzz"
    | DivisibleBy 3  -> "Fizz"
    | DivisibleBy 5  -> "Buzz"
    | other -> string other

[1..15] |> List.map fizzBuzz
// ["1"; "2"; "Fizz"; "4"; "Buzz"; "Fizz";
//  "7"; "8"; "Fizz"; "Buzz"; "11";
//  "Fizz"; "13"; "14"; "FizzBuzz"]
```

💡 The active pattern `DivisibleBy 3` does not return any data. It's just the syntactic sugar equivalent of `if y |> isDivisibleBy 3` . In such a case, [F♯ 9](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-9#partial-active-patterns-can-return-bool-instead-of-unit-option) allows the Boolean to be returned directly, rather than having to go through the `Option` type:

```fsharp
let (|DivisibleBy|_|) factor number =
    number |> isDivisibleBy factor
```

### Alternative

```fsharp
let isDivisibleBy factor number =
    number % factor = 0

let boolToOption b =
    if b then Some () else None

let (|Fizz|_|) number = number |> isDivisibleBy 3 |> boolToOption
let (|Buzz|_|) number = number |> isDivisibleBy 5 |> boolToOption

let fizzBuzz = function
    | Fizz & Buzz -> "FizzBuzz"
    | Fizz -> "Fizz"
    | Buzz -> "Buzz"
    | other -> string other
```

* The 2 solutions are equal. It's a matter of style / personal taste.
* In F♯ 9, no need to do `|> boolToOption`.

## Active patterns use cases

1. Factor a guard _(see previous fizz buzz exercise)_
2. Wrapping a BCL method _(see `(|Regexp|_|)` and below)_.
3. Improve expressiveness, help to understand logic _(see below)_

```fsharp
[<RequireQualifiedAccess>]
module String =
    let (|Int|_|) (input: string) = // string -> int option
        match System.Int32.TryParse(input) with
        | true, i  -> Some i
        | false, _ -> None

let addOneOrZero = function
    | String.Int i -> i + 1
    | _ -> 0

let v1 = addOneOrZero "1"  // 2
let v2 = addOneOrZero "a"  // 0
```

## Expressiveness with active patterns

```fsharp
type Movie = { Title: string; Director: string; Year: int; Studio: string }

module Movie =
    let inline private satisfy ([<InlineIfLambda>] predicate) (movie: Movie) =
        match predicate movie with
        | true -> Some ()
        | false -> None

    let (|Director|_|) director = satisfy (fun movie -> movie.Director = director)
    let (|Studio|_|) studio = satisfy (fun movie -> movie.Studio = studio)
    let (|In|_|) year = satisfy (fun movie -> movie.Year = year)
    let (|Between|_|) min max = satisfy (fun { Year = year } -> year >= min && year <= max)

open Movie

let ``Is anime rated 10/10`` = function
    | Studio "Bones" & (Between 2001 2007 | In 2014)
    | Director "Hayao Miyazaki" -> true
    | _ -> false

let topAnimes =
    [ { Title = "Cowboy Bebop"; Director = "Shinichirō Watanabe"; Year = 2001; Studio = "Bones" }
      { Title = "Princess Mononoke"; Director = "Hayao Miyazaki"; Year = 1997; Studio = "Ghibli" } ]
    |> List.filter ``Is anime rated 10/10``
```

## Active pattern: first-class citizen

An active pattern ≃ function with metadata \
→ first-class citizen in F♯

```fsharp
// 1. Return an active pattern from a function
let (|Hayao_Miyazaki|_|) movie =
    (|Director|_|) "Hayao Miyazaki" movie

// 2. Take an active pattern as parameter -- A bit tricky
let firstItems (|Ok|_|) list =
    let rec loop values = function
        | Ok (item, rest) -> loop (item :: values) rest
        | _ -> List.rev values
    loop [] list

let (|Even|_|) = function
    | item :: rest when (item % 2) = 0 -> Some (item, rest)
    | _ -> None

let test = [0; 2; 4; 5; 6] |> firstItems (|Even|_|)  // [0; 2; 4]
```
