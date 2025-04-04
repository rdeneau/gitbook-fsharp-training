---
description: 6 questions to test your memory
---

# 🍔 Quiz

## 1. How to define the return value (`v`) of a function (`f`)? ⏱ 10''

**A.** Simply name the value `result`.

**B.** End the function with `return v`.

**C.** Do `f = v`

**D.** `v` is the last line of `f`.

***

## 2. How to write an `add` function taking 2 `strings` and returning an `int`. ⏱ 20’’

**A.** `let add a b = a + b`

**B.** `let add (a: string) (b: string) = (int a) + (int b)`

**C.** `let add (a: string) (b: string) : int = a + b`

***

## 3. What does this code: `add >> multiply`? ⏱ 10’’

**A.** Create a pipeline

**B.** Define a named function

**C.** Compose 2 functions

***

## 4. Find the name of these functions from [FSharp.Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/) ⏱ 60’’

**A.** `let ? _ = ()`

**B.** `let ? x = x`

**C.** `let ? f x = f x`

**D.** `let ? x f = f x`

**E.** `let ? f g x = g (f x)`

💡 Tips: These may be operators.

***

## 5. Describe the following functions in terms of the number of parameters, their type, the type of return ⏱ 60’’

**A.** `int -> unit`

**B.** `unit -> int`

**C.** `string -> string -> string`

**D.** `('T -> bool) -> 'T list -> 'T list`

***

## 6. What is the signature of the `h` function below? ⏱ 30’’

```fsharp
let f x = x + 1
let g x y = $"%i{x} + %i{y}"
let h = f >> g
```

**A.** `int -> int`

**B.** `int -> string`

**C.** `int -> int -> string`

**D.** `int -> int -> int`

***

## 7. What value returns `f 2`? ⏱ 10’’

```fsharp
let f = (-) 1;
f 2 // ?
```

**A.** `1`

**B.** `3`

**C.** `-1`
