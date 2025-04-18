---
description: >-
  Functions available in multiple modules, each version customized for its
  target type
---

# Common functions

Operations: access, construct, find, select, aggregate...

**☝️ Convention** used here:

1. Functions are given by their name
   * To use them, we need to qualify them by the module.
2. The last parameter is omitted for brevity
   * It's always the collection.

**Example:** `head` _vs_ `List.head x`

## Access to an element

| ↓ Access \ Return element → | Directly or 💣 | Optionally      |
| --------------------------- | -------------- | --------------- |
| By index                    | `x[index]`     |                 |
| By index                    | `item index`   | `tryItem index` |
| First element               | `head`         | `tryHead`       |
| Last element                | `last`         | `tryLast`       |

💣 `ArgumentException` or `IndexOutOfRangeException`

```fsharp
[1; 2] |> List.tryHead    // Some 1
[1; 2] |> List.tryItem 2  // None
```

### Cost ⚠️

| ↓ Function \ Module → | Array | List   | Seq    |
| --------------------- | ----- | ------ | ------ |
| `head`                | O(1)  | O(1)   | O(1)   |
| `item`                | O(1)  | O(n) ❗ | O(n) ❗ |
| `last`                | O(1)  | O(n) ❗ | O(n) ❗ |
| `length`              | O(1)  | O(n) ❗ | O(n) ❗ |

## Combine collections

<table><thead><tr><th width="141">Function</th><th>Parameters</th><th>Final size</th></tr></thead><tbody><tr><td><code>append</code> / <code>@</code></td><td>2 collections of sizes N1 et N2</td><td>N1 + N2</td></tr><tr><td><code>concat</code></td><td>K collections of sizes N1..Nk</td><td>N1 + N2 + ... + Nk</td></tr><tr><td><code>zip</code></td><td>2 collections of same size N ❗</td><td>N pairs</td></tr><tr><td><code>allPairs</code></td><td>2 collections of sizes N1 et N2</td><td>N1 * N2 pairs</td></tr></tbody></table>

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

<table><thead><tr><th width="273">↓ Which element \ Returns →</th><th>Direct or 💣</th><th>Optional</th></tr></thead><tbody><tr><td>First found</td><td><code>find</code></td><td><code>tryFind</code></td></tr><tr><td>Last found</td><td><code>findBack</code></td><td><code>tryFindBack</code></td></tr><tr><td>Index of first found</td><td><code>findIndex</code></td><td><code>tryFindIndex</code></td></tr><tr><td>Index of last found</td><td><code>findIndexBack</code></td><td><code>tryFindIndexBack</code></td></tr></tbody></table>

```fsharp
[1; 2] |> List.find (fun x -> x < 2)      // 1
[1; 2] |> List.tryFind (fun x -> x >= 2)  // Some 2
[1; 2] |> List.tryFind (fun x -> x > 2)   // None
```

## Search elements

| Search           | How many items | Function         |
| ---------------- | -------------- | ---------------- |
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

| ↓ Which elements \ Search → | By size      | By predicate  |
| --------------------------- | ------------ | ------------- |
| All those found             |              | `filter f`    |
| First ignored               | `skip n`     | `skipWhile f` |
| First found                 | `take n`     | `takeWhile f` |
|                             | `truncate n` |               |

☝ **Notes:**

* `skip`, `take` _vs_ `truncate` when `n` > collection's size
  * `skip`, `take`: 💣 exception
  * `truncate`: empty collections w/o exception
* Alternative for `Array`: _Range_ `arr[2..5]`

## Map elements

Functions taking as input:

* A mapping function `f` (a.k.a. _mapper_)
* A collection of type `~~~~ 'T` with `~~~~` meaning `Array`, `List`, or `Seq`

<table><thead><tr><th width="111">Function</th><th>Mapping f</th><th width="113">Returns</th><th>How many elements?</th></tr></thead><tbody><tr><td><code>map</code></td><td>       <code>'T -> 'U</code>      </td><td><code>'U ~~~~</code></td><td>As many in than out</td></tr><tr><td><code>mapi</code></td><td><code>int -> 'T -> 'U</code>      </td><td><code>'U ~~~~</code></td><td>As many in than out</td></tr><tr><td><code>collect</code></td><td>       <code>'T -> 'U ~~~~</code> </td><td><code>'U ~~~~</code></td><td><em>flatMap</em></td></tr><tr><td><code>choose</code></td><td>       <code>'T -> 'U option</code></td><td><code>'U ~~~~</code></td><td>Less</td></tr><tr><td><code>pick</code></td><td>       <code>'T -> 'U option</code></td><td><code>'U</code>     </td><td>1 <em>(the first matching)</em> or 💣</td></tr><tr><td><code>tryPick</code></td><td>       <code>'T -> 'U option</code></td><td><code>'U option</code></td><td>1 <em>(the first matching)</em></td></tr></tbody></table>

### `map` _vs_ `mapi`

`mapi` ≡ `map` _with index_

The difference is on the `f` mapping parameter:

* `map`: `'T -> 'U`
* `mapi`: `int -> 'T -> 'U` → the additional `int` parameter is the item index

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

`iter` looks like `map` with

* no mapping: `'T -> unit` _vs_ `'T -> 'U`
* no output: `unit` _vs_ `'U list`

But `iter` is used for a different use case:\
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

• `choose  : mapper: ('T -> 'U option) -> items: 'T ~~~~ -> 'U ~~~~`\
• `pick    : mapper: ('T -> 'U option) -> items: 'T ~~~~ -> 'U`\
• `tryPick : mapper: ('T -> 'U option) -> items: 'T ~~~~ -> 'U option`

Notes:

* The mapper may return `None` for some items not mappable (or just ignored)
* `~~~~` stands for the collection type: `Array`, `List`, or `Seq`

Different use cases:

* `choose` to get all the mappable items mapped
* `pick` or `tryPick` to get the first one

When no items are mappable:

* `choose` returns an empty collection
* `tryPick` returns `None`
* `pick` raises a `KeyNotFoundException` 💣

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

`choose f` ≃\
• `collect (f >> Option.to{Collection})`\
• `(filter (f >> Option.isSome)) >> (map (f >> Option.value))`

`(try)pick f` ≃\
• `(try)find(f >> Option.isSome)) >> f`\
• `choose f >> (try)head`

## Aggregate

### Specialized aggregate functions

<table><thead><tr><th>Direct</th><th>By projection</th><th data-type="checkbox">Mapping</th><th>Constraint</th></tr></thead><tbody><tr><td>×</td><td><code>countBy</code></td><td>false</td><td>×</td></tr><tr><td><code>max</code></td><td><code>maxBy</code></td><td>false</td><td><code>comparison</code></td></tr><tr><td><code>min</code></td><td><code>minBy</code></td><td>false</td><td><code>comparison</code></td></tr><tr><td><code>sum</code></td><td><code>sumBy</code></td><td>true</td><td><a data-footnote-ref href="#user-content-fn-1"><em>Monoid</em></a>📍</td></tr><tr><td><code>average</code></td><td><code>averageBy</code></td><td>true</td><td><a data-footnote-ref href="#user-content-fn-1"><em>Monoid</em></a>📍</td></tr></tbody></table>

{% hint style="warning" %}
#### Warning

💣 `ArgumentException` if the collection is empty.
{% endhint %}

### CountBy

Uses a projection `f: 'T -> 'U` to compute a "key" for each item\
Returns all the keys with the number of items having this key

```fsharp
let words = [ "hello"; "world"; "fsharp"; "is"; "awesome" ]
let wordCountByLength = words |> List.countBy String.length |> List.sortBy fst
//val wordCountByLength: (int * int) list = [(2, 1); (5, 2); (6, 1); (7, 1)]
```

💡 Useful to determine duplicates:

```fsharp
let findDuplicates items =
    items
    |> List.countBy id
    |> List.choose (fun (item, count) -> if count > 1 then Some item else None)

let t = findDuplicates [1; 2; 3; 4; 5; 1; 2; 3]
// val t: int list = [1; 2; 3]
```

### Max(By), Min(By)

```fsharp
type N = One | Two | Three | Four | Five

let max = List.max [ One; Two; Three; Four; Five ]
// val max: N = Five

let maxText = List.maxBy string [ One; Two; Three; Four; Five ]
// val maxText: N = Two
```

☝️ **Notes:**

* Unions are `IComparable` by default\
  → `N` follows the `comparison` constraint.
* `maxBy` and `minBy` perform no mapping: \
  → See the returned value in the example: `Two` (`N`), ≠ `"Two"` (`string`)

### Sum(By), Average(By)

```fsharp
let sumKO = List.sum [ (1,"a"); (2,"b"); (3,"c") ]
//                      ~~~~~ 💥 Error FS0001
// Expecting a type supporting the operator 'get_Zero' but given a tuple type

let sum = List.sumBy fst [ (1,"a"); (2,"b"); (3,"c") ]
// val sum: int = 6 
```

☝️ **Notes:**

* The error `FS0001` at line 1 reveals the _Monoid_ constraint _(see below)_ that must be satisfied by:
  * The item type `'T` for `sum` and `average`,
  * The projection return type `'U` for `sumBy` and `averageBy`.
* `sumBy`  and `averageBy` perform a mapping\
  → See the returned type in the example: `int` ≠ item type: `int * string`
* `sum` ≡ `sumBy id`\
  `average` ≡ `averageBy id`

### Monoid constraint

To satisfy the monoid constraint, a type must have:

* A `Zero` static read-only property
* An overload of the `+` operator

Example of a custom type with these members:

```fsharp
type Point = Point of X:int * Y:int with
    static member Zero = Point (0, 0)
    static member (+) (Point (ax, ay), Point (bx, by)) = Point (ax + bx, ay + by)

let addition = (Point (1, 1)) + (Point (2, 2))
// val addition : Point = Point (3, 3)

let sum = [1..3] |> List.sumBy (fun i -> Point (i, -i))
// val sum : Point = Point (6, -6)
```

### Versatile aggregate functions

• `fold       (f: 'U -> 'T -> 'U) (seed: 'U) items`\
• `foldBack   (f: 'T -> 'U -> 'U) items (seed: 'U)`\
• `reduce     (f: 'T -> 'T -> 'T) items`\
• `reduceBack (f: 'T -> 'T -> 'T) items`

`f` takes 2 parameters: an "accumulator" `acc` and the current element `x`\
→ For `foldBack`, the parameters are in a reversed order: `x acc`

`xxxBack` _vs_ `xxx`:

* Iterates from last to first element
* Parameters `seed` and `items` reversed (for `foldBack`)

`reduce(Back)` _vs_ `fold(Back)`:

* `reduce(Back)` uses the first item as the _seed_ and performs no mapping _(see `'T -> 'T` vs `'T -> 'U`)_
* `reduce(Back)` fails if the collection is empty 💣

Examples:

```fsharp
["a";"b";"c"] |> List.reduce (+)  // "abc"
[ 1; 2; 3 ] |> List.reduce ( * )  // 6

[1;2;3;4] |> List.reduce     (fun acc x -> 10 * acc + x)  // 1234
[1;2;3;4] |> List.reduceBack (fun x acc -> 10 * acc + x)  // 4321

("all:", [1;2;3;4]) ||> List.fold     (fun acc x -> $"{acc}{x}")  // "all:1234"
([1;2;3;4], "rev:") ||> List.foldBack (fun x acc -> $"{acc}{x}")  // "rev:4321"
```

### Fold(Back) versatility

We could write almost all functions with `fold` or `foldBack` _(performance aside)_

```fsharp
// collect function using fold
let collect f list = List.fold (fun acc x -> acc @ f x) [] list

let test1 = [1; 2; 3] |> collect (fun x -> [x; x])  // [1; 1; 2; 2; 3; 3]

// filter function using foldBack
let filter f list = List.foldBack (fun x acc -> if f x then x :: acc else acc) list []

let test2 = [1; 2; 3; 4; 5] |> filter ((=) 2)  // [2]

// map function using foldBack
let map f list = List.foldBack (fun x acc -> f x :: acc) list []

let test3 = [1; 2; 3; 4; 5] |> map (~-)  // [-1; -2; -3; -4; -5]
```

## Change the order of elements

| Operation   | Direct                   | Mapping                   |
| ----------- | ------------------------ | ------------------------- |
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

💡 Elements are divided into groups

<table><thead><tr><th width="156">Operation</th><th width="385">Result (; omitted)</th><th>Remark</th></tr></thead><tbody><tr><td><code>[1..10]</code></td><td><code>[ 1 2 3 4 5 6 7 8 9 10 ]</code></td><td><code>length = 10</code></td></tr><tr><td><code>chunkBySize 3</code></td><td><code>[[1 2 3] [4 5 6] [7 8 9] [10]]</code></td><td><code>forall: length &#x3C;= 3</code></td></tr><tr><td><code>splitInto 3</code></td><td><code>[[1 2 3 4] [5 6 7] [8 9 10]]</code></td><td><code>length &#x3C;= 3</code></td></tr><tr><td><code>splitAt 3</code></td><td><code>([1 2 3],[4 5 6 7 8 9 10])</code></td><td>Tuple ❗</td></tr></tbody></table>

☝️ Notice how `splitInto n` distributes the items equally, from left to right:

```fsharp
let a7  = [1..7]  |> List.splitInto 3  // [[1; 2; 3]; [4; 5]; [6; 7]]
let a8  = [1..8]  |> List.splitInto 3  // [[1; 2; 3]; [4; 5; 6]; [7; 8]]
let a9  = [1..9]  |> List.splitInto 3  // [[1; 2; 3]; [4; 5; 6]; [7; 8; 9]]
let a10 = [1..10] |> List.splitInto 3  // [[1; 2; 3; 4]; [5; 6; 7]; [8; 9; 10]]
```

```
Size in  →  Sizes out
7        →  3  2  2
8        →  3  3  2
9        →  3  3  3
10       →  4  3  3
```

## Group items

### By size

💡 Items can be **duplicated** into different groups

<table><thead><tr><th width="149">Operation</th><th width="305">Result (' and ; omitted)</th><th>Remark</th></tr></thead><tbody><tr><td><code>[1..5]</code></td><td><code>[ 1 2 3 4 5 ]</code></td><td></td></tr><tr><td><code>pairwise</code></td><td><code>[(1,2) (2,3) (3,4) (4,5)]</code></td><td>Tuple ❗</td></tr><tr><td><code>windowed 2</code></td><td><code>[[1 2] [2 3] [3 4] [4 5]]</code></td><td>Array of arrays of 2 items</td></tr><tr><td><code>windowed 3</code></td><td><code>[[1 2 3] [2 3 4] [3 4 5]]</code></td><td>Array of arrays of 3 items</td></tr></tbody></table>

### By criteria

| Operation   | Criteria                | Result                                |
| ----------- | ----------------------- | ------------------------------------- |
| `partition` | `predicate: 'T -> bool` | `('T list * 'T list)`                 |
|             |                         | → 1 pair `([OKs], [KOs])`             |
| `groupBy`   | `projection: 'T -> 'K`  | `('K * 'T list) list`                 |
|             |                         | → N tuples `[(key, [related items])]` |

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

| From \ to | `Array`        | `List`         | `Seq`         |
| --------- | -------------- | -------------- | ------------- |
| `Array`   | ×              | `List.ofArray` | `Seq.ofArray` |
|           | ×              | `Array.toList` | `Array.toSeq` |
| `List`    | `Array.ofList` | ×              | `Seq.ofList`  |
|           | `List.toArray` | ×              | `List.toSeq`  |
| `Seq`     | `Array.ofSeq`  | `List.ofSeq`   | ×             |
|           | `Seq.toArray`  | `Seq.toList`   | ×             |

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

* [Array module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-arraymodule.html)
* [List module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-listmodule.html)
* [Map module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-mapmodule.html)
* [Seq module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-seqmodule.html)
* [Set module](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-collections-setmodule.html)

[^1]: See Sum(By) below
