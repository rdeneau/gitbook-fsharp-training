# Operators

## Definition

Function whose name is a set of symbols

* Unary operator: `let (~symbols) = ...`
* Binary operator: `let (symbols) = ...`
* _Symbols_ = combination of `% & * + - . / < = > ? @ ^ | ! $`

2 ways of using operators:

1. As an operator: `1 + 2`
   * without the `()`
   * prefix position if unary `-1`.
   * infix position id binary `1 + 2`
2. As a function: `(+) 1 2`
   * with the `()`

## Standard operators

Also defined in `FSharp.Core`.

* Arithmetic operators: `+`, `-`...
* Pipeline operators
* Composition operators

## Piping operators

Binary operators, placed between a simple value and a function

* Apply the value to the function = Pass value as argument
* Avoid parentheses otherwise required for precedence reason
* There are several _pipes_
  * _Pipe right_ `|>` : the usual _pipe_
  * _Pipe left_ `<|` a.k.a. reversed _pipe_
  * _Pipe right 2_ `||>`
  * Etc.

### Pipe right `|>`

Reverse the order between function and value: `val |> fn` ≡ `fn val`

* Natural "subject-verb" order, as a method call of an object (`obj.M(x)`)
* _Pipeline_: chain function calls, without intermediate variable
* Help the type inference - example:

```fsharp
let items = ["a"; "bb"; "ccc"]

let longestKo = List.maxBy (fun x -> x.Length) items  // ❌ Error FS0072
//                                   ~~~~~~~~

let longest = items |> List.maxBy (fun x -> x.Length) // ✅ Return "ccc"
```

### Pipe left `<|`

`fn <| expression` ≡ `fn (expression)`

* ☝ Usage a little less common than `|>`
* ✅ Minor advantage: avoids parentheses
* ❌ Major disadvantage: reads from right to left → Reverses natural English reading direction and execution order

```fsharp
printf "%i" 1+2          // 💥 Error
printf "%i" (1+2)        // With parentheses
printf "%i" <| 1+2       // With reversed pipe
```

#### Quid of such expression: `x |> fn <| y` ❓

Executed from left to right: `(x |> fn) <| y` ≡ `(fn x) <| y` ≡ `fn x y`

* Goal: use `fn` as infix
* Cons: difficult to read because of double reading direction ❗

👉 Tip: **TO AVOID**

### Pipe right 2 `||>`

`(x, y) ||> fn` ≡ `fn x y`

* To pass 2 arguments at once, in the form of a tuple
* Used infrequently, but useful with `fold` to pass the initial value (`seed`) and the list before defining the `folder` function and help the type inference for a lambda `folder`.

```fsharp
let items = [1..5]

// 😕 While reading, we can miss the 0 at the end (the seed)
let sumOfEvens =
    items |> List.fold (fun acc x -> if x % 2 = 0 then acc + x else acc) 0

let sumOfEvens' =
    (0, items)
    ||> List.fold (fun acc x -> if x % 2 = 0 then acc + x else acc)

// 💡 Replace lambda with named function
let addIfEven acc x = if x % 2 = 0 then acc + x else acc
let sumOfEvens'' = items |> List.fold addIfEven 0
```

☝️ This operator corresponds to the notion of "uncurrying", i.e. being able to call a curried `f` function by passing it these 2 parameters in the form of a tuple:

```fsharp
let uncurry f (a,b) = f a b
```

## Compose `>>`

Binary operators, placed **between two functions**\
→ The result of the 1st function is used as an argument for the 2nd function

`f >> g` ≡ `fun x -> g (f x)` ≡ `fun x -> x |> f |> g`

{% hint style="warning" %}
* The out/in types must match: `f: 'T -> 'U` and `g: 'U -> 'V` → `'U` ✅
* Signature of the final function: `'T -> 'V`.
{% endhint %}

```fsharp
let add1 x = x + 1
let times2 x = x * 2

let add1Times2 x = times2 (add1 x) // Verbose
let add1Times2' = add1 >> times2   // Terse 👍
```

## Reversed Compose `<<`

Rarely used, except to restore a natural reading order.

Example with `not` _(which replaces the `!` in C♯):_

```fsharp
let Even x = x % 2 = 0

// Pipeline
let Odd x = x |> Even |> not

// Compose
let Odd = not << Even
```

## _Pipe_ `|>` or _Compose_ `>>` ?

<table><thead><tr><th width="137"></th><th>Definition</th><th>Focus, Mindset</th></tr></thead><tbody><tr><td><em><strong>Compose</strong></em> </td><td><strong><code>let h = f >> g</code></strong></td><td>Functions</td></tr><tr><td><em><strong>Pipe</strong></em> </td><td><strong><code>let result = value |> f</code></strong></td><td>Values</td></tr></tbody></table>

## Point-free style

A.k.a _Tacit programming_

Writing functions without mentioning the parameters _(referred here as "points"),_ just by using function composition (1) or partial application (2).

```fsharp
// Example 1: function composition
let add1 x = x + 1                // (x: int) -> int
let times2 x = x * 2              // (x: int) -> int
let add1Times2 = add1 >> times2   // int -> int

// Example 2: partial application
let isEven x = (x % 2 = 0)
let evens = List.filter isEven // int list -> int list

// Example 2': printfn partially applied
let greetLong name age = printfn $"My name is %s{name} and I am %i{age} years old!" // name:string -> age:int -> unit
let greetShort = printfn "My name is %s and I am %d years old!" // (string -> int -> unit)
```

### Pros/Cons ⚖️

#### ✅ Pros

* Concise style
* At function/operation level, by abstracting the parameters

#### ❌ Cons

Loses the name of the parameter now implicit in the signature\
→ Unimportant if the function remains understandable:

* When the parameters name is not significant (e.g. `x`)
* When the combination of parameters type + function name is unambiguous
* For private usage - Not recommended for a public API

### Limit 🛑

Works poorly with generic functions:

```fsharp
let isNotEmptyKo = not << List.isEmpty
// 💥 Error FS0030: Value restriction: The value 'isNotEmptyKo' has an inferred generic function type

// Alternatives
let isNotEmpty<'a> = not << List.isEmpty<'a>    // 👌 Explicit type annotation
let isNotEmpty' list = not (List.isEmpty list)  // 👌 Explicit parameter
```

🔗 [F♯ coding conventions > Partial application and point-free programming](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#partial-application-and-point-free-programming)

## Custom operators

2 possibilities:

* Operator overload
* Creation of a new operator

## Operator overload

Usually concerns a specific type\
→ Overload defined within the associated type _(as in C♯)_

```fsharp
type Vector = { X: int; Y: int } with
    // Unary operator (see ~ and 1! param) for vector inversion
    static member (~-) (v: Vector) =
        { X = -v.X
          Y = -v.Y }

    // Binary addition operator for 2 vectors
    static member (+) (a: Vector, b: Vector) =
        { X = a.X + b.X
          Y = a.Y + b.Y }

let v1 = -{ X=1; Y=1 } // { X = -1; Y = -1 }
let v2 = { X=1; Y=1 } + { X=1; Y=3 } // { X = 2; Y = 4 }
```

## Creation of a new operator

* Definition rather in a module or associated type
* Usual use-case: alias for existing function, used as infix

```fsharp
// "OR" Composition of 2 functions (fa, fb) which return an optional result
let (<||>) fa fb x =
    match fa x with
    | Some v -> Some v // Return value produced by (fa x) call
    | None   -> fb x   // Return value produced by (fb x) call

// Functions: int -> string option
let tryMatchPositiveEven x = if x > 0 && x % 2 = 0 then Some $"Even {x}" else None
let tryMatchPositiveOdd x = if x > 0 && x % 2 <> 0 then Some $"Odd {x}" else None
let tryMatch = tryMatchPositiveEven <||> tryMatchPositiveOdd

tryMatch 0;; // None
tryMatch 1;; // Some "Odd 1"
tryMatch 2;; // Some "Even 2"
```

## Symbols allowed in an operator

**Unary operator "tilde "**\
→ `~` followed by `+`, `-`, `+.`, `-.`, `%`, `%%`, `&`, `&&`

**Unary operator "snake "**\
→ Several `~`, e.g. `~~~~`

**Unary operator "bang "**\
→ `!` followed by a combination of `!`, `%`, `&`, `*`, `+`, `.`, `/`, `<`, `=`, `>`, `@`, `^`, `|`, `~`, `?`\
→ Except `!=` (!=) which is binary

**Binary operator**\
→ Any combination of `!`, `%`, `&`, `*`, `+`, `.`, `/`, `<`, `=`, `>`, `@`, `^`, `|`, `~`, `?`\
→ which does not match a unary operator

## Usage symbols

All operators are used as is\
❗ Except the unary operator "tilde": used without the initial `~`.

| Operator     | Declaration         | Usage     |
| ------------ | ------------------- | --------- |
| Unaire tilde | `let (~&&) x = …`   | `&&x`     |
| Unaire snake | `let (~~~) x = …`   | `~~~x`    |
| Unaire bang  | `let (!!!) x = …`   | `!!!x`    |
| Binary       | `let (<ˆ>) x y = …` | `x <ˆ> y` |

☝ To define an operator beginning or ending with a `*`, you must put a space between `(` and `*` as well as between `*` and `)` to distinguish from a block of F♯ comments `(* *)`.\
→ `let ( *+ ) x y = x * y + y` ✅

## Operator or function?

### Infix operator _vs_ function

👍 **Pros** :

* Respects the natural reading order (left → right)
*   avoids parentheses\
    → `1 + 2 * 3` _vs_ `multiply (add 1 2) 3`

    → `1 + 2 * 3` _vs_ `multiply (add 1 2) 3`

⚠️ **Cons** :

* A "folkloric" operator (e.g. `@!`) will be less comprehensible than a function whose name uses the **domain language**.

### Using an operator as a function

💡 You can use the partial application of a binary operator :

Examples:

* Instead of a lambda:\
  → `(+) 1` ≡ `fun x -> x + 1`
* To define a new function :\
  → `let isPositive = (<) 0` ≡ `let isPositive x = 0 < x` ≡ `x >= 0` \\
