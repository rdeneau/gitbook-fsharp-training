# Dedicated functions

## `List` module

_Cons_ `1 :: [2; 3]`

- Item added to top of list
- List appears in reverse order 😕
- But operation efficient: in **O(1)** _(`Tail` preserved)_ 👍

_Append_ `[1] @ [2; 3]`

- List in normal order
- But operation in **O(n)** ❗ _(New `Tail` at each iteration)_

## `Map` module

### `change`

Signature : `Map.change key (f: 'T option -> 'T option) table`

Depending on the `f` function passed as an argument, we can:
→ Add, modify or delete the element of a given key

| Key       | Input        | `f` returns `None`       | `f` returns `Some newVal`    |
|-----------|--------------|--------------------------|------------------------------|
| -         | -            | ≡ `Map.remove key table` | ≡ `Map.add key newVal table` |
| Found     | `Some value` | Remove the entry         | Change the value to _newVal_ |
| Not found | `None`       | Ignore this key          | Add the item _(key, newVal)_ |

### `containsKey` _vs_ `exists` _vs_ `filter`

```txt
Function      Signature                        Comment                                                   
------------+--------------------------------+-----------------------------------------------------------
containsKey   'K -> Map<'K,'V> -> bool         Indicates whether the key is present                      
exists         f -> Map<'K,'V> -> bool         Indicates whether a key/value pair satisfies the predicate
filter         f -> Map<'K,'V> -> Map<'K,'V>   Keeps key/value pairs satisfying the predicate            

With predicate f: 'K -> 'V -> bool
```

```fsharp
let table = Map [ (2, "A"); (1, "B"); (3, "D") ]

table |> Map.containsKey 0  // false
table |> Map.containsKey 2  // true

let isEven i = i % 2 = 0
let isFigure (s: string) = "AEIOUY".Contains(s)

table |> Map.exists (fun k v -> (isEven k) && (isFigure v))  // true
table |> Map.filter (fun k v -> (isEven k) && (isFigure v))  // map [(2, "A")]
```

## `Seq` Module

### `Seq.cache`

As a sequence is lazy, it's reconstructed each time it's iterated. This reconstruction can be costly. An algorithm that iterates (even partially) an invariant sequence several times can be optimized by caching the sequence using the `Seq.cache` function.

Signature : `Seq.cache: source: 'T seq -> 'T seq`

Caching is optimized by being deferred and performed only on the elements iterated.

## `string` type

To manipulate the characters in a string, several options are available.

### `Seq` module

The `string` type implements `Seq<char>`. \
→ We can therefore use the functions of the `Seq` module.

### `Array` module

`ToCharArray()` method returns the characters of a string as a `char array`.
→ We can therefore use the functions of the `Array` module.

### `String` module

There is also a `String` module (from `FSharp.Core`) offering a number of interesting functions that are individually more powerful than those of `Seq` and `Array`:

```fsharp
String.concat (separator: string) (strings: seq<string>) : string
String.init (count: int) (f: (index: int) -> string) : string
String.replicate (count: int) (s: string) : string

String.exists (predicate: char -> bool) (s: string) : bool
String.forall (predicate: char -> bool) (s: string) : bool
String.filter (predicate: char -> bool) (s: string) : string

String.collect (mapping:        char -> string) (s: string) : string
String.map     (mapping:        char -> char)   (s: string) : string
String.mapi    (mapping: int -> char -> char)   (s: string) : string
// Idem iter/iteri which returns unit
```

**Examples:**

```fsharp
let a = String.concat "-" ["a"; "b"; "c"]  // "a-b-c"
let b = String.init 3 (fun i -> $"#{i}")   // "#0#1#2"
let c = String.replicate 3 "0"             // "000"

let d = "abcd" |> String.exists (fun c -> c >= 'b')  // true
let e = "abcd" |> String.forall (fun c -> c >= 'b')  // false
let f = "abcd" |> String.filter (fun c -> c >= 'b')  // "bcd"

let g = "abcd" |> String.collect (fun c -> $"{c}{c}")  // "aabbccdd"

let h = "abcd" |> String.map (fun c -> (int c) + 1 |> char)  // "bcde"
```

☝️ **Note:** after `open System`, `String` stands for 3 things that the compiler is able to figure them out:

- `(new) String(...)` constructors
- `String.` gives access to all functions of the `String` module (in _camelCase_)...
- ... and static methods of `System.String` (in _PascalCase_)

## Array2D

Instead of working with N-level nested collections, F# offers multidimensional arrays (also called matrixes). However, to create them, there's no specific syntax: you have to use a create function. Let's take a look at 2-dimensional arrays.

### Type

The type of a 2-dimensional array is `'t[,]` but IntelliSense sometimes just gives `'t array` which is less precise, no longer indicating the number of dimensions.

### Construction

`array2D` creates a 2-dimensional array from a collection of collections all of the same length.

`Array2D.init` generates an array by specifying: \
→ Its length according to the 2 dimensions N and P \
→ A function generating the element at the two specified indexes.

```fsharp
let rows = [1..3]
let cols = ['A'..'C']

let cell row col = $"%c{col}%i{row}"
let rowCells row = cols |> List.map (cell row)

let list = rows |> List.map rowCells
// val arr: string list list =
//   [["A1"; "B1"; "C1"]; ["A2"; "B2"; "C2"]; ["A3"; "B3"; "C3"]]

let matrix = array2D list
// val matrix: string[,] = [["A1"; "B1"; "C1"]
//                          ["A2"; "B2"; "C2"]
//                          ["A3"; "B3"; "C3"]]

let matrix' = Array2D.init 3 3 (fun i j -> cell i cols[j])
```

### Access by indexes

```fsharp
let a1  = list[0][0]    // "A1" // arr.[0].[0] before F# 6
let a1' = matrix[0,0]  // "A1" // matrix.[0,0] before F# 6
```

### Lengths

`Array2D.length1` and `Array2D.length2` return the number of rows and columns.

```fsharp
let rowCount =
    list.Length,
    list |> List.length,
    matrix |> Array2D.length1
// val rowCount: int * int * int = (3, 3, 3)

let columnCount =
    list[0] |> List.length,
    matrix |> Array2D.length2
// val columnCount: int * int = (3, 3)
```

### Slicing

Syntax for taking only one horizontal and/or vertical slice, using the `..` operator and the wilcard `*` character to take all indexes.

```fsharp
let m1 = matrix[1.., *]  // Remove first row: A2..C3
let m2 = matrix[0..1, *] // Remove last row : A1..C2
let m3 = matrix[*, 1..]  // Remove first column: B1..C3
let m4 = matrix[*, 0]    // First column: [|"A1"; "A2"; "A3"|]
                         // 💡 1 dimension array (i.e. vector)
```

This feature offers a lot of flexibility. As with a collection of collections, a matrix allows access by row (`matrix[row, *]` vs `list[row]`). A matrix also allows direct access by column (`matrix[*, col]`), which is not possible with a collection of collections without first transposing it - see `transpose` function ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html#transpose)).

### Other use cases

```fsharp
// Mapping with indexes
let doubleNotations =
    matrix
    |> Array2D.mapi (fun row col value -> $"{value} => [R{row+1}C{col+1}]")
//   [["A1 => [R1C1]"; "B1 => [R1C2]"; "C1 => [R1C3]"]
//    ["A2 => [R2C1]"; "B2 => [R2C2]"; "C2 => [R2C3]"]
//    ["A3 => [R3C1]"; "B3 => [R3C2]"; "C3 => [R3C3]"]]

```

Other functions: \
🔗 [https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-array2dmodule.html](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-array2dmodule.html)
