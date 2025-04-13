# Tuples

## Key points

* Types constructed from **literal values**
* Anonymous types, but aliases can be defined to give them a name
* Product types by definition
  * Hence the `*` sign in the type signature: `A * B`
* Number of elements in the tuples:
  * 👌 2 or 3 (`A * B * C`)
  * ⚠️ > 3 : possible but prefer _Record_
* Order of elements is important
  * If `A` ≠ `B`, then `A * B` ≠ `B * A`

## Construction

Syntax of literals: `a,b` or `a, b` or `(a, b)`

* Comma `,`: symbol dedicated to tuples in F#
* Spaces   are optional
* Parentheses `()` may be necessary

**⚠️ Pitfall:** the symbol used is different for literals _vs_ types

* `,` for literal
* `*` for signature
* E.g. `true, 1.2` → `bool * float`

## Deconstruction

👍 Same syntax as construction

⚠️ All elements must appear in the deconstruction\
&#x20;  → Use `_` _(discard)_ to ignore one of the elements

```fsharp
let point = 1.0, 2.5
let x, y = point

let x, y = 1, 2, 3 // 💥 Error FS0001: Type incompatibility...
                   // ... Tuples have lengths other than 2 and 3

let result = System.Int32.TryParse("123") // (bool * int)
let _, value = result // Ignore the "bool"
```

## Tuples in practice

### Use cases

Use a tuple for a data structure:

* Small: 2 to 3 elements
* Light: no need for element names
* Local: small scope

Tuples are immutable:\
→ modifications are made by creating a new tuple

```fsharp
let addOneToTuple (x, y, z) = x + 1, y + 1, z + 1
```

### Structural equality

Structural equality works by default, but only between 2 tuples of the same signature:

```fsharp
(1,2) = (1,2)       // true
(1,2) = (0,0)       // false
(1,2) = (1,2,3)     // 💥 Error FS0001: Type incompatibility...
                    // ... Tuples have lengths other than 2 and 3
(1,2) = (1,(2,3))   // 💥 Error FS0001: This expression was supposed to have type `int`...
                    // ... but here it has type `'a * 'b`
```

### Nesting

Tuples can be nested in bigger tuples using parentheses `()`

```fsharp
let doublet = (true,1), (false, "a")  // (bool * int) * (bool * string) → pair of pairs
let quadruplet = true, 1, false, "a"  // bool * int * bool * string → quadruplet
doublet = quadruplet                  // 💥 Error FS0001: Type incompatibility...
```

## Pattern matching

Patterns recognized with tuples:

```fsharp
let print move =
    match move with
    | 0, 0 -> "No move"                     // Constant 0
    | 0, y -> $"Vertical {y}"               // Variable y (!= 0)
    | x, 0 -> $"Horizontal {x}"
    | x, y when x = y -> $"Diagonal {x}"    // Condition x and y equal
                                            // `x, x` is not a recognized pattern ❗
    | x, y -> $"Other ({x}, {y})"
```

☝ **Notes:**

* Patterns are ordered from specific to generic
* The last pattern `x, y` is the default one to deconstruct a tuple

## Pairs

* 2-element tuples
* So common that 2 helpers are associated with them:
  * `fst` as _first_ to extract the 1st element of the pair
  * `snd` as _second_ to extract the 2nd element of the pair
  * ⚠️ Only works for pairs

```fsharp
let pair = 'a', "b"
fst pair  // 'a' (char)
snd pair  // "b" (string)
```

## 🕹️ Quiz

**1. Implement `fst` and `snd`**

```fsharp
let fst ... ?
let snd ... ?
```

<details>

<summary>Answer</summary>

```fsharp
let inline fst (x, _) = x  // Signature : 'a * 'b -> 'a
let inline snd (_, y) = y  // Signature : 'a * 'b -> 'b
```

* Tuple deconstruction: `(x, y)`
* We _discard_ one element using `_` wildcard
* Functions can be `inline`

</details>

**2. What is the signature of this function?**

```fsharp
let toList (x, y) = [x; y]
```

<details>

<summary>Answer</summary>

`x: 'a * y: 'a -> 'a list`

* Returns a list with the 2 elements `x` and `y` of the pair
* The elements are therefore of the same type
* There is no constraint on this type → generic type `'a`

</details>
