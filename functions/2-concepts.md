# Concepts

## Currying

### Definition

Consists in transforming :

* a function taking N parameters \
  `Func<T1, T2, ...Tn, TReturn>` in C♯
* into a chain of N functions taking 1 parameter \
  `Func<T1, Func<Tn, ...Func<Tn, TReturn>>`

### Partial application

Calling a function with fewer arguments than its number of parameters.

* Possible thanks to currying
* Returns a function taking the remaining number of arguments as parameters.

```fsharp
// Template function with 2 parameters
let insideTag (tagName: string) (content: string) =
    $"<{tagName}>{content}</{tagName}>"

// Helpers with a single parameter `content`
// `tagName` is fixed by partial application
let emphasize = insideTag "em"     // `tagName` fixed to "em"
let strong    = insideTag "strong" // `tagName` fixed to "strong"

// Equivalent less elegant but more explicit
let em content = insideTag "em" content
```

:warning: **Caution :** partial application impacts the signature:

```fsharp
val insideTag: tagName: string -> content: string -> string
val emphasize: (string -> string)  // 👈 (1)(2)
val em: content: string -> string
```

1. Loss of parameter names (`content: string` becomes just `string`)
2. Signature of a function value, hence the parentheses (`(string -> string)`)

### Syntax of F♯ functions

Parameters separated by spaces:

* Indicates that functions are curried
* Hence the `->` in the signature between parameters

```fsharp
let fn () = result         // unit -> TResult
let fn arg = ()            // T    -> unit
let fn arg = result        // T    -> TResult

let fn x y = (x, y)        // T1 -> T2 -> (T1 * T2)

// Equivalent, explicitly curried:
let fn x = fun y -> (x, y) // 1. With a lambda
let fn x =                 // 2. With a named sub-function
    let fn' y = (x, y)     // 👈 `x` captured from the outer scope
    fn'
```

### IntelliSense with Ionide

💡 In VsCode with Ionide, IntelliSense provides a more readable description of functions, putting each argument in a new line:

```fsharp
let pair x y = (x, y)
// val pair:
//    x: 'a ->
//    y: 'b
//    -> 'a * 'b

let triplet x y z = (x, y, z)
// val triplet:
//    x: 'a ->
//    y: 'b ->
//    z: 'c
//    -> 'a * 'b * 'c
```

### .NET compilation of a curried function

☝ A curried function is compiled differently depending on how it's called!

* Basically, it is compiled as a method with tuple-parameters \
  → Viewed as a regular method when consumed in C♯

Example: F♯ then C♯ equivalent, based on a simplified version from [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjoA3esAx4iuAEy5EAPgZNcARnJA===)_ :

```fsharp
module A =
    let add x y = x + y
    let value = 2 |> add 1
```

```csharp
public static class A
{
    public static int add(int x, int y) => x + y;
    public static int value => 3;
}
```

* When partially applied, the function is compiled as a pseudo `Delegate` class extending `FSharpFunc<int, int>` with an `Invoke` method that encapsulates the 1st supplied arguments.

Example : F♯ then C♯ equivalent, based on a simplified version from [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjqMYAeQB2eIgya4AjEA===)_:

```fsharp
module A =
    let add x y = x + y
    let addOne = add 1
```

```csharp
public static class A
{
    internal sealed class addOne@3 : FSharpFunc<int, int>
    {
        internal static readonly addOne@3 @_instance = new addOne@3();

        internal addOne@3() { }

        public override int Invoke(int y) => 1 + y;
    }

    public static FSharpFunc<int, int> addOne => addOne@3.@_instance;

    public static int add(int x, int y) => x + y;
}
```

## Unified function design

`unit` type and currying make it possible to design functions simply as:

* **Takes a single parameter** of any type
  * including `unit` for a “parameterless” function
  * including another _(callback)_ function
* **Returns a single value** of any type
  * including `unit` for a “return nothing” function
  * including another function

👉 **Universal signature** of a function in F♯: `'T -> 'U`.

## Parameters order

Between C♯ and F♯, the parameter concerning the main object (the `this` in case of a method) is not placed in the same place:

* In a method extension C♯, the `this` object is the 1st parameter.
  * E.g. `items.Select(x => x)`
* In F♯, the main object is _rather_ the **last parameter**:\
  _(it's called the **data-last style**)_
  * E.g. `List.map (fun x -> x) items`

The _data-last_ style favors :

* Pipeline: `items |> List.map square |> List.sum`
* Partial application: `let sortDesc = List.sortBy (fun i -> -i)`
* Composition of partially applied functions up to param “_data_”.
  * `(List.map square) >> List.sum`

⚠️ There can be some friction .NET/BCL methods because the BCL is also _data-first_ driven. The solution is to wrap the method in a new curried function having parameters sorted in an order more F♯ piping friendly.

```fsharp
let startsWith (prefix: string) (value: string) =
    value.StartsWith(prefix)
```

💡 Tips

Prefer to put **1st** the most static parameters = those likely to be predefined by partial applications.

E.g.: “dependencies” that would be injected into an object in C♯.

👉 Partial application is a way to implement the dependency injection in F♯.
