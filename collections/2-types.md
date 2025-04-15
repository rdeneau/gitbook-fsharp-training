# Types

## Type `List`

Implemented as a **simply chained list:**

- 1 list = 1 element _(Head)_ + 1 sub-list _(Tail)_
- Construction named _Cons_ and noted `::`

To avoid infinite recursion, we need an "exit" case: \
→ Empty list named _Empty_ and noted `[]`

👉 **Generic and recursive union type:**

```fsharp
type List<'T> =
  | ( [] )
  | ( :: ) of head: 'T * tail: List<'T>
```

☝️ **Note:** this syntax with cases as operator is only allowed in `FSharp.Core`.

### Type alias

- `List` _(big L)_ refers to the F♯ type or its companion module.
- `list` _(small l)_ is an alias of F♯'s `List` type, often used with OCaml notation: \
  → `let l : string list = ...`

⚠️ **Warnings:**

- ❗ After `open System.Collections.Generic`:
  - `List` is the C♯ mutable list, hiding the F♯ type!
  - The `List` F♯ companion module remains available → confusion!
- 💡 Use the `ResizeArray` alias to reference the C♯ list directly
  - no need for `open System.Collections.Generic` 👍

### Immutability

A `List` is **immutable**: \
→ It is not possible to modify an existing list.

Adding an element in the list: \
= Cheap operation with the _Cons_ operator (`::`) \
→ Creates a new list with: \
  • _Head_ = given element \
  • _Tail_ = existing list

🏷️ **Related concepts:**

- linked list
- recursive type

### Literals

| \# | Notation    | Equivalent          | Meaning (*)                         |
|----|-------------|---------------------|-------------------------------------|
| 0  | `[]`        | `[]`                | Empty                               |
| 1  | `[1]`       | `1 :: []`           | Cons (1, Empty)                     |
| 2  | `[2; 1]`    | `2 :: 1 :: []`      | Cons (2, Cons (1, Empty))           |
| 3  | `[3; 2; 1]` | `3 :: 2 :: 1 :: []` | Cons (3, Cons (2, Cons (1, Empty))) |

(*) We can verify it with [SharpLab.io](https://sharplab.io/#v2:DYLgZgzgPsCmAuACAbgBkQXkQbQLoFgAoOJZARkxzIOIRQCZLt6BuRaoklAZie7dbsaRIA==) :

```csharp
//...
v1@2 = FSharpList<int>.Cons(1, FSharpList<int>.Empty);
v2@3 = FSharpList<int>.Cons(2, FSharpList<int>.Cons(1, FSharpList<int>.Empty));
//...
```
### Initialisation

```fsharp
// Range: Start..End (Step=1)
let numFromOneToFive = [1..5]     // [1; 2; 3; 4; 5]

// Range: Start..Step..End
let oddFromOneToNine = [1..2..9]  // [1; 3; 5; 7; 9]

// Comprehension
let pairsWithDistinctItems =
    [ for i in [1..3] do
      for j in [1..3] do
        if i <> j then
            i, j ]
// [(1; 2); (1; 3); (2, 1); (2, 3); (3, 1); (3, 2)]
```

### Exercices 🕹️

**1.** Implement the `rev` function \
→ Inverts a list: `rev [1; 2; 3]` ≡ `[3; 2; 1]`

**2.** Implement the `map` function \
→ Transforms each element: `[1; 2; 3] |> map ((+) 1)` ≡ `[2; 3; 4]`

💡 **Hints**
   → Use empty list `[]` or _Cons_ `head :: tail` patterns
   → Write a recursive function

<details>

<summary>Solution</summary>

```fsharp
let rev list =
    let rec loop acc rest =
        match rest with
        | [] -> acc
        | x :: xs -> loop (x :: acc) xs
    loop [] list

let map f list =
    let rec loop acc rest =
        match rest with
        | [] -> acc
        | x :: xs -> loop (f x :: acc) xs
    list |> loop [] |> rev
```

</details>

We can verify that it's working in FSI console:

```fsharp
let (=!) actual expected =
    if actual = expected
    then printfn $"✅ {actual}"
    else printfn $"❌ {actual} != {expected}"

[1..3] |> rev =! [3; 2; 1];;
// ✅ [3; 2; 1]

[1..3] |> map ((+) 1) =! [2; 3; 4];;
// ✅ [2; 3; 4]
```

💡 **Bonus:** verify the tail recursion with [sharplab.io](https://sharplab.io)

## Type `Array`

Signature: `'T array` _(recommended)_ or `'T[]` or `'T []`

Main differences compared to the `List`:

- Fixed-size
- Fat square brackets `[| |]` for literals
- Mutable ❗
- Access by index in `O(1)` 👍

```fsharp
// Literal
[| 1; 2; 3; 4; 5 |]  // val it : int [] = [|1; 2; 3; 4; 5|]

// Range
[| 1 .. 5 |] = [| 1; 2; 3; 4; 5 |]  // true
[| 1 .. 3 .. 10 |] = [| 1; 4; 7; 10 |] // true

// Comprehension
[| for i in 1 .. 5 -> i, i * 2 |]
// [|(1, 2); (2, 4); (3, 6); (4, 8); (5, 10)|]

// Slicing: sub-array between the given `(start)..(end)` indices
let names =    [|"0: Alice"; "1: Jim"; "2: Rachel"; "3: Sophia"; "4: Tony"|]

names[1..3] // [|            "1: Jim"; "2: Rachel"; "3: Sophia"           |]
names[2..]  // [|                      "2: Rachel"; "3: Sophia"; "4: Tony"|]
names[..3]  // [|"0: Alice"; "1: Jim"; "2: Rachel"; "3: Sophia"           |]

// Mutation
let names = [| "Juliet"; "Tony" |]
names[1] <- "Bob"
names;;  // [| "Juliet"; "Bob" |]
```

💡 Slicing works also with `string`: `"012345"[1..3]` ≡ `"123"`

## Alias `ResizeArray`

Alias for BCL `System.Collections.Generic.List<T>`

```fsharp
let rev items = items |> Seq.rev |> ResizeArray
let initial = ResizeArray [ 1..5 ]
let reversed = rev initial // ResizeArray [ 5..-1..0 ]
```

**Advantages** 👍

- No need for `open System.Collections.Generic`
- No name conflicts on `List`

**Notes** ☝️

- Do not confuse the alias `ResizeArray` with the `Array` F♯ type.
- `ResizeArray` is in F♯ a better name for the BCL generic `List<T>`
  - Closer semantically and in usages to an array than a list

## Type `Seq`

**Definition:** Series of elements of the same type

`'t seq` ≡ `Seq<'T>` ≡ `IEnumerable<'T>`

**Lazy:** sequence built gradually as it is iterated
≠ All other collections built entirely from their declaration

### Syntax

`seq { items | range | comprehension }`

```fsharp
seq { yield 1; yield 2 }   // 'yield' explicit 😕
seq { 1; 2; 3; 5; 8; 13 }  // 'yield' omitted 👍

// Range
seq { 1 .. 10 }       // seq [1; 2; 3; 4; ...]
seq { 1 .. 2 .. 10 }  // seq [1; 3; 5; 7; ...]

// Comprehension
seq { for i in 1 .. 5 do i, i * 2 }
// seq [(1, 2); (2, 4); (3, 6); (4, 8); ...]
```

### Infinite sequence

2 options to write an infinite sequence

- Use `Seq.initInfinite` function
- Write a recursive function to generate the sequence

#### Option 1

`Seq.initInfinite` function

- Signature: `(initializer: (index: int) -> 'T) -> seq<'T>`
- Parameter: `initializer` is used to create the specified index element (>= 0)

```fsharp
let seqOfSquares = Seq.initInfinite (fun i -> i * i)

seqOfSquares |> Seq.take 5 |> List.ofSeq;;
// val it: int list = [0; 1; 4; 9; 16]
```

#### Option 2

Recursive function to generate the sequence

```fsharp
[<TailCall>]
let rec private squaresStartingAt n =
    seq {
        yield n * n
        yield! squaresStartingAt (n + 1) // 🔄️
    }

let squares = squaresStartingAt 0

squares |> Seq.take 10 |> List.ofSeq;;
// val it: int list = [0; 1; 4; 9; 16; 25; 36; 49; 64; 81]
```

## Type `Set`

- Self-ordering collection of unique elements _(without duplicates)_
- Implemented as a binary tree

```fsharp
// Construct
set [ 2; 9; 4; 2 ]          // set [2; 4; 9]  // ☝ Only one '2' in the set
Set.ofArray [| 1; 3 |]      // set [1; 3]
Set.ofList [ 1; 3 ]         // set [1; 3]
seq { 1; 3 } |> Set.ofSeq   // set [1; 3]

// Add/remove element
Set.empty         // set []
|> Set.add 2      // set [2]
|> Set.remove 9   // set [2]    // ☝ No exception
|> Set.add 9      // set [2; 9]
|> Set.remove 9   // set [2]
```

### Informations

→ `count`, `minElement`, `maxElement`

```fsharp
let oneToFive = set [1..5]          // set [1; 2; 3; 4; 5]

// Number of elements: `Count` property or `Set.count` function - ⚠️ O(N)
// ☝ Do not confuse with `Xxx.length` for Array, List, Seq
let nb = Set.count oneToFive // 5

// Element min, max
let min = oneToFive |> Set.minElement   // 1
let max = oneToFive |> Set.maxElement   // 5
```

### Operations

|   | Operation    | Operator | Function for 2 sets  | Function for N sets  |
|---|--------------|----------|----------------------|----------------------|
| ⊖ | Difference   | `-`      | `Set.difference`     | `Set.differenceMany` |
| ∪ | Union        | `+`      | `Set.union`          | `Set.unionMany`      |
| ∩ | Intersection | ×        | `Set.intersect`      | `Set.intersectMany`  |

**Examples:**

```txt
| Union             | Difference        | Intersection      |
|-------------------|-------------------|-------------------|
|   [ 1 2 3 4 5   ] |   [ 1 2 3 4 5   ] |   [ 1 2 3 4 5   ] |
| + [   2   4   6 ] | - [   2   4   6 ] | ∩ [   2   4   6 ] |
| = [ 1 2 3 4 5 6 ] | = [ 1   3   5   ] | = [   2   4     ] |
```

## Type `Map`

Associative array { _Key_ → _Value_ } ≃ C♯ immutable dictionary

```fsharp
// Construct: from collection of (key, val) pairs
// → `Map.ofXxx` function • Xxx = Array, List, Seq
let map1 = seq { (2, "A"); (1, "B") } |> Map.ofSeq
// → `Map(tuples)` constructor
let map2 = Map [ (2, "A"); (1, "B"); (3, "C"); (3, "D") ]
// map [(1, "B"); (2, "A"); (3, "D")]
// 👉 Ordered by key (1, 2, 3) and deduplicated in last win - see '(3, "D")'

// Add/remove entry
Map.empty         // map []
|> Map.add 2 "A"  // map [(2, "A")]
|> Map.remove 5   // map [(2, "A")] // ☝ No exception if key not found
|> Map.add 9 "B"  // map [(2, "A"); (9, "B")]
|> Map.remove 2   // map [(9, "B")]
```

### Access/Lookup by key

```fsharp
let table = Map [ ("A", "Abc"); ("G", "Ghi"); ("Z", "Zzz") ]

// Indexer by key
table["A"];;  // val it: string = "Abc"
table["-"];;  // 💣 KeyNotFoundException

// `Map.find`: return the matching value or 💣 if the key is not found
table |> Map.find "G";; // val it: string = "Ghi"

// `Map.tryFind`: return the matching value in an option
table |> Map.tryFind "Z";;  // val it: string option = Some "Zzz"
table |> Map.tryFind "-";;  // val it: string option = None
```

## Dictionnaires

### `dict` function

- Builds an `IDictionary<'k, 'v>` from a sequence of key/value pairs
- The interface is error prone: the dictionary is **immutable** ❗

```fsharp
let table = dict [ (1, 100); (2, 200) ]
// val table: System.Collections.Generic.IDictionary<int,int>

table[1];;          // val it: int = 100

table[99];;         // 💣 KeyNotFoundException

table[1] <- 111;;   // 💣 NotSupportedException: This value cannot be mutated
table.Add(3, 300);; // 💣 NotSupportedException: This value cannot be mutated
```

### `readOnlyDict` function

- Builds an `IReadOnlyDictionary<'k, 'v>` from a sequence of key/value pairs
- The interface is honest: the dictionary is **immutable**

```fsharp
let table = readOnlyDict [ (1, 100); (2, 200) ]
// val table: System.Collections.Generic.IReadOnlyDictionary<int,int>

table[1];;          // val it: int = 100

table[99];;         // 💣 KeyNotFoundException

do table[1] <- 111;;
// ~~~~~~~~ 💥 Error FS0810: Property 'Item' cannot be set

do table.Add(3, 300);;
//       ~~~ 💥 Error FS0039:
// The type 'IReadOnlyDictionary<_,_>' does not define the field, constructor or member 'Add'.
```

### Recommendation

`dict` returns an object that does not implement fully `IDictionary<'k, 'v>` \
→ Violate the Liskov's substitution principle❗

`readOnlyDict` returns an object that respects `IReadOnlyDictionary<'k, 'v>`

👉 Prefer `readOnlyDict` to `dict` when possible

### `KeyValue` active pattern

Used to deconstruct a `KeyValuePair` dictionary entry to a `(key, value)` pair

```fsharp
// FSharp.Core / prim-types.fs#4983
let (|KeyValue|) (kvp: KeyValuePair<'k, 'v>) : 'k * 'v =
    kvp.Key, kvp.Value

let table =
    readOnlyDict
        [ (1, 100)
          (2, 200)
          (3, 300) ]

// Parcourt du dictionnaire
for kv in table do // kv: KeyValuePair<int,int>
    printfn $"{kv.Key}, {kv.Value}"

// De manière + élégante avec l'active pattern
for KeyValue (key, value) in table do
    printfn $"{key}, {value}"
```

## Lookup performance

🔗 [High Performance Collections in F♯](https://www.compositional-it.com/news-blog/high-performance-hash-tables-for-f/), Compositional IT, Jan 2021

### `Map` vs `Dictionary`

Fonction `readOnlyDict` permet de créer rapidement un `IReadOnlyDictionary` → à partir d'une séquence de tuples `key, item` → très performant : 10x plus rapide que `Map` pour le _lookup_

### `Dictionary` vs `Array`

→ `Array` suffit si peu de lookups (< 100) et peu d'éléments (< 100) \
→ `Dictionary` sinon

## `Map` and `Set` _vs_ `IComparable`

Only work if elements (of a `Set`) or keys (of a `Map`) are **comparable**!

Examples:

```fsharp
// Classes are not comparable by default, so you cannot use them in a set or a map
type NameClass(name: string) =
    member val Value = name

let namesClass = set [NameClass("Alice"); NameClass("Bob")]
//                    ~~~~~~~~~~~~~~~~~~
// 💥 Error FS0193: The type 'NameClass' does not support the 'comparison' constraint.
//     For example, it does not support the 'System.IComparable' interface

// We can use a struct instead
[<Struct>]
type NameStruct(name: string) =
    member this.Name = name

let namesStruct = set [NameStruct("Alice"); NameStruct("Bob")]

// or a F# functional type: tuple, record, union
type Name = Name of string

let names = set [Name "Alice"; Name "Bob"]
```

Last option for classes: we implement `IComparable` _(but not `IComparable<'T>`)_
