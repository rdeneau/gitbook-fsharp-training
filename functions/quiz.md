---
description: 6 questions to test your memory
---

# 🍔 Quiz

## 1. How to define the return value (`v`) of a function (`f`)? ⏱ 10''

**A.** Simply name the value `result`.

**B.** End the function with `return v`.

**C.** Do `f = v`

**D.** `v` is the last line of `f`.

<details>

<summary>Answer</summary>

**A.** ❌ F♯ is not a convention-based language.

**B.** ❌ F♯ does have a `return` keyword, but it's used in computation expressions. 📍

**C.** ❌ This is the VB way, not the F♯ way.

**D.** ✅ The returned value of a function is its whole body: an expression that can be composed from sub-expressions.

</details>

***

## 2. How to write an `add` function taking 2 `strings` and returning an `int`? ⏱ 20’’

**A.** `let add a b = a + b`

**B.** `let add (a: string) (b: string) = (int a) + (int b)`

**C.** `let add (a: string) (b: string) : int = a + b`

<details>

<summary>Answer</summary>

**A.** ❌

* `+` operator works on different types.
* There is no way to define a type for all the types supported by `+`.
* The compiler only takes the first type, which is `int`.
* In `let add a b = a + b`, `a` and `b` are inferred to be of type `int`.

**B.** ✅

* The type of `a` and `b` must be specified.
* They must be converted to `int`.
* The `int` return type can be inferred.

**C.** ❌

* Returns a string

</details>

***

## 3. What does this code: `add >> multiply`? ⏱ 10’’

**A.** Create a pipeline

**B.** Define a named function

**C.** Compose 2 functions

<details>

<summary>Answer</summary>

**A.** ❌ Pipelines are based on the pipe operator `|>`

**B.** ❌ A **named** function is defined with a `let` binding. `add >> multiply` is an **anonymous** function.

**C.** ✅ `>>` is the compose operator, taking 2 functions as arguments to build a new function.

</details>

***

## 4. Find the name of these functions from [FSharp.Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/) ⏱ 60’’

**A.** `let ? _ = ()`

**B.** `let ? x = x`

**C.** `let ? f x = f x`

**D.** `let ? x f = f x`

**E.** `let ? f g x = g (f x)`

> 💡 **Tips:** These may be operators.

<details>

<summary>Answer</summary>

**A.** `let ? _ = ()`

This function discards its input parameter and returns `unit`.\
→ `let inline ignore _ = ()` - see [prim-types.fs#L459](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L459)

**B.** `let ? x = x`

This function just returns its input parameter: it's the _identity_ function.\
→ `let id x = x` - see [prim-types.fs#L4831](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L4831)

**C.** `let ? f x = f x`

* This function is taking 2 parameters `f` and `x`.
* `f x` implies that `f` is a function.
* This function as a function adds no value.
* On the contrary, as operator, we get the benefit of having the possibility to use the operator between the function and its value, for instance to remove the need to use parentheses around the expression used to be passed as argument to the function.
* This operator is called _pipe left._

→ `let inline (<|) func arg = func arg` - see [prim-types.fs#L3914](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3914)

**D.** `let ? x f = f x`

* Closed to the previous function, with the 2 parameters swapped.
* Similarly, it makes sense only as an operator, called _pipe right_ or just _pipe_.

→ `let inline (|>) arg func = func arg` (Pipe Right) - see [prim-types.fs#L3908](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3908)

**E.** `let ? f g x = g (f x)`

* This function is taking 3 parameters `f`, `g` and `x`.

- `f x` and `g (...)` indicates that `f` and `g` are functions.
- We call `f` with the argument `x`, then we call `g` with the previous result.
- It's the definition of the _compose right_ operator, noted `>>`.

→ `let inline (>>) func1 func2 x = func2 (func1 x)` - see [prim-types.fs#L3920](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3920)

</details>

***

## 5. Describe the following functions in terms of the number of parameters, their type, the return type ⏱ 60’’

**A.** `int -> unit`

**B.** `unit -> int`

**C.** `string -> string -> string`

**D.** `('T -> bool) -> 'T list -> 'T list`

<details>

<summary>Answer</summary>

**A.** `int -> unit`

> * 1 parameter: `int`
> * no return value

**B.** `unit -> int`

> * no parameter
> * returns an `int`

**C.** `string -> string -> string`

> * 2 parameters, both `string`
> * returns a `string`
>
> → Can be the `+` operator.

**D.** `('T -> bool) -> 'T list -> 'T list`

> * 2 parameters
>   * `('T -> bool)` is a function returning a boolean\
>     → It's a _predicate_.
>   * `'T list` is a list.
> * returns a list
>
> → Can be the `List.filter` function.

</details>

***

## 6. What is the signature of the `h` function below? ⏱ 30’’

{% code lineNumbers="true" %}
```fsharp
let f x = x + 1
let g x y = $"%i{x} + %i{y}"
let h = f >> g
```
{% endcode %}

**A.** `int -> int`

**B.** `int -> string`

**C.** `int -> int -> string`

**D.** `int -> int -> int`

<details>

<summary>Answer</summary>

**C.** `int -> int -> string` ✅

1. `+ 1` → `x: int` \
   → `f: (x: int) -> int`
2. `%i{x}` and `%i{y}` in interpolated string `$"..."` → `x` and `y` are `int`s\
   → `(x: int) -> (y: int) -> string`
3. &#x20;`>>` compose operator\
   → `h` takes the same parameter as `f` and returns what is returned by the call to `g` with the value returned from `f`\
   → But, `g` takes 2 parameters. Hence, `g` is partially applied and 1 argument is still needed.

The exact signature is in fact `int -> (int -> string)`.

☝️ **Notes:**

* This question was hard enough to illustrate the **misuse** of `>>` because composed functions have different arities (`f` has 1 parameter, `g` has 2).
* It's way simpler to write `let h x y = g (f x) y`.

</details>

***

## 7. What value returns `f 2`? ⏱ 10’’

```fsharp
let f = (-) 1
f 2;; // ?
```

**A.** `1`

**B.** `3`

**C.** `-1`

<details>

<summary>Answer</summary>

**C.** `-1` ❗

* Counter-intuitive, isn't it? We expect f to decrement by 1.
* It's easier to understand what's going on if we write `f` like this: `let f x = 1 - x`.

💡 **Hint:** the function that decrements by 1 can be written as:

* `let f = (+) -1` _(this works here because `+` is commutative whereas `-` is not)_
* `let f x = x - 1`

</details>
