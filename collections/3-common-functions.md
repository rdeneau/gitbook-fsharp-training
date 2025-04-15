# Common functions

Functions available in different modules
→ Customized for the target type

Operations: access, construct, find, select, aggregate...

**Convention** used here:

1. Functions are given by their name
   - To use them, we need to qualify them by the module.
2. The last parameter is omitted for brevity
   - It's always the collection.

**Example:** `head` _vs_ `List.head x`

## Access to an element

| ↓ Access \ Return → | `'T` or 💣    | `'T option`     |
|---------------------|---------------|-----------------|
| By index            | `list[index]` |                 |
| By index            | `item index`  | `tryItem index` |
| First element       | `head`        | `tryHead`       |
| Last element        | `last`        | `tryLast`       |

💣 `ArgumentException` or `IndexOutOfRangeException`

```fsharp
[1; 2] |> List.tryHead    // Some 1
[1; 2] |> List.tryItem 2  // None
```

### Cost ⚠️

| Function \ Module | `Array` | `List` | `Seq`  |
|-------------------|---------|--------|--------|
| `head`            | O(1)    | O(1)   | O(1)   |
| `item`            | O(1)    | O(n) ❗ | O(n) ❗ |
| `last`            | O(1)    | O(n) ❗ | O(n) ❗ |
| `length`          | O(1)    | O(n) ❗ | O(n) ❗ |

## Combine collections

| Function       | Parameters                      | Final size         |
|----------------|---------------------------------|--------------------|
| `append` / `@` | 2 collections of sizes N1 et N2 | N1 + N2            |
| `concat`       | K collections of sizes N1..Nk   | N1 + N2 + ... + Nk |
| `zip`          | 2 collections of same size N ❗ | N pairs            |
| `allPairs`     | 2 collections of sizes N1 et N2 | N1 * N2 pairs      |

```fsharp
List.concat [ [1]; [2; 3] ];;  // [1; 2; 3]
List.append [1;2;3] [4;5;6];;  // [1; 2; 3; 4; 5; 6]

// @ operator: alias of `List.append` only • not working with Array, Seq
[1;2;3] @ [4;5;6];;            // [1; 2; 3; 4; 5; 6]

List.zip      [1; 2] ['a'; 'b'];;  // [(1, 'a'); (2, 'b')]
List.allPairs [1; 2] ['a'; 'b'];;  // [(1, 'a'); (1, 'b'); (2, 'a'); (2, 'b')]
```

## Find an element

Using a predicate `f : 'T -> bool`:

| Which element \ Returns | `'T` or 💣      | `'T option`        |
|-------------------------|-----------------|--------------------|
| First found             | `find`          | `tryFind`          |
| Last found              | `findBack`      | `tryFindBack`      |
| Index of first found    | `findIndex`     | `tryFindIndex`     |
| Index of last found     | `findIndexBack` | `tryFindIndexBack` |

```fsharp
[1; 2] |> List.find (fun x -> x < 2)      // 1
[1; 2] |> List.tryFind (fun x -> x >= 2)  // Some 2
[1; 2] |> List.tryFind (fun x -> x > 2)   // None
```

## Search elements

| Search           | How many items | Function         |
|------------------|----------------|------------------|
| By value         | At least 1     | `contains value` |
| By predicate `f` | At least 1     | `exists f`       |
| "                | All            | `forall f`       |

```fsharp
[1; 2] |> List.contains 0      // false
[1; 2] |> List.contains 1      // true
[1; 2] |> List.exists (fun x -> x >= 2)  // true
[1; 2] |> List.forall (fun x -> x >= 2)  // false
```

## Select elements

| What elements   | By size      | By predicate `f` |
|-----------------|--------------|------------------|
| All those found |              | `filter f`       |
| First ignored   | `skip n`     | `skipWhile f`    |
| First found     | `take n`     | `takeWhile f`    |
|                 | `truncate n` |                  |

☝ **Notes:**

- `skip`, `take` _vs_ `truncate` when `n` > collection's size
  - `skip`, `take`: 💣 exception
  - `truncate`: empty collections w/o exception
- Alternative for `Array`: _Range_ `arr[2..5]`

## Map elements

Functions taking as input:

- A mapping function `f` (a.k.a. _mapper_)
- A collection of elements of type `'T`

| Function  | Mapping `f`              | Returns     | How many elements?           |
|-----------|--------------------------|-------------|------------------------------|
| `map`     | `       'T -> 'U`        | `'U list`   | As many in than out          |
| `mapi`    | `int -> 'T -> 'U`        | `'U list`   | As many in than out          |
| `collect` | `       'T -> 'U list`   | `'U list`   | *flatMap*                    |
| `choose`  | `       'T -> 'U option` | `'U list`   | Less                         |
| `pick`    | `       'T -> 'U option` | `'U`        | 1 (the first matching) or 💣 |
| `tryPick` | `       'T -> 'U option` | `'U option` | 1 (the first matching)       |

### `map` _vs_ `mapi`

`mapi` ≡ `map` _with index_

The difference is on the `f` mapping parameter:

- `map`: `'T -> 'U`
- `mapi`: `int -> 'T -> 'U` → the additional `int` parameter is the item index

```fsharp
["A"; "B"]
|> List.mapi (fun i x -> $"{i+1}. {x}")
// ["1. A"; "2. B"]
```

### Alternative to `mapi`

Apart from `map` and `iter`, no `xxx` function has a `xxxi` variant.

💡 Use `indexed` to obtain elements with their index

```fsharp
let isOk (i, x) = i >= 1 && x <= "C"

["A"; "B"; "C"; "D"]
|> List.indexed       // [ (0, "A"); (1, "B"); (2, "C"); (3, "D") ]
|> List.filter isOk   //           [ (1, "B"); (2, "C") ]
|> List.map snd       //               [ "B" ; "C" ]
```

### `map` _vs_ `iter`

`iter` looks like `map` with \
• no mapping: `'T -> unit` _vs_ `'T -> 'U` \
• no output: `unit` _vs_ `'U list`

But `iter` is in fact used for a different use case: \
→ to trigger an action, a side-effect, for each item

Example: print the item to the console

```fsharp
["A"; "B"; "C"] |> List.iteri (fun i x -> printfn $"Item #{i}: {x}")
// Item #0: A
// Item #1: B
// Item #2: C
```

### `choose`, `pick`, `tryPick`

Signatures:

- `choose  : mapper: ('a -> 'b option) -> list: 'a list -> 'b list`
- `pick    : mapper: ('a -> 'b option) -> list: 'a list -> 'b`
- `tryPick : mapper: ('a -> 'b option) -> list: 'a list -> 'b option`

The mapping may return `None` for some items not mappable (or just ignored)

Different use cases:

- `choose` to get all the mappable items mapped
- `pick` or `tryPick` to get the first one

When no items are mappable:

- `choose` returns an empty collection
- `tryPick` returns `None`
- `pick` raises a `KeyNotFoundException` 💣

**Examples:**

```fsharp
let tryParseInt (s: string) =
    match System.Int32.TryParse(s) with
    | true,  i -> Some i
    | false, _ -> None

["1"; "2"; "?"] |> List.choose tryParseInt   // [1; 2]
["1"; "2"; "?"] |> List.pick tryParseInt     // 1
["1"; "2"; "?"] |> List.tryPick tryParseInt  // Some 1
```

**Analogies:**

`choose f` ≃ \
• `collect (f >> Option.to{Collection})` \
• `(filter (f >> Option.isSome)) >> (map (f >> Option.value))`

`(try)pick f` ≃ \
• `(try)find(f >> Option.isSome)) >> f` \
• `choose f >> (try)head`

## Aggregate

### Versatile functions

`fold       (f: 'U -> 'T -> 'U) (seed: 'U) list` \
`foldBack   (f: 'T -> 'U -> 'U) list (seed: 'U)` \
`reduce     (f: 'T -> 'T -> 'T) list` \
`reduceBack (f: 'T -> 'T -> 'T) list`

folder `f` takes 2 parameters: an "accumulator" `acc` and the current element `x`

`xxxBack` _vs_ `xxx`:

- Iterates from last to first element
- Parameters `seed` and `list` reversed (for `foldBack`)
- Folder `f` parameters reversed (`x acc`)

`reduceXxx` _vs_ `foldXxx`:

- `reduceXxx` uses the first item as the _seed_ and performs no mapping
- `reduceXxx` fails if the list is empty 💥

Examples:

```fsharp
["a";"b";"c"] |> List.reduce (+)  // "abc"
[ 1; 2; 3 ] |> List.reduce ( * )  // 6

[1;2;3;4] |> List.reduce     (fun acc x -> 10 * acc + x)  // 1234
[1;2;3;4] |> List.reduceBack (fun x acc -> 10 * acc + x)  // 4321

("all:", [1;2;3;4]) ||> List.fold     (fun acc x -> $"{acc}{x}")  // "all:1234"
([1;2;3;4], "rev:") ||> List.foldBack (fun x acc -> $"{acc}{x}")  // "rev:4321"
```

### Specialized functions

| Direct    | With mapping |
|-----------|--------------|
| `max`     | `maxBy`      |
| `min`     | `minBy`      |
| `sum`     | `sumBy`      |
| `average` | `averageBy`  |
| `length`  | `countBy`    |

`xxxBy f` ≃ `map f >> xxx`

```fsharp
[1; 2; 3] |> List.max  // 3

[ (1,"a"); (2,"b"); (3,"c") ] |> List.sumBy fst  // 6

["abc";"ab";"c";"a";"bc";"a";"b";"c"] |> List.countBy id
// [("abc", 1); ("ab", 1); ("c", 2); ("a", 2); ("bc", 1); ("b", 1)]
```

### `sum`: type constraints

The `sum` functions only work if the type of elements in the collection includes:
• a static `Zero` member
• an overload of the `+` operator

```fsharp
type Point = Point of X:int * Y:int with
    static member Zero = Point (0, 0)
    static member (+) (Point (ax, ay), Point (bx, by)) = Point (ax + bx, ay + by)

let addition = (Point (1, 1)) + (Point (2, 2))
// val addition : Point = Point (3, 3)

let sum = [1..3] |> List.sumBy (fun i -> Point (i, -i))
// val sum : Point = Point (6, -6)
```

## Change the order of elements

| Operation   | Direct                   | Mapping                   |
|-------------|--------------------------|---------------------------|
| Inversion   | `rev list`               | ×                         |
| Sort asc    | `sort list`              | `sortBy f list`           |
| Sort desc   | `sortDescending list`    | `sortDescendingBy f list` |
| Sort custom | `sortWith comparer list` | ×                         |

```fsharp
[1..5] |> List.rev // [5; 4; 3; 2; 1]
[2; 4; 1; 3; 5] |> List.sort // [1..5]
["b1"; "c3"; "a2"] |> List.sortBy (fun x -> x[0]) // ["a2"; "b1"; "c3"] because a < b < c
["b1"; "c3"; "a2"] |> List.sortBy (fun x -> x[1]) // ["b1"; "a2"; "c3"] because 1 < 2 < 3
```

## Separate

💡 Elements are divided into groups.

| Operation       | Result _(`;` omitted)_                       | Remark                |
|-----------------|----------------------------------------------|-----------------------|
| `[1..10]`       | `[ 1   2   3   4   5   6   7   8   9   10 ]` | `length = 10`         |
| `chunkBySize 3` | `[[1   2   3] [4   5   6] [7   8   9] [10]]` | `forall: length <= 3` |
| `splitInto 3`   | `[[1   2   3   4] [5   6   7] [8   9   10]]` | `length <= 3`         |
| `splitAt 3`     | `([1   2   3],[4   5   6   7   8   9   10])` | Tuple ❗              |

## Group items

### By size

💡 Items can be **duplicated** into different groups.

| Operation    | Result _(`'` and `;` omitted)_         | Remark                     |
|--------------|----------------------------------------|----------------------------|
| `[1..5]`     | `[ 1       2       3       4      5 ]` |                            |
| `pairwise`   | `[(1,2)   (2,3)   (3,4)   (4,5)]`      | Tuple ❗                   |
| `windowed 2` | `[[1 2]   [2 3]   [3 4]   [4 5]]`      | Array of arrays of 2 items |
| `windowed 3` | `[[1 2 3] [2 3 4] [3 4 5]]`            | Array of arrays of 3 items |

### By criteria

| Operation   | Criteria                 | Result                                |
|-------------|--------------------------|---------------------------------------|
| `partition` | `predicate:  'T -> bool` | `('T list * 'T list)`                 |
|             |                          | → 1 pair `([OKs], [KOs])`             |
| `groupBy`   | `projection: 'T -> 'K`   | `('K * 'T list) list`                 |
|             |                          | → N tuples `[(key, [related items])]` |

```fsharp
let isOdd i = (i % 2 = 1)
[1..10] |> List.partition isOdd // (        [1; 3; 5; 7; 9] ,         [2; 4; 6; 8; 10]  )
[1..10] |> List.groupBy isOdd   // [ (true, [1; 3; 5; 7; 9]); (false, [2; 4; 6; 8; 10]) ]

let firstLetter (s: string) = s.[0]
["apple"; "alice"; "bob"; "carrot"] |> List.groupBy firstLetter
// [('a', ["apple"; "alice"]); ('b', ["bob"]); ('c', ["carrot"])]
```

## Change collection type

Your choice: `Dest.ofSource` or `Source.toDest`

| From \\ to | `Array`        | `List`         | `Seq`         |
|------------|----------------|----------------|---------------|
| `Array`    | ×              | `List.ofArray` | `Seq.ofArray` |
|            | ×              | `Array.toList` | `Array.toSeq` |
| `List`     | `Array.ofList` | ×              | `Seq.ofList`  |
|            | `List.toArray` | ×              | `List.toSeq`  |
| `Seq`      | `Array.ofSeq`  | `List.ofSeq`   | ×             |
|            | `Seq.toArray`  | `Seq.toList`   | ×             |

## Functions _vs_ comprehension

The functions of `List`/`Array`/`Seq` can often be replaced by a comprehension, more versatile:

```fsharp
let list = [ 0..99 ]

list |> List.map f                   <->  [ for x in list do f x ]
list |> List.filter p                <->  [ for x in list do if p x then x ]
list |> List.filter p |> List.map f  <->  [ for x in list do if p x then f x ]
list |> List.collect g               <->  [ for x in list do yield! g x ]
```

## Additional resources

Documentation FSharp.Core 👍

- [Array module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-arraymodule.html)
- [List module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html)
- [Map module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-mapmodule.html)
- [Seq module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-seqmodule.html)
- [Set module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-setmodule.html)
