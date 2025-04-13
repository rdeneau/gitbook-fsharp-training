# Patterns

{% hint style="info" %}
### Definition

Patterns are rules for detecting input data structure.
{% endhint %}

Used extensively in F♯

- *match expression*, *let binding*, parameters deconstruction...
- Very practical for manipulating F♯ algebraic types (tuple, record, union)
- Composable: supports multiple nesting levels
- Composed by logical AND/OR
- Supports literals: `1.0`, `"test"`...

## Wildcard Pattern

Represented by `_`, alone or combined with another *pattern*.

Always true \
→ To be placed last in a *match expression*.

⚠️ Always seek 1st to handle all cases exhaustively/explicitly
     When impossible, then use `_`

```fsharp
match option with
| Some 1 -> ...
| _ -> ...              // ⚠️ Non exhaustive

match option with
| Some 1 -> ...
| Some _ | None -> ...  // 👌 More exhaustive
```

{% hint style="warning" %}
### Recommendation

- Always seek first to handle all cases exhaustively/explicitly
- When it's impossible (by combination explosion), too ugly to read or too boring to write
- Then use `_` to discard all other cases

{% endhint %}

## Constant Pattern

Detects constants, `null` and number literals, `char`, `string`, `enum`.

```fsharp
[<Literal>]
let Three = 3   // Constant

let is123 num = // int -> bool
    match num with
    | 1 | 2 | Three -> true
    | _ -> false
```

☝ **Notes :**

- The `Three` pattern is also classified as an *Identifier Pattern* 📍
- For `null` matching, we also talk about *Null Pattern*.

## Variable Pattern

Assigns the detected value to a "variable" for subsequent use

Example: `b` variable below

```fsharp
let isInt (s: string) =
    match System.Int32.TryParse(s) with
    | b, _ -> b
```

⚠️ You cannot link to the same variable more than once.

```fsharp
let elementsAreEqualKo tuple =
    match tuple with
    | x, x -> true  // 💥 Error FS0038: x' is linked twice in this model
    | _, _ -> false
```

💡 **Solution:** use 2 variables then check their equality

```fsharp
// 1. Guard (`when`) 📍
let elementsAreEqual = function
    | x, y when x = y -> true
    | _, _ -> false

// 2. Deconstruction: even simpler!
let elementsAreEqual' (x, y) =
    x = y
```

## Identifier Pattern

Detects *cases* of a union type and their possible contents

```fsharp
type PersonName =
    | FirstOnly of string
    | LastOnly of string
    | FirstLast of string * string

let classify personName =
    match personName with
    | LastOnly _ -> "Last name only"
    | FirstOnly _ -> "First name only"
    | FirstLast _ -> "First and last names"
```

The identifier pattern can be combined with both constant and variable patterns. When the cases fields have labels, we can pattern match on them too:

```fsharp
type Shape =
    | Circle of radius: int
    | Rectangle of height: int * width: int

let describe shape =
    match shape with
    | Circle radius -> $"Circle ∅ %i{2*radius}"

    // 1. "Anonymous" pattern of the field as tuple
    // ⚠️ All elements must be matched, hence the `_` to discard the `width`
    | Rectangle(0, _)

    // 2. Match on a single field by its name
    | Rectangle(width = 0) -> "Flat rectangle"

    // 3. Match on several fields by name
    // ⚠️ We use `;` to separate fields, like for a record, even though the union fields are closer to tuples.
    | Rectangle(width = w; height = h) -> $"Rectangle %i{w} × %i{h}"

```

## OR / AND Patterns

Combine two patterns *(named `P1` and `P2` below)*:

- `P1 | P2` → P1 or P2 - E.g. `Rectangle (0, _) | Rectangle (_, 0)`
- `P1 & P2` → P1 and P2 - used especially with *active patterns* 📍

```fsharp
type Upload = { Filename: string; Title: string option }

let titleOrFile ({ Title = Some name } | { Filename = name }) = name

titleOrFile { Filename = "Report.docx"; Title = None }            // Report.docx
titleOrFile { Filename = "Report.docx"; Title = Some "Report+" }  // "Report+"
```

☝️ **Explanations:**

- The `Title` is optional. The first pattern `{ Title = Some name }` will match only if the `Title` is specified.
- The second pattern `{ Filename = name }` will always works. It's written to extract the `Filename` into the same variable `name` as the first pattern.

{% hint style="warning" %}
### Warning

This code is a bit convoluted, meant for demo purpose, too "smart" for maintenable code.
{% endhint %}

## Alias Pattern

`as` is used to name an element whose content is deconstructed

```fsharp
let (x, y) as coordinate = (1, 2)
printfn "%i %i %A" x y coordinate  // 1 2 (1, 2)
```

💡 Also works within functions to get back the parameter name, `person` below:

```fsharp
type Person = { Name: string; Age: int }

let acceptMajorOnly ({ Age = age } as person) = // person: Person -> Person option
    if age < 18 then None else Some person
```

💡 The AND pattern can be used as an alternative to the alias pattern. \
☝️ However, it won't work to get the parameter name:

```fsharp
type Person = { Name: string; Age: int }

let acceptMajorOnly' ({ Age = age } & person) = // Person -> Person option
    if age < 18 then None else Some person
```

## Parenthesized Pattern

Purpose: Use of parentheses `()` to group patterns

Common use case: tackle precedence

```fsharp
type Shape = Circle of Radius: int | Square of Side: int

let countFlatShapes shapes =
    let rec loop rest count =
        match rest with
        | (Square(Side = 0) | (Circle(Radius = 0))) :: tail -> loop tail (count + 1) // ①
        | _ :: tail -> loop tail count
        | [] -> count
    loop shapes 0
```

☝ Line ① would compile without doing `() :: tail` \
⚠️ Parentheses complicate reading \
💡 Try to do without when possible:

```fsharp
// 1. Repeat `tail` variable
let countFlatShapes shapes =
    let rec loop rest count =
        match rest with
        | Circle(Radius = 0) :: tail
        | Square(Side = 0) :: tail -> loop tail (count + 1)
        // [...]

// 2. Extract isFlatShape function
let isFlat =
    function
    | Circle(Radius = 0)
    | Square(Side = 0) -> true
    | _ -> false

let countFlatShapes shapes =
    let rec loop rest count =
        match rest with
        | shape :: tail when shape |> isFlat -> loop tail (count + 1)
        // [...]
```

## Construction Patterns

Use type construction syntax to deconstruct a type

- Cons and List Patterns
- Array Pattern
- Tuple Pattern
- Record Pattern

### Cons and List Patterns

≃ Inverses of the 2 ways to construct a list

*Cons Pattern* : `head :: tail` → decomposes a list *(with >= 1 element)* into:

- *Head*: 1st element
- *Tail*: another list with the remaining elements - can be empty

*List Pattern* : `[items]` → decompose a list into 0..N items

- `[]` : empty list
- `[x]` : list with 1 element set in the `x` variable
- `[x; y]` : list with 2 elements set in variables `x` and `y`
- `[_; _]`: list with 2 elements ignored

💡 `x :: []` ≡ `[x]`, `x :: y :: []` ≡ `[x; y]`...

The default *match expression* combines the 2 patterns: \
→ A list is either empty `[]`, or composed of an item and the rest: `head :: tail`

**Example:** \
→ Recursive functions traversing a list \
→ The `[]` pattern is used to stop recursion:

```fsharp
[<TailCall>]
let rec printList l =
    match l with
    | head :: tail ->
        printf "%d " head
        printList tail
    | [] -> printfn ""
```

### Array Pattern

Syntax: `[| items |]` for 0..N items between `;`

```fsharp
let length vector =
    match vector with
    | [| x |] -> x
    | [| x; y |] -> sqrt (x*x + y*y)
    | [| x; y; z |] -> sqrt (x*x + y*y + z*z)
    | _ -> invalidArg (nameof vector) $"Vector with more than 4 dimensions not supported"
```

☝ There is no pattern for sequences, as they are *"lazy "*.

### Tuple Pattern

Syntax: `items` or `(items)` for 2..N items between `,`.

💡 Useful to match several values at the same time

```fsharp
type Color = Red | Blue
type Style = Background | Text

let css color style =
    match color, style with
    | Red, Background -> "background-color: red"
    | Red, Text -> "color: red"
    | Blue, Background -> "background-color: blue"
    | Blue, Text -> "color: blue"
```

### Record Pattern

Syntax: `{ Field1 = var1; ... }`

- It's not required to specify all fields in the record
- In case of ambiguity on the record type, qualify the field: `Record.Field`

💡 Also works for function parameters:

```fsharp
type Person = { Name: string; Age: int }

let displayMajority { Age = age; Name = name } =
    if age >= 18
    then printfn "%s is major" name
    else printfn "%s is minor" name

let john = { Name = "John"; Age = 25 }
displayMajority john // John is major
```

⚠️ **Reminder:** there is no pattern for anonymous *Records*!

```fsharp
type Person = { Name: string; Age: int }

let john = { Name = "John"; Age = 25 }
let { Name = name } = john  // 👌 val name : string = "John"

let john' = {| john with Civility = "Mister" |}
let {| Name = name' |} = john'  // 💥
```

## Type Test Pattern

Syntax: `my-object :? sub-type` and returns a `bool` \
→ ≃ `my-object is sub-type` in C♯

Usage: with a type hierarchy

```fsharp
open System.Windows.Forms

let RegisterControl (control: Control) =
    match control with
    | :? Button as button -> button.Text <- "Registered."
    | :? CheckBox as checkbox -> checkbox.Text <- "Registered."
    | :? Windows -> invalidArg (nameof control) "Window cannot be registered"
    | _ -> ()
```

☝️ **Note:** the `:?` operator is not designed to check evidence, which can be confusing. \
→ Example: *(from [stackoverflow](https://stackoverflow.com/q/70845434/8634147))*

```fsharp
type Car = interface end

type Mercedes() =
    interface Car

let benz = Mercedes()

let t1 = benz :? Mercedes
//       ~~~~~~~~~~~~~~~~ ⚠️
// Warning FS0067: This type test or downcast will always hold

let t2 = benz :? Car
//       ~~~~~~~~~~~ 💥
// Error FS0193: Type constraint mismatch. The type 'Car' is not compatible with type 'Mercedes'

let t3 = box benz :? Car
// val t3: bool = true
```

### `try`/`with` block

This pattern is common in `try`/`with` blocks:

```fsharp
try
    printfn "Difference: %i" (42 / 0)
with
| :? DivideByZeroException as x -> 
    printfn "Fail! %s" x.Message
| :? TimeoutException -> 
    printfn "Fail! Took too long"
```

### Boxing

The *Type Test Pattern* only works with reference types. \
→ For a value type or unknown type, it must be boxed.

```fsharp
let isIntKo = function :? int -> true | _ -> false
//                     ~~~~~~
// 💥 Error FS0008: This runtime coercion or type test from type 'a to int
//    involves an indeterminate type based on information prior to this program point.

let isInt x =
    match box x with
    | :? int -> true
    | _ -> false
```
