# First concepts

## Expression vs Statement

> A **statement** will produce a side effect.\
> An **expression** will produce a value... and a possible side effect (that we should avoid).

* F♯ is a functional, expression-based language only.
* In comparison, C♯ is an imperative language, based on statements, but includes more and more syntactic sugar based on expressions:
  * Ternary operator `b ? x : y`
  * [Null-conditional operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) `?.` in C♯ 6 : `model?.name`
  * [Null-coalescing operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) `??` in C♯ 8 : `label ?? '(Vide)'`
  * Lambda expressions in C♯ 3 with LINQ : `numbers.Select(x => x + 1)`
  * [Expression-bodied members](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members) in C♯ 6 and 7
  * `switch` expression in C♯ 8

### ⚖️ Benefits of expressions over instructions

* **Conciseness**: code + compact == + readable
* **Composability**: composing expressions is like composing values
* **Understanding**: no need to know the previous instructions to understand the current one
* **Testability**: pure expressions _(no side effects)_, easier to test
  * _Predictable_: same inputs produce same outputs
  * _Isolated_: shorter Arrange/Setup phase in tests _(no mocks...)_

## Everything is an expression

* A function is declared and behaves like a value
  * We can pass it as parameter or return it from another function \
    &#xNAN;_&#x53;ee high-order functions_
* The _control flow_ building blocks are also expressions
  * `if/else` , `match`
  * `for` , **`while`** just return nothing - see `unit` below

**Consequences**

* No `void` \
  → Best replaced by the `unit` type having only one value written`()` 📍
* No _**Early Exit**_
  * In C#, you can exit a function with `return` and exit a `for/while` loop with `break`.
  * In F# these keywords do not exist.

### Early exit alternatives

The most questionable solution is to raise an exception 💩 (see [réponse StackOverflow](https://stackoverflow.com/a/42018355/8634147))

One solution in imperative style is to use mutable variables 😕

```fsharp
let firstItemOrDefault defaultValue predicate (items: 't array) =
    let mutable result = None
    let mutable i = 0
    while i < items.Length && result.IsNone do
        let item = items[i]
        if predicate item then
            result <- Some item
        i <- i + 1

    result
    |> Option.defaultValue defaultValue

let test1' = firstItemOrDefault -1 (fun x -> x > 5) [| 1 |]     // -1
```

The most recommended and idiomatic solution in functional programming is to use a recursive function 📍

```fsharp
[<TailCall>] // F# 8 📍
let rec firstOr defaultValue predicate list =
    match list with
    | [] -> defaultValue                                // 👈 Exit
    | x :: _ when predicate x -> x                      // 👈 Exit
    | _ :: rest -> firstOr defaultValue predicate rest  // 👈 Recursive call to continue

let test1 = firstOr -1 (fun x -> x > 5) [1]     // -1
let test2 = firstOr -1 (fun x -> x > 5) [1; 6]  // 6
```

&#x20;💡 As indicated by the `TailCall` attribute (introduced in F# 8), this is a case of [tail recursion](https://en.wikipedia.org/wiki/Tail_call): the compiler will convert the call to this recursive function into a simple loop with much better performance. You can check this in [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfA2gHgCoEMCWwDCmwwAfALoCwAUMAKYAuABAE40DGDY2TEdA8kwxg0wmAK7A6ANUKiaDAA4sY2Vpjpzg2HgwC8VBgYYBbNawAWDTdoDu2Omf2HEDZKQYBaYoOFiJ04LKGQcEhoaEA9OEMgLwbgBI7DADKAPZMdNg0jgbOAB4MICAMAPoM1mY0AHYKSipqcrmeDLlhzUGRMfHJqemZDM7F+cw02g2c3HwCQiLiUjJyijTKquqD2gxtcQwAgvLyNMDMAJesotzYYAyASYQMrEnlaeWyVEA=).

## Typing, inference and ceremony

The ceremony is correlated to the typing strength

<table><thead><tr><th width="146">Language</th><th>Typing strength</th><th>Inference strength</th><th>Ceremony</th></tr></thead><tbody><tr><td>JS</td><td>Low <br><em>(dynamic typing)</em></td><td>×</td><td>Low</td></tr><tr><td>C♯</td><td>Moyen (statique nominal)</td><td>Low</td><td>High</td></tr><tr><td>TS</td><td>Fort (statique structurel + ADT)</td><td>Medium</td><td>Medium</td></tr><tr><td>F♯</td><td>Fort (statique nominal + ADT)</td><td>High</td><td>Low</td></tr></tbody></table>

🔗 [Zone of Ceremony](https://blog.ploeh.dk/2019/12/16/zone-of-ceremony/) by Mark Seemann

## Type inference

Goal: write type annotations as little as possible

* Less code to write 👍
* Compiler ensures consistency
* IntelliSense helps with coding and reading

### Type inference in F♯

[Hindley–Milner](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system) method

* Able to deduce the type of variables, expressions and functions in a program without any type annotation
* Based on both the implementation and the usage(s)

```fsharp
let helper instruction source =
    if instruction = "inc" then // 1. `instruction` has the same type than `"inc"` => `string`
      source + 1                // 2. `source` has the same type than `1` => `int`
    elif instruction = "dec" then
      source - 1
    else
      source                    // 3. `return` has the same type than `source` => `int`
```

### Automatic generalization in F♯ inference

If something can be inferred as generic, it will be\
→ Open to more cases 🥳

```fsharp
// Generic value
let a = [] // 'a list

// Generic function with both parameters generic
let listOf2 x y = [x; y]
// val listOf2: x: 'a -> y: 'a -> 'a list

// Generic type constraint inference: 'a must be "comparable"
let max x y = if x > y then x else y
```

☝ In F♯, a generic type starts with an apostrophe and can be in camelCase (`'a`) or PascalCase (`'A`) .

### Inference _vs_ type annotation

* Pros:
  * code terser
  * automatic generalization
* Cons:
  * we can break code in cascade
  * inference limited:&#x20;
    * an object type cannot be determine by the call to one of its members (1)\
      → exception: Record types 📍
    * sensible to the order of declaration (2)

(1) Example:

```fsharp
let helperKO instruction source =
    match instruction with
    | 'U' -> source.ToUpper()
    //       ~~~~~~~~~~~~~~~~ 💥
    // Error FS0072: Lookup on object of indeterminate type based on information prior to this program point.
    // A type annotation may be needed prior to this program point to constrain the type of the object.
    | _   -> source

let helperOk instruction (source: string) = [...]
// Type annotation needed here  : ^^^^^^

// If there is a function equalivalent to the method, it will work
let info list = if list.Length = 0 then "Vide" else "..." // 💥 Error FS0072...
let info list = if List.length list = 0 then "Vide" else $"{list.Length} éléments" // 👌
```

(2) Example:

```fsharp
let listKo = List.sortBy (fun x -> x.Length) ["three"; "two"; "one"]
//                                 ~~~~~~~~ 💥 Error FS0072: Lookup on object of indeterminate type...

// Solution 1: reverse the order by piping the list
let listOk = ["three"; "two"; "one"] |> List.sortBy (fun x -> x.Length)

// Solution 2: use the equivalent function
let listOk' = List.sortBy String.length ["three"; "two"; "one"]
```
