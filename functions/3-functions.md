# Functions

## Binding of a function

`let f x y = x + y + 1`

* Binding made with `let` keyword
* Binds both name (`f`) and parameters (`x` and `y`)
* Optional type annotations for parameters and/or return
  * `let f (x: int) (y: int) : int = ...`
  * Otherwise, type inference, with possible automatic generalization
* Last expression = function return value
* Possible definition of (non-generic) sub-functions

## Generic function

* In many cases, inference works with automatic generalization
  * `let listOf x = [x]` → `(x: 'a) -> 'a list`
* Explicit inline annotation of generic parameters
  * `let f (x: 'a) = ...`
* Explicit inline annotation with generic type inference
  * `let f (list: list<_>) = ...`
* Full explicit annotation of generic parameters
  * `let f<'a> (x: 'a) = ...`
  * Pros: callers can specify the type parameter

## Anonymous / Lambda function

Expression defining a function

Syntax: `fun parameter1 parameter2 etc -> expression`

☝ **Note:**

* `fun` keyword mandatory
* Thin arrow `->` _(like Java)_ ≠ Bold arrow `=>` _(C♯, Js)_

### Lambda: usage examples

#### **1.** As an argument to a _high-order function_

* To avoid having to define a named function
* Recommended for a short function, to keep it readable

```fsharp
[1..10] |> List.map (fun i -> i + 1) // 👈 () around the lambda

// Alternative with a named function
let add1 i = i + 1
[1..10] |> List.map add1
```

{% hint style="warning" %}
Beware of useless lambdas: `List.map (fun x -> f x)` ≡ `List.map f`
{% endhint %}

#### **2.** In a `let` binding with inference

* To make it explicit when the function returns a function
* A kind of manual currying
* Use sparingly ❗

```fsharp
let add x y = x + y                     // Regular version: auto curried
let add' x = fun y -> x + y             // 1 nested lambda
let add'' = fun x -> (fun y -> x + y)   // 2 nested lambdas
```

#### **3.** In an annotated let binding

The function signature is pre-defined using a type alias

* Force implementation to follow signature
* Widely used in 📗 _Domain modelling made functional_ by Scott Wlaschin

```fsharp
type Add = int -> int -> int

let add: Add =
    fun x y ->
        x + y

// val add: x: int -> y: int -> int
// ☝ Parameters are sont nommés 👍
```

{% hint style="info" %}
- We can't provide the names of the parameters in the type alias. ❌
- But the parameters are named in the function signature. 👍
- To get named parameters in the type itself, we can use a `delegate` type or an `interface`. 📍
{% endhint %}

### Conversion to a Func

We can convert a lambda into a `Func<>` but explicitly by passing through its constructor:

```fsharp
let isNull = System.Func<_, _>(fun x -> x = null)
// val isNull: System.Func<'a,bool> when 'a: equality and 'a: null
```

### Conversion to LINQ Expression

It's a bit more complex to write an `Expression<Func<>` from a lambda. You have to go through a _quotation_ [^quotation] and then convert it into an expression using either the `FSharp.Linq.RuntimeHelpers.LeafExpressionConverter` for simple cases (see example below), or the NuGet package `FSharp.Quotations.Evaluator` for more complex cases.

[^quotation]: The _quotations_ are out of scope. Just 2 words about them:
    - They are written between `<@ ... @>`.
    - They're a bit like C# `Expression`
    - more info [here](https://stackoverflow.com/a/8134113/8634147).

```fsharp
open System
open System.Linq.Expressions
open Microsoft.FSharp.Linq.RuntimeHelpers

let isNullQuot = <@ Func<_, _> (fun (x: obj) -> x = null) @>
// val isNullQuot: Quotations.Expr<System.Func<obj,bool>> =
//   NewDelegate (Func`2, x, Call (None, op_Equality, [x, Value (<null>)]))

let isNullExpr =
    isNullQuot
    |> LeafExpressionConverter.QuotationToExpression 
    |> unbox<Expression<Func<obj, bool>>>
// val isNullExpr: System.Linq.Expressions.Expression<System.Func<obj,bool>> =
//   x => (x == null)
```

### `function` keyword

* Define an anonymous function with an implicit `match` expression surrounding the body.
* Short syntax equivalent to `fun x -> match x with`.
* Takes 1 parameter which is implicit

```fsharp
let ouiNon x =
  match x with
  | true  -> "Oui"
  | false -> "Non"
// val ouiNon: x: bool -> string

// 👉 Same with `function`:
let ouiNon = function
  | true  -> "Oui"
  | false -> "Non"
// val ouiNon: _arg1: bool -> string
```

{% hint style="info" %}
## A matter of taste
- For fans of point-free 📍
- More generally, in a succession of _pipes_ `arg |> f1 |> function ... |> f2...` \
→ The idea is to pattern match a large expression without storing it in an intermediate variable.
{% endhint %}

## Parameters deconstructing

* As in JavaScript, you can deconstruct _inline_ a parameter.
* This is also a way of indicating the type of parameter.
* The parameter appears without a name in the signature.

Example with a _Record_ type 📍

```fsharp
type Person = { Name: string; Age: int }

let name { Name = x } = x     // Person -> string
let age { Age = x } = x       // Person -> int
let age' person = person.Age  // Alternative w/o deconstruction

let bob = { Name = "Bob"; Age = 18 } // Person
let bobAge = age bob // int = 18
```

This is also called _pattern matching_.
→ But I prefer to reserve this expression to `match x with ...`

## Tuple parameter

* As in C♯, you may want to group function parameters together.
  * For the sake of cohesion, when these parameters form a whole.
  * To avoid the [long parameter list](https://refactoring.guru/smells/long-parameter-list) _code smell._
* They can be grouped together in a tuple and even deconstructed.

```fsharp
// V1 : too many parameters
let f x y z = ...

// V2 : parameters grouped in a tuple
let f params =
    let (x, y, z) = params
    ...

// V3 : idem with tuple deconstructed on site
let f (x, y, z) = ...
```

* `f (x, y, z)` looks a lot like a C♯ method!
* The signature indicates the change: `(int * int * int) -> TResult`.
  * The function now only has only 1 parameter instead of 3.
  * Loss of partial application of each element of the tuple.

{% hint style="info" %}
## ☝ **Conclusion**

* Resist the temptation to use a tuple all the time _(because it's familiar, a habit from C♯)_
* Use it only when it makes sense to group parameters together.
* If relevant, even use a dedicated (anonymous) record type. 📍
{% endhint %}

## Recursive function

* Function that calls itself
* Special syntax with keyword `rec` otherwise error `FS0039: ... is not defined`
* Very common in F♯ to replace `for` loops
  * Because it's often easier to design
  * And still performant thanks to tail recursion _(details just after)_

Example: find the number of steps to reach 1 in the [Collatz conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture)

```fsharp
let rec steps (n: int) : int =
    if n = 1       then 0
    elif n % 2 = 0 then 1 + steps (n / 2)
    else                1 + steps (3 * n + 1)
```

## Tail recursion

* Type of recursion where the recursive call is the last instruction
* Detected by the compiler and optimized as a loop
  * Prevents stack overflow
* The usual way of making tail recursive is:
  * Add an "accumulator" parameter, just like with `fold`/`reduce` functions. 📍

```fsharp
let steps (number: int) : int =
    let rec loop count n = // 👈 `loop` = idiomatic name for this type of recursive internal function
        if n = 1       then count
        elif n % 2 = 0 then loop (count + 1) (n / 2)      // 👈 Last call is `loop` -> Tail recursive
        else                loop (count + 1) (3 * n + 1)  // 👈 Same
    loop 0 number // 👈 Start the loop with 0 as the initial value for `count`
```

## Mutually recursive functions

* Functions that call each other
* Must be declared together:
  * 1st function indicated as recursive with `rec`.
  * other functions added to the declaration with `and`.

```fsharp
// ⚠️ Convoluted algo, just for illustration purposes

let rec Even x =        // 👈 Keyword `rec`
    if x = 0 then true
    else Odd (x-1)      // 👈 Call to `Odd` defined below
and Odd x =             // 👈 Keyword `and`
    if x = 0 then false
    else Even (x-1)     // 👈 Call to `Even` defined above
```

## Function overload

{% hint style="warning" %}
- A function cannot be overloaded!
- Each version should have a dedicated name.
{% endhint %}

Example:

* `List.map  (mapping:                 'T -> 'U) (items: 'T list) : 'U list`
* `List.mapi (mapping: (index: int) -> 'T -> 'U) (items: 'T list) : 'U list`

## Template function

Create specialized "overloads":

```fsharp
type ComparisonResult = Bigger | Smaller | Equal

// Fonction template, 'private' pour la "cacher"
let private compareTwoStrings (comparison: StringComparison) string1 string2 =
//  ^^^^^^^ private to hide it (implementation details)
    let result = System.String.Compare(string1, string2, comparison)
    if result > 0 then
        Bigger
    elif result < 0 then
        Smaller
    else
        Equal

// Partial application of the 'comparison' parameter
let compareCaseSensitive   = compareTwoStrings StringComparison.CurrentCulture
let compareCaseInsensitive = compareTwoStrings StringComparison.CurrentCultureIgnoreCase
```

☝ Parameter order

The additional parameter is placed at a different location in C♯ and F♯:

* Comes last in C♯:

```csharp
String.Compare(String, String, StringComparison)
String.Compare(String, String)
```

* Comes first in F♯, to enable its partial application:

```fsharp
compareTwoStrings    : StringComparison -> String -> String -> ComparisonResult
compareCaseSensitive :                     String -> String -> ComparisonResult
```

## `inline` function

### Principle

> **Inline extansion** ou **inlining :** compiler optimization to replace a function call with the its body. \
> 🔗 [Inline expansion | Wikipedia](https://en.wikipedia.org/wiki/Inline_expansion)

* Performance gain 👍
* Longer compilation ⚠️

💡 Same as the refactoring [_Inline Method_](https://refactoring.guru/inline-method)

### `inline` keyword

- Tells the compiler to _"inline"_ the function
- Typical usage: small "syntactic sugar" function/operator

Examples from [FSharp.Core](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs): \
→ Function `ignore`  📍 TODO: fonctions-standard.md \
→ Operator `|>`      📍 TODO: operateur-pipe-right-or-greater-than

```fsharp
let inline ignore _ = ()
let inline (|>) x f = f x

let t = true |> ignore
//   ~= ignore true // After the inlining of |>
//   ~= ()          // After the inlining of ignore
```

The other use of `inline` functions relates to SRTP 📍
