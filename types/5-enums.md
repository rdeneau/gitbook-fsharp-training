# Enums

→ Real .NET `enum`!

## Declaration

Set of integer constants (`byte`, `int`...)

```fsharp
type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3
```

☝️ Note the syntax difference with a enum-like union:

```fsharp
type Color = Red | Green | Blue
```

### Underlying type

- Unlike C♯, there is no default underlying type in F♯.
- The underlying type is defined by means of literals defining member values:
  - `1`, `2`, `3` → `int`
  - `1uy`, `2uy`, `3uy` → `byte`
  - Etc. - see [Literals](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/literals)

Corollary: you must use the same type for all members, otherwise it won't compile!

```fsharp
type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3uy
// 💥         ~~~
// This expression was expected to have type 'int' but here has type 'byte'
```

### Char

Le type `char` peut être utilisé comme type sous-jacent :

```fsharp
type AnswerChar = Yes='Y' | No='N'

// L'équivalent ne marche pas avec 'string'
type AnswerChar = Yes="Y" | No="N"
// 💥 ~~~~~~~~~~ Littéraux énumérés doivent être de type 'int'...
```

### Casse

Autre différence avec les types union, les membres peuvent être en camelCase :

```fsharp
type File =
    | a = 'a'
    | b = 'b'
    | c = 'c'
```

## Usages

⚠️ Unlike unions, the use of an `enum` literal is necessarily **qualified**

```fs
type AnswerChar = Yes='Y' | No='N'

let answerKo = Yes  // 💥 Error FS0039
//             ~~~     The value or constructor 'Yes' is not defined.

let answer = AnswerChar.Yes   // 👌 OK
```

💡 We can force the qualification for union types too:

```fs
[<RequireQualifiedAccess>] // 👈
type Color = Red | Green | Blue
```

### Conversion `int <-> enum`

```fs
let redValue = int ColorN.Red         // enum -> int
let redAgain = enum<ColorN> redValue  // int -> enum
let red: ColorN = enum redValue       // int -> enum

// ⚠️ Does not work with char enum
let ko = char AnswerChar.No   // 💥 Error FS0001
let no: AnswerChar = enum 'N' // 💥 Error FS0001
```

## Pattern matching

⚠️ Unlike unions, *pattern matching* on enums is **not exhaustive**
→ See `Warning FS0104: Enums may take values outside known cases...`

```fs
type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3

let toHex color =
    match color with
    | ColorN.Red   -> "#FF0000"
    | ColorN.Green -> "#00FF00"
    | ColorN.Blue  -> "#0000FF"
    | _ -> invalidArg (nameof color) $"Color {color} not supported" // 👈
```

## Flags

Même principe qu'en C♯ où l'on choisit comme valeurs des multiples de 2 afin de pouvoir les combiner :

```fsharp
open System

[<Flags>]
type PermissionFlags =
    | Read    = 1
    | Write   = 2
    | Execute = 4

let permissions = PermissionFlags.Read ||| PermissionFlags.Write
// val permissions: PermissionFlags = Read, Write

let canRead    = permissions.HasFlag PermissionFlags.Read    // true
let canWrite   = permissions.HasFlag PermissionFlags.Write   // true
let canExecute = permissions.HasFlag PermissionFlags.Execute // false
```

💡 Note the `|||` operator called "binary OR" (same as `|` in C♯)

💡 **Hint:** use binary notation for flag values:

```fsharp
[<Flags>]
type PermissionFlags =
    | Read    = 0b001
    | Write   = 0b010
    | Execute = 0b100
```

## Values

`System.Enum.GetValues()` returns the list of members of an `enum` \
⚠️ Weakly typed: `Array` (non-generic array) \
💡 Use a helper like:

```fsharp
let enumValues<'a> () =
    Enum.GetValues(typeof<'a>)
    :?> ('a array)
    |> Array.toList

let allPermissions = enumValues<PermissionFlags>()
// val allPermissions: PermissionFlags list = [Read; Write; Execute]
```

## Conversion

`enum<'enum>`: converts an integer value to the specified `'enum`

```fsharp
type AnswerNum =
    | Yes = 1
    | No  = 0

let y1 = enum<AnswerNum> 1  // val y1 : AnswerNum = Yes
let y2: AnswerNum = enum 0  // val y2 : AnswerNum = No
```

`int` allows reverse conversion to retrieve the underlying value of an enum member

```fsharp
let yesNum = int AnswerNum.Yes  // val yesNum : int = 1
```

For enums whose underlying type is `char` \
→ `enum`, `int` and `char` functions do not work:

```fsharp
type AnswerChar =
    | Yes = 'Y'
    | No  = 'N'

let no_ko: AnswerChar = enum 'N' // 💥 Le type 'int32' ne correspond pas au type 'char'

let y1_ko = int AnswerChar.Yes   // 💥 Le type 'AnswerChar' ne prend pas en charge une conversion vers le type 'int'
let y2_ko = char AnswerChar.Yes  // 💥 Le type 'AnswerChar' ne prend pas en charge une conversion vers le type 'char'
```

You must use the `LanguagePrimitives` module:

```fsharp
let no_ok: AnswerChar = LanguagePrimitives.EnumOfValue 'N' // val no_ok : AnswerChar = No

let y1_ok = LanguagePrimitives.EnumToValue AnswerChar.Yes // val y1_ok : char = 'Y'
let y2_ok = unbox<char> AnswerChar.Yes  // val y2_ok : char = 'Y'
```

## Enum *vs* Union

| Type  | Data inside | Qualification | Exhaustivity |
|-------|-------------|---------------|--------------|
| Enum  | integers    | Required      | ❌ No         |
| Union | any         | Optional      | ✅ Yes        |

☝ **Recommendation:**

- Prefer Union over Enum in most cases
- Choose an Enum for:
  - .NET Interop
  - int data
  - Flags feature

## Extras

💡 NuGet package [FSharpx.Extras](https://github.com/fsprojects/FSharpx.Extras) \
   → Includes an `Enum` module with these helpers:

- `parse<'enum>: string -> 'enum`
- `tryParse<'enum>: string -> 'enum option`
- `getValues<'enum>: unit -> 'enum seq`

```fsharp
#r "nuget: FSharpx.Extras"
open FSharpx

type ColorN =
    | Red   = 1
    | Green = 2
    | Blue  = 3

let red = "Red" |> Enum.tryParse<ColorN> // val red: ColorN option = Some Red
let none = "xx" |> Enum.tryParse<ColorN> // val none: ColorN option = None

let blue: ColorN = "Blue" |> Enum.parse // val blue: ColorN = Blue
let ko =
    try
        "Ko" |> Enum.parse<ColorN>
    with ex ->
        printfn "💥 %s %s" (ex.GetType().FullName) ex.Message
        // 💥 System.ArgumentException: Requested value 'Ko' was not found
        enum<ColorN> 0

let colors = Enum.getValues<ColorN> () |> Seq.toList
// val colors: ColorN list = [Red; Green; Blue]
```
