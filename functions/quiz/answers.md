# Answers

# 🍔 Quiz

## 1. How to define the return value (`v`) of a function (`f`)? ⏱ 10''

**A.** Simply name the value `result`. ❌

**B.** End the function with `return v`. ❌

**C.** Do `f = v`.  ❌

**D.** `v` is the last line of `f`. ✅

***

## 2. How to write an `add` function taking 2 `strings` and returning an `int`. ⏱ 20’’

**A.** `let add a b = a + b` ❌

* Wrong type inferred for `a` and `b` : `int`

**B.** `let add (a: string) (b: string) = (int a) + (int b)` ✅

* The type of `a` and `b` must be specified.
* They must be converted to `int`.
* The `int` return type can be inferred.

**C.** `let add (a: string) (b: string) : int = a + b` ❌

* Returns a string

***

## 3. What does this code: `add >> multiply`? ⏱ 10’’

**A.** Create a pipeline ❌

**B.** Define a named function ❌ (although it can be a function body)

**C.** Compose 2 functions ✅

***

## 4. Find the name of these functions from [FSharp.Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/) ⏱ 60’’

**A.** `let ? _ = ()`

→ `let inline ignore _ = ()` - see [prim-types.fs#L459](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L459)

**B.** `let ? x = x`

→ `let id x = x` (Identity) - see [prim-types.fs#L4831](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L4831)

**C.** `let ? f x = f x`

`let inline (<|) func arg = func arg` (Pipe Left) - see [prim-types.fs#L3914](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3914)

**D.** `let ? x f = f x`

`let inline (|>) arg func = func arg` (Pipe Right) - see [prim-types.fs#L3908](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3908)

**E.** `let ? f g x = g (f x)`

`let inline (>>) func1 func2 x = func2 (func1 x)` (Compose Right) - see [prim-types.fs#L3920](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs#L3920)

***

## 5. Describe the following functions in terms of the number of parameters, their type, the type of return ⏱ 60’’

**A.** `int -> unit`

> 1 parameter: `int` - no return value

**B.** `unit -> int`

> no parameter - return a `int`

**C.** `string -> string -> string`

> 2 parameters: `string` - return a `string`

**D.** `string -> string -> string`

> 2 parameters: a predicate and a list - returns a list \
> → `filter` function


***

## 6. What is the signature of the `h` function below? ⏱ 30’’

```fsharp
let f x = x + 1
let g x y = $"%i{x} + %i{y}"
let h = f >> g
```

**C.** `int -> int -> string` ✅

`let f x = x + 1` → `f: (x: int) -> int` » `1` → `int` → `x: int` → `x + 1: int`

`let g x y = $"{+x} + {+y}"` → `(x: int) -> (y: int) -> string` » `%i{x}` → `int` » `$"..."` → `string`

`let h = f >> g` \
» `h` can be written `let h x y = g (f x) y`

☝️ **Note:** this question was hard enough to illustrate the **misuse** of `>>` because compound functions have different arities (`f` has 1 parameter, `g` has 2).

***

## 7. What value returns `f 2`? ⏱ 10’’

```fsharp
let f = (-) 1;
f 2 // ?
```

**C.** `-1` ❗

* Counter-intuitive, isn't it? We expect f to decrement by 1.
* It's easier to understand what's going on if we write `f` like this: `let f x = 1 - x`.

💡 **Hint:** the function that decrements by 1 can be written as:

* `let f = (+) -1` _(this works here because `+` is commutative whereas `-` is not)_
* `let f x = x - 1`
