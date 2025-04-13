# Option type

A.k.a `Maybe` *(Haskell),* `Optional` *(Java 8)*

Models the absence of value \
→ In the sense of a possibility for a value to be absent \
→ ≠ `unit`: used in the case where there is never a value

Defined as a union with 2 *cases* :

```fsharp
type Option<'Value> =
    | None              // Case without data → when value is missing
    | Some of 'Value    // Case with data → when value is present
```

Common use cases:

- Modeling an optional field
- Turning a partial operation into a total operation

## Modeling an optional field

```fsharp
type Civility = Mr | Mrs
type User = { Name: string; Civility: Civility option }

let joey  = { Name = "Joey"; Civility = Some Mr }
let guest = { Name = "Guest"; Civility = None }
```

→ Make it explicit that `Name` is mandatory and `Civility` optional

☝ **Warning:** this design does not prevent `Name = null` here *(BCL limit)*

## Partial to total operation

- An operation is **partial** when no output value is possible for certain inputs.
- The operation can become **total** by wrapping the result in an `option`, `None` being used when the operation gives no output.

### Example 1: inverse of a number

```fsharp
let inverse n = 1.0 / n

let tryInverse n =
    match n with
    | 0.0 -> None
    | n   -> Some (1.0 / n)
```

| Function     | Operation | Signature               | `n = 0.5`  | `n = 0.0`    |
|--------------|-----------|-------------------------|------------|--------------|
| `inverse`    | Partial   | `float -> float`        | `2.0`      | `infinity` ❓ |
| `tryInverse` | Total     | `float -> float option` | `Some 2.0` | `None` 👌    |

### Example 2: find an element in a collection

- Partial operation: `find predicate` → 💥 when item not found
- Total operation: `tryFind predicate` → `None` or `Some item`

### Benefits 👍

- Explicit, honest / partial operation
  - No special value: `null`, `infinity`
  - No exception
- Forces calling code to handle all cases:
  - `Some value` → output value given
  - `None .....` → output value missing

## Control flow

How to test for the presence of the value *(of type `'T`)* in the option?

- ❌ Do not use `if option.IsSome then ... option.Value` pattern
- ✅ Do *pattern match* the option
- ✅ Do use `Option.xxx` functions

### Control flow with *pattern matching*

Example:

```fsharp
let print option =
    match option with
    | Some x -> printfn "%A" x
    | None   -> printfn "None"

print (Some 1.0)  // 1.0
print None        // None
```

### Control flow with `Option.xxx` helpers

*Mapping* of the inner value (of type `'T`) **if present**: \
→ `map f option` with `f` total operation `'T -> 'U` \
→ `bind f option` with `f` partial operation `'T -> 'U option`

Keep value **if present** and if conditions are met: \
→ `filter predicate option` with `predicate: 'T -> bool` called only if value present

#### Exercise

Implement `map`, `bind` and `filter` with *pattern matching*

<details>

<summary>Solution</summary>

```fsharp
let map f option =             // (f: 'T -> 'U) -> 'T option -> 'U option
    match option with
    | Some x -> Some (f x)
    | None   -> None           // 🎁 1. Why can't we write `None -> option`?

let bind f option =            // (f: 'T -> 'U option) -> 'T option -> 'U option
    match option with
    | Some x -> f x
    | None   -> None

let filter predicate option =  // (predicate: 'T -> bool) -> 'T option -> 'T option
    match option with
    | Some x when predicate x -> option
    | _ -> None                // 🎁 2. Implement `filter` with `bind`?
```

<details>

<summary>Bonus questions</summary>

```fsharp
// 🎁 1. Why can't we write `None -> option`?
let map (f: 'T -> 'U) (option: 'T option) : 'U option =
    match option with
    | Some x -> Some (f x)
    | None   -> (*None*) option  // 💥 Type error: `'U option` given != `'T option` expected
```

```fsharp
// 🎁 2. Implement `filter` with `bind`?
let filter predicate option =  // (predicate: 'T -> bool) -> 'T option -> 'T option
    option |> bind (fun x -> if predicate x then option else None)
```

</details>

</details>

#### Example

```fsharp
// Question/answer console application
type Answer = A | B | C | D

let tryParseAnswer =
    function
    | "A" -> Some A
    | "B" -> Some B
    | "C" -> Some C
    | "D" -> Some D
    | _   -> None

/// Called when the user types the answer on the keyboard
let checkAnswer (expectedAnswer: Answer) (givenAnswer: string) =
    tryParseAnswer givenAnswer
    |> Option.filter ((=) expectedAnswer)
    |> Option.map (fun _ -> "✅")
    |> Option.defaultValue "❌"

["X"; "A"; "B"] |> List.map (checkAnswer B)  // ["❌"; "❌"; "✅"]
```

#### Advantages

Makes business logic more readable

- No `if hasValue then / else`
- Highlight the *happy path*
- Handle corner cases at the end

💡 Alternative syntax more light: ad-hoc *computation expressions* 📍

## `Option`: comparison with other types

1. `Option` *vs* `List`
2. `Option` *vs* `Nullable`
3. `Option` *vs* `null`

### `Option` *vs* `List`

Conceptually closed \
→ Option ≃ List of 0 or 1 items \
→ See `Option.toList` function: `'t option -> 't list` (`None -> []`, `Some x -> [x]`)

💡 `Option` & `List` modules: many functions with the same name \
→ `contains`, `count`, `exist`, `filter`, `fold`, `forall`, `map`

☝ A `List` can have more than 1 element \
→ Type `Option` models absence of value better than type `List`

### `Option` *vs* `Nullable`

`System.Nullable<'T>` ≃ `Option<'T>` but more limited

- ❗ Does not work for reference types
- ❗ Lacks monadic behavior i.e. `map` and `bind` functions
- ❗ Lacks built-in pattern matching `Some x | None`
- ❗ In F♯, no magic as in C♯ / keyword `null`

**Example:**

```fsharp
open System

let x: Nullable<int> = null
// ⚠️ The type 'Nullable<int>' does not have 'null' as a proper value. To create a null value for a Nullable type use 'System.Nullable()' (FS43)

let x' = Nullable<int>()
// val x': Nullable<int> = <null>

let y = Nullable(1)
// val y: Nullable<int> = 1
```

👉 C♯ uses `Nullable` whereas F♯ uses only `Option`. However, `Nullable` can be required with some libraries, for instance to deal with nullable columns in a database.

### `Option` *vs* `null`

Due to the interop with the BCL, F♯ has to deal with `null` objects in some cases.

👉 **Good practice**: isolate these cases and wrap them in an `Option` type.

```fsharp
let readLine (reader: System.IO.TextReader) =
    reader.ReadLine() // Can return `null`
    |> Option.ofObj   // `null` becomes None

    // Same than:
    match reader.ReadLine() with
    | null -> None
    | line -> Some line
```

## Nullable reference types

F♯ 9 introduces nullable reference types: a type-safe way to deal with reference types that can have `null` as a valid value.

This feature must be activated:

- Adds `<Nullable>enable</Nullable>` in your `.fsproj`
- Passes `--checknulls+` options to `dotnet fsi` - see `FSharp.FSIExtraInteractiveParameters` settings in vscode

Then, `| null` needs to be added to the type annotation to indicate that `null` as a valid value. It's really similar to nullable reference types: F♯ `string | null` is equivalent to C♯ `string?`, with the usual tradeoff for explicitness over terseness/magic.

```fsharp
let notAValue: string | null = null

let isAValue: string | null = "hello world"

let isNotAValue2: string = null
// Warning FS3261: Nullness warning: The type 'string' does not support 'null'.
```

🔗 More details regarding this feature and how F♯ handles nullity (e.g. with `AllowNullLiteral` attribute): [Nullable Reference Types in F# 9](https://devblogs.microsoft.com/dotnet/nullable-reference-types-in-fsharp-9/).
