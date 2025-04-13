# First concepts

## Expression vs Statement

> A **statement** will produce a side effect.\
> An **expression** will produce a value... and a possible side effect (that we should avoid).

* F♯ is a functional, expression-based language only.
* In comparison, C♯ is an imperative language, based on statements, but includes more and more syntactic sugar based on expressions:
  * Ternary operator `b ? x : y`
  * [Null-coalescing operator](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator) `??` in C♯ 8 : `label ?? "(Empty)"`
  * [Expression-bodied members](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/expression-bodied-members) in C♯ 6 and 7
  * `switch` expression in C♯ 8

### ⚖️ Benefits of expressions over instructions

* **Conciseness**: less visual clutters → more readable
* **Composability**: composing expressions is like composing values
* **Understanding**: no need to know the previous instructions to understand the current one
* **Testability**: pure expressions[^1] are easier to test
  * _Predictable_: same inputs mean same outputs
  * _Isolated_: shorter _Arrange/Setup_ phase in tests, no need for mocks

## Everything is an expression

* A function is declared and behaves like a value
  * We can pass it as parameter or return it from another function (1)
* The _control flow_ building blocks are also expressions
  * `if … then/else` , `match … with`
  * `for … in`, `for … to`, `while … do` just return "nothing" (2)

{% hint style="info" %}
#### Notes

* (1) See _1st-class citizens, high-order functions_ 📍
* (2) Except in _collection comprehensions_ 📍
{% endhint %}

**Consequences**

* No `void`\
  → Best replaced by the `unit` type 📍
* No _**Early Exit**_
  * In C#, you can exit a function with `return` and exit a `for/while` loop with `break`.
  * In F♯ these keywords do not exist.

### Early exit alternatives

The most questionable solution is to raise an exception 💩 (see [StackOverflow](https://stackoverflow.com/a/42018355/8634147))

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
[<TailCall>] // F♯ 8 📍
let rec firstOr defaultValue predicate list =
    match list with
    | [] -> defaultValue                                // 👈 Exit
    | x :: _ when predicate x -> x                      // 👈 Exit
    | _ :: rest -> firstOr defaultValue predicate rest  // 👈 Recursive call to continue

let test1 = firstOr -1 (fun x -> x > 5) [1]     // -1
let test2 = firstOr -1 (fun x -> x > 5) [1; 6]  // 6
```

## Typing, inference and ceremony

The ceremony is correlated to the typing weakness

<table><thead><tr><th width="146">Language</th><th>Typing strength</th><th>Inference strength</th><th>Ceremony</th></tr></thead><tbody><tr><td>JS</td><td>Low<br><em>(dynamic typing)</em></td><td>×</td><td>Low</td></tr><tr><td>C♯</td><td>Medium (static nominal)</td><td>Low</td><td>High</td></tr><tr><td>TS</td><td>Strong (static structural + ADT)</td><td>Medium</td><td>Medium</td></tr><tr><td>F♯</td><td>Strong (static nominal + ADT)</td><td>High</td><td>Low</td></tr></tbody></table>

🔗 [Zone of Ceremony](https://blog.ploeh.dk/2019/12/16/zone-of-ceremony/) _by Mark Seemann_

## Type inference

Goal: write type annotations as little as possible

* Less code to write 👍
* Compiler ensures consistency
* IntelliSense helps with coding and reading

### Type inference in C♯

* Method parameters and return value ❌
* Variable declaration: `var o = new { Name = "John" }` ✔️
* Lambda as argument: `list.Find(i => i == 5)` ✔️
* Lambda declaration: `var f3 = () => 1;` ✔️ in C# 10 _(limited)_
* Array initialisation: `var a = new[] { 1, 2 };` ✔️
* Generic classes:
  * constructor: `new Tuple<int, string>(1, "a")` ❌
  * static helper class: `Tuple.Create(1, "a")` ✔️
* C♯ 9 _target-typed expression_ `StringBuilder sb = new();` ✔️

### Type inference in F♯

[Hindley–Milner](https://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system) method

* Able to deduce the type of variables, expressions and functions in a program without any type annotation
* Based on both the implementation and the usage

**Example:**

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

{% hint style="info" %}
#### Generic type parameter notation

* starts with an apostrophe `'` _(a.k.a. tick)_
* can be in camelCase (`'a`) or PascalCase (`'A`)
* C♯ `TXxx` → F♯ `'xxx` or `'Xxx`
{% endhint %}

### Inference _vs_ type annotation

* Pros:
  * code terser
  * automatic generalization
* Cons:
  * we can break code in cascade
  * inference limited:
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

// Solution 2: use a named function  instead of a lambda
let listOk' = List.sortBy String.length ["three"; "two"; "one"]
```

[^1]: with no side-effects
