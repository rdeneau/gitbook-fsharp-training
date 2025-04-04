# Standard functions

The `FSharp.Core.dll` library is automatically imported into an F# project or into the FSI console. It provides standard operators and functions ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-operators.html)). Here are the main functions:

## Conversion

`box`, `tryUnbox`, `unbox` : _boxing_ and _unboxing_ (attempt)

```fsharp
let i = 123     // val i : int = 123
let o = box i   // val o : obj = 123

let i2: int = unbox o     // val i3 : int = 123
let i3 = unbox<int> o     // val i2 : int = 123

let ok = tryUnbox<int> o      // val ok : int option = Some 123
let ko = tryUnbox<string> o   // val ko : string option = None

unbox<float> o  //  💥 System.InvalidCastException
```

`byte`, `char`, `decimal`, `float`, `int`, `string` : to convert into `byte`, `char`, ...

```fsharp
let c_ko = char "ab"  // 💥 System.FormatException
let c1 = char "c"     // val c1 : char = 'c'
let c2 = char 32      // val c2 : char = ' '

let i_ko = int " "  // 💥 System.FormatException
let i1 = int "30"   // val i1 : int = 30
let i2 = int ' '    // val i2 : int = 32
let i3 = int 12.6   // val i3 : int = 12
```

`enum<'TEnum>` : conversion to the specified enum 📍 Composite types

## Reflection

`nameof`, `typeof` :

```fsharp
let myVariable = None
let myVariableName = nameof myVariable
// val myVariableName : string = "myVariable"

let t1 = typeof<string option>
// val t1 : System.Type = Microsoft.FSharp.Core.FSharpOption`1[System.String]

```

`typedefof` is interesting with a generic type:

```fsharp
let t2 = typedefof<string option>
// val t2 : System.Type = Microsoft.FSharp.Core.FSharpOption`1[T]

// t2 equivalent ways
let t2' = typedefof<_ option>
let t2'' = t1.GetGenericTypeDefinition ()

// t1 equivalent way
let t1' = t2.MakeGenericType(typeof<string>)
```

## Math

* `abs`, `sign` : absolute value, sign (-1 if < 0...)
* `(a)cos(h)`, `(a)sin`, `(a)tan` : (co)sine/tangent (inverse/hyperbolic)
* `ceil`, `floor`, `round` : rounding (lower, upper)
* `exp`, `log`, `log10` : exponential, logarithm...
* `pown x (n: int)` : power = `x` to the `n` power
* `sqrt` : square root

## Identity: `id`

Definition `let id x = x`

Signature : `(x: 'T) -> 'T` \
→ Single input parameter function \
→ Only returns this parameter

Why such a function ❓ \
→ Name `id` = abbreviation of `identity` \
→ Zero / Neutral element of function composition

<table><thead><tr><th>Operation</th><th width="186.33333333333331">Neutral element</th><th>Example</th></tr></thead><tbody><tr><td><code>+ </code> Addition</td><td><code>0</code></td><td><code>0 + 5</code> ≡ <code>5 + 0</code> ≡ <code>5</code></td></tr><tr><td><code>* </code> Multiplication</td><td><code>1</code></td><td><code>1 * 5</code> ≡ <code>5 * 1</code> ≡ <code>5</code></td></tr><tr><td><code>&gt;&gt;</code> Composition</td><td><code>id</code></td><td><code>id >> fn</code> ≡ <code>fn >> id</code> ≡ <code>fn</code></td></tr></tbody></table>

#### `id` use cases

With a _high-order function_ doing 2 things:

* 1 operation
* 1 value mapping via param `'T -> 'U`

E.g. `List.collect fn list` = flatten + mapping

How to do just the operation and not the mapping?

* `list |> List.collect (fun x -> x)` 👎
* `list |> List.collect id` 👍
* ☝ Best alternative : `List.concat list` 💯

## Others

* `compare a b : int`: returns -1 if a < b, 0 if =, 1 if >.
* `hash` : compute the hash (`HashCode`)
* `max`, `min` : maximum and minimum of 2 comparable values
* `ignore` : to "swallow" a value; returns `()`

## Extras

The following non-standard functions are commonly used, so recode them or use those in the `Prelude` module ([source](https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/Prelude.fs)) in the NuGet package [FSharpx.Extras](https://github.com/fsprojects/FSharpx.Extras).

### Flip

`flip`, `flip3` et `flip4` are used to reverse the order of a function's parameters, so that the last parameter becomes the first.

```fsharp
let inline flip f a b = f b a
let inline flip3 f a b c = f c a b
let inline flip4 f a b c d = f d a b c

// Example: defaultArg inversion
// defaultArg: (arg: option<'T> -> defaultValue: 'T -> 'T)
// defaultVal: (defaultValue: 'T -> arg: option<'T> -> 'T)
let inline defaultVal value option = defaultArg option value

// Comparison:
let withoutFlip    = Some 1 |> (fun x -> defaultArg x 0)
let withFlipInline = Some 1 |> (flip defaultArg 0)
let withFlipManual = Some 1 |> (defaultVal 0)
```

### Curry

* `curry` and `curry3` can be used to curry tuplified functions with 2 or 3 parameters.
* `uncurry` and `uncurry3` do the opposite.

```fsharp
let inline curry f a b = f (a, b)
let inline uncurry f (a, b) = f a b

let inline curry3 f a b c = f (a, b, c)
let inline uncurry3 f (a, b, c) = f a b c

// Example: DateTime
let dateIn2022Curry3 = 2022 |> curry3 System.DateTime // Piping is required because of the other 3-parameters constructor `DateTime(date: System.DateOnly, time: System.TimeOnly, kind: System.DateTimeKind)`
let dateIn2022Manual month day = System.DateTime(2022, month, day)
let d = dateIn2022Manual 1 31  // val d: System.DateTime = 31/01/2022 00:00:00
```

{% hint style="info" %}
## Comparison

* `dateIn2022Curry3: (int -> int -> System.DateTime)`
* `dateIn2022Manual: month: int -> day: int -> System.DateTime`

👉 `dateIn2022Manual` has a little longer implementation but is more easy to use thanks to named parameters.
{% endhint %}

### Konst

`konst` is a _sink_ function due to its second argument that is ignored in order to always return the first argument, the constant.

```fsharp
let inline konst a _ = a
```

**Example:** generate an always-true predicate, to pass to the higher-order function `filterMap`:

```fsharp
let inline filterMap predicate mapper list =
    list
    |> List.filter predicate
    |> List.map mapper

let onlyMappingF = [1..3] |> filterMap (fun _ -> true) ((+) 1)  // [2; 3; 4]
let onlyMappingK = [1..3] |> filterMap (konst true) ((+) 1)     // [2; 3; 4]
```

💡 `ignore` ≅ `konst ()`

### Tee

`tee` (also called `tap`) is used to apply a value to a function and return that value, ignoring any return from the function. This is useful with a side-effect function, for example to log an intermediate value in a pipeline:

```fsharp
let inline tee fn x = // fn: ('a -> 'b) -> x : 'a -> 'a
    fn x |> ignore
    x

// Example
let test =
    [1..10]
    |> List.map ((*) 3)
    |> tee (printfn "[Debug] After *3: %A")
    |> List.map ((+) 1)
    |> tee (printfn "[Debug] After +1: %A")
    |> List.filter (fun x -> x % 4 = 0)
// [Debug] After *3: [3; 6; 9; 12; 15; 18; 21; 24; 27; 30]
// [Debug] After +1: [4; 7; 10; 13; 16; 19; 22; 25; 28; 31]
// val test: int list = [4; 16; 28]
```

### Parse

The primitive types `Boolean`, `Byte`, `SByte`, `UInt16`, `Int16`, `UInt32`, `Int32`, `UInt64`, `Int64`, `Decimal`, `Single`, `Double`, `DateTime`, `DateTimeOffset` are extended with the static method `parse`, which encapsulates the classic .NET static method `TryParse`. The aim is to return an `Option` 📍 rather than a `bool` and an `out` variable.

```fsharp
// https://github.com/fsprojects/FSharpx.Extras/blob/master/src/FSharpx.Extras/Prelude.fs#L89

open System
open System.Globalization

let inline toOption x =
    match x with
    | true,  v -> Some v
    | false, _ -> None

type Int32 with
    static member parseWithOptions style provider (x: string) =
        Int32.TryParse(x, style, provider) |> toOption

    static member parse x =
        Int32.parseWithOptions NumberStyles.Integer CultureInfo.InvariantCulture x

// Usage examples:
let test1a = Int32.TryParse "1"  // val test1a : bool * int = (true, 1)
let test1b = Int32.parse "1"     // val test1b : int option = Some 1

let test2a = Int32.TryParse "z"  // val test2a : bool * int = (false, 0)
let test2b = Int32.parse "z"     // val test2b : int option = None
```

### String

The `String` module encapsulates string instance methods in functions pipeline-friendly. Here are a few examples:

```fsharp
open System

module String =
    let inline startsWith (value: string) (s: string) = s.StartsWith(value)
    let inline contains   (value: string) (s: string) = s.Contains(value)
    let inline endsWith   (value: string) (s: string) = s.EndsWith(value)

    let inline insert startIndex value (s: string) = s.Insert(startIndex, value)

    let inline padLeft  totalWidth (s: string) = s.PadLeft(totalWidth)
    let inline padRight totalWidth (s: string) = s.PadRight(totalWidth)
    let inline padLeft'  totalWidth paddingChar (s: string) = s.PadLeft(totalWidth, paddingChar)
    let inline padRight' totalWidth paddingChar (s: string) = s.PadRight(totalWidth, paddingChar)

    let inline replace (oldChar: char) (newChar: char) (s: string) = s.Replace(oldChar, newChar)
    let inline replace' (oldValue: string) (newValue: string) (s: string) = s.Replace(oldValue, newValue)

    let inline toLower (s: string) = s.ToLower()
    let inline toLowerInvariant (s: string) = s.ToLowerInvariant()
    let inline toUpper (s: string) = s.ToUpper()
    let inline toUpperInvariant (s: string) = s.ToUpperInvariant()

    let inline trim (s: string) = s.Trim()
```
