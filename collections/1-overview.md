# Overview

## Common F♯ collections

| Module |  Type          | - | BCL Equivalent             | Immutable   | Structural comparison |
|--------|----------------|---|----------------------------|-------------|-----------------------|
| `Array`|  `'T array`    | ≡ | `Array<T>`                 | ❌          | ✅                   |
| `List` |  `'T list`     | ≃ | `ImmutableList<T>`         | ✅          | ✅                   |
| ❌     | `ResizeArray`  | ≡ | `List<T>`                  | ✅          | ❌                   |
| `Seq`  |  `seq<'T>`     | ≡ | `IEnumerable<T>`           | ✅          | ✅                   |
| `Set`  |  `Set<'T>`     | ≃ | `ImmutableHashSet<T>`      | ✅          | ✅                   |
| `Map`  |  `Map<'K, 'V>` | ≃ | `ImmutableDictionary<K,V>` | ✅          | ✅                   |
| ❌     | `dict`         | ≡ | `IDictionary<K,V>`         | ☑️ readonly | ❌                   |
| ❌     | `readOnlyDict` | ≡ | `IReadOnlyDictionary<K,V>` | ☑️ readonly | ❌                   |

## Functions consistency 👍

Common to all 5 modules:

- `empty`/`isEmpty`, `exists`/`forall`
- `find`/`tryFind`, `pick`/`tryPick`, `contains` (`containsKey` for `Map`)
- `map`/`iter`, `filter`, `fold`

Common to `Array`, `List`, `Seq`:

- `append`/`concat`, `choose`, `collect`
- `item`, `head`, `last`
- `take`, `skip`
- ... _a hundred functions altogether!_

## Syntax consistency 👍

| Type    | Construction   | Range          | Comprehension |
|---------|----------------|----------------|---------------|
| `Array` | `[∣ 1; 2 ∣]`   | `[∣ 1..5 ∣]`   | ✅            |
| `List`  | `[ 1; 2 ]`     | `[ 1..5 ]`     | ✅            |
| `Seq`   | `seq { 1; 2 }` | `seq { 1..5 }` | ✅            |
| `Set`   | `set [ 1; 2 ]` | `set [ 1..5 ]` | ✅            |

## Syntax trap ⚠️

Square brackets `[]` are used for:

- _Value:_ instance of a list `[ 1; 2 ]` (of type `int list`)
- _Type:_ array `int []`, e.g. of `[| 1; 2 |]`

☝ **Recommendations**

- Distinguish between type _vs_ value ❗
- Write `int array` rather than `int[]`

## Comprehension

- Syntax similar to `for` loop
- Same principle as generators in C♯, JS
  - `yield` keyword but often **optional** (since F♯ 4.7)
  - `yield!` keyword _(pronounce "yield bang")_ ≡ `yield*` in JS
  - Works for all collections 👍

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
        yield! set [i; i+1] ]
```
