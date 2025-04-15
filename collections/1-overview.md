# Overview

## Common F♯ collections

<table><thead><tr><th width="85">Module</th><th width="146">Type</th><th width="44" align="center">-</th><th width="227">BCL Equivalent</th><th width="105">Immutable</th><th>Structural comparison</th></tr></thead><tbody><tr><td><code>Array</code></td><td><code>'T array</code></td><td align="center">≡</td><td><code>Array&#x3C;T></code></td><td>❌</td><td>✅</td></tr><tr><td><code>List</code></td><td><code>'T list</code></td><td align="center">≃</td><td><code>ImmutableList&#x3C;T></code></td><td>✅</td><td>✅</td></tr><tr><td><code>Seq</code></td><td><code>seq&#x3C;'T></code></td><td align="center">≡</td><td><code>IEnumerable&#x3C;T></code></td><td>✅</td><td>✅</td></tr><tr><td><code>Set</code></td><td><code>Set&#x3C;'T></code></td><td align="center">≃</td><td><code>ImmutableHashSet&#x3C;T></code></td><td>✅</td><td>✅</td></tr><tr><td><code>Map</code></td><td><code>Map&#x3C;'K, 'V></code></td><td align="center">≃</td><td><code>ImmutableDictionary&#x3C;K,V></code></td><td>✅</td><td>✅</td></tr><tr><td>❌</td><td><code>dict</code></td><td align="center">≡</td><td><code>IDictionary&#x3C;K,V></code></td><td>☑️ ❗</td><td>❌</td></tr><tr><td>❌</td><td><code>readOnlyDict</code></td><td align="center">≡</td><td><code>IReadOnlyDictionary&#x3C;K,V></code></td><td>☑️</td><td>❌</td></tr><tr><td>❌</td><td><code>ResizeArray</code></td><td align="center">≡</td><td><code>List&#x3C;T></code></td><td>❌</td><td>❌</td></tr></tbody></table>

## Functions consistency 👍

Common to all 5 modules:

* `empty`/`isEmpty`, `exists`/`forall`
* `find`/`tryFind`, `pick`/`tryPick`, `contains` (`containsKey` for `Map`)
* `map`/`iter`, `filter`, `fold`

Common to `Array`, `List`, `Seq`:

* `append`/`concat`, `choose`, `collect`
* `item`, `head`, `last`
* `take`, `skip`
* ... _a hundred functions altogether!_

## Syntax consistency 👍

| Type    | Construction   | Range          | Comprehension📍 |
| ------- | -------------- | -------------- | --------------- |
| `Array` | `[∣ 1; 2 ∣]`   | `[∣ 1..5 ∣]`   | ✅               |
| `List`  | `[ 1; 2 ]`     | `[ 1..5 ]`     | ✅               |
| `Seq`   | `seq { 1; 2 }` | `seq { 1..5 }` | ✅               |
| `Set`   | `set [ 1; 2 ]` | `set [ 1..5 ]` | ✅               |

## Syntax trap ⚠️

Square brackets `[]` are used both for:

* _Value:_ instance of a list `[ 1; 2 ]` (of type `int list`)
* _Type:_ array `int []`, e.g. of `[| 1; 2 |]`

☝ **Recommendations**

* Distinguish between type _vs_ value ❗
* Write `int array` rather than `int[]`

## Comprehension

* **Purpose:** syntactic sugar to construct collection
  * Handy, succinct, powerful
  * Syntax includes `for` loops, `if` condition
* Same principle as generators in C♯, JS
  * `yield` keyword but often **optional** (since F♯ 4.7)
  * `yield!` keyword _(pronounce "yield bang")_ ≡ `yield*` in JS
  * Works for all collections 👍

**Examples:**

```fsharp
// Multi-line (recommended)
let squares =
    seq { for i in 1 .. 10 do
        yield i * i // 💡 'yield' can be omitted most of the time 👍
    }

// Single line
let squares = seq { for i in 1 .. 10 -> i * i }

// Can contain 'if'
let halfEvens =
    [ for i in [1..10] do
        if (i % 2) = 0 then i / 2 ]  // [1; 2; 3; 4; 5]

// Nested 'for'
let pairs =
    [ for i in [1..3] do
      for j in [1..3] do
        i, j ]              // [(1, 1); (1; 2); (1; 3); (2, 1); ... (3, 3)]
```

Flattening:

```fsharp
// Multiple items
let twoToNine =
    [ for i in [1; 4; 7] do
        if i > 1 then i
        i + 1
        i + 2 ]  // [2; 3; 4; 5; 6; 7; 8; 9]

// With 'yield! collections'
let oneToSix =
    [ for i in [1; 3; 5] do
        yield! [i; i+1] ]
```
