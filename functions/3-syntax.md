# Syntax

## Declaration

`let f x y = x + y + 1`

* Binding made with `let` keyword
* Binds both name (`f`) and parameters (`x` and `y`)
* Optional type annotations for parameters and/or returned value
  * `let f (x: int) (y: int) : int = ...`
* Otherwise, they will be inferred, eventually with an automatic generalization
* The function body is an expression:
  * → no need for a `return` keyword here.
  * → possibility to define intermediary variables and functions inside the body

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

Syntax: `fun parameter1 parameter2 etc. -> expression`

☝ **Note:**

* `fun` keyword mandatory
* Thin arrow `->` ≠ Bold arrow `=>` _(C♯, Js)_

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
Beware of useless lambdas:\
`List.map (fun x -> f x)` ❌\
`List.map f` ✅
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
* 📗 _Domain modelling made functional_ by Scott Wlaschin

```fsharp
type Add = int -> int -> int

let add: Add =
    fun x y ->
        x + y

// val add: x: int -> y: int -> int
```

{% hint style="info" %}
#### Parameter naming

* We can't provide the names of the parameters in the type alias. ❌
* But the parameters are named in the function signature. 👍
* To get named parameters in the type itself, we can use a `delegate` type or an `interface`. 📍
{% endhint %}

### Conversion to a Func

We can convert a lambda into a `Func<>` but explicitly by passing through its constructor:

```fsharp
let isNull = System.Func<_, _>(fun x -> x = null)
// val isNull: System.Func<'a,bool> when 'a: equality and 'a: null
```

### Conversion to a LINQ Expression 🚀

<details>

<summary>Conversion to a LINQ Expression 🚀</summary>

Contrary to C#, it's more complex in F♯ to write an `Expression<Func<>` from a lambda. You have to go through a _quotation_ and then convert it into an expression using either the `FSharp.Linq.RuntimeHelpers.LeafExpressionConverter` for simple cases (see example below), or the NuGet package `FSharp.Quotations.Evaluator` for more complex cases.

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

</details>

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
let ouiNon =
  function
  | true  -> "Oui"
  | false -> "Non"
// val ouiNon: _arg1: bool -> string
```

{% hint style="info" %}
#### A matter of taste

* For fans of point-free 📍
* More generally, in a succession of _pipes_ `arg |> f1 |> function ... |> f2...`\
  → The idea is to pattern match a large expression without storing it in an intermediate variable.
{% endhint %}

## Parameters deconstruction

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

This is also called _pattern matching_.\
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

{% hint style="success" %}
#### **Conclusion**

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

**Example:** find the number of steps to reach 1 in the [Collatz conjecture](https://en.wikipedia.org/wiki/Collatz_conjecture)

```fsharp
let rec steps (n: int) : int =
    if n = 1       then 0
    elif n % 2 = 0 then 1 + steps (n / 2)
    else                1 + steps (3 * n + 1)
```

## Tail recursion

* Type of recursion where the recursive call is the last instruction\
  🔗 [Tail recursion | Wikipedia](https://en.wikipedia.org/wiki/Tail_call)
* Detected by the compiler and rewritten into a simple loop\
  → ✅ Better performance\
  → ✅ Prevents stack overflow

### 💡 Accumulator argument

In general, we can transform a non-tail-recursive function into a tail-recursive one by adding an additional parameter, playing the role of "**accumulator**" like with `fold`/`reduce` functions - see [#versatile-aggregate-functions](../collections/3-common-functions.md#versatile-aggregate-functions "mention") 📍

**→ Example:** previous steps function rewritten as tail-recursive

{% code lineNumbers="true" %}
```fsharp
let steps (number: int) : int =
    [<TailCall>]
    let rec loop count n =
        if n = 1       then count
        elif n % 2 = 0 then loop (count + 1) (n / 2)
        else                loop (count + 1) (3 * n + 1)

    loop 0 number
```
{% endcode %}

{% hint style="info" %}
### Notes

1. Line 2: `TailCall` attribute was added in F♯ 8 to indicate (to the compiler and to the reader) that the function should be tail recursive.
2. Line 3: `loop` is a name idiomatic for this type of recursive inner-function. It's "accumulator" parameter is usually named `acc`, unless there is an obvious better name like `count` here.
3. Line 8: we start the "loop" with `count = 0`.
4. Lines 5 and 6: the recursive calls to `loop` are really the last call, hence the tail recursion.
5. We can verify that the function is compiled as a `while` loop in [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfA2gHgCoEMCWwDCmwwAfALoCwAUMAKYAuABAE40DGDY2TEdA8kwxg0wmAK7A6ANUKiaDAA4sY2Vpjpzg2HgwC8VBgYYBbNawAWDTdoDu2Omf2HEDZKQYBaYoOFiJ04LKGQcEhoaEA9OEMgLwbgBI7DADKAPZMdNg0jgbOAB4MICAMAPoM1mY0AHYKSipqcrmeDLlhzUGRMfHJqemZDM7F+cw02g2c3HwCQiLiUjJyijTKquqD2gxtcQwAgvLyNMDMAJesotzYYAyASYQMrEnlaeWyVEA=).
{% endhint %}

### 🚀 Continuation-passing style

Another common trick to write a tail-recursive function, although more advanced, is to accumulate values inside a series of continuation function calls.

**→ Example:** we can write `List.append` using this style:

{% code lineNumbers="true" %}
```fsharp
let append list1 list2 =
    let rec loop list cont =
        match list with
        | [] -> cont list2
        | head :: tail -> loop tail (fun rest -> cont (head :: rest))

    loop list1 id
```
{% endcode %}

{% hint style="info" %}
### Notes

* The `loop` inner function is recursively reading the `list1`, accessing the current item as the `head` and passing the `tail` to the next iteration of the loop.
* Line 4: when we reach the end of `list1`, we call the continuation function, called here `cont`.\
  &#xNAN;_&#x41;nother usual name for the continuation function parameter is `next`._
* Line 5: the `head` is captured/accumulated in the continuation function built with a lambda that takes a list called `rest`, put the `head` on top of it, and pass the resulting list to the previous version of the continuation function.
* Let's unwrap the continuation functions:\
  `0 ┬ fun rest0 -> id (list1[0] :: rest0)`\
  &#x20;  `└ fun rest0 -> list1[0] :: rest0`\
  &#x20;`1 ┬ fun rest1 -> (fun rest0 -> list1[0] :: rest0)(list1[1] :: rest1)`\
  &#x20;  `└ fun rest1 -> list1[0] :: list1[1] :: rest1`\
  &#x20;`...`\
  &#x20;`N ┬ fun restN -> list1[0] :: list1[1] :: ... :: list1[N] :: restN`\
  &#x20;  `└ list1[0] :: list1[1] :: ... :: list1[N] :: list2`
* The real implementation of `List.append` is different and is optimized using other techniques including mutation.
{% endhint %}

🔗 [Continuation-passing style | Wikipedia](https://en.wikipedia.org/wiki/Continuation-passing_style)\
🔗 [Understanding continuations | fsharp for fun and profit](https://fsharpforfunandprofit.com/posts/computation-expressions-continuations/#continuation-passing-style)&#x20;

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
### Warning

* A function cannot be overloaded!
* Each version of the functions should have a dedicated and unique name.
{% endhint %}

**Example:**

* LINQ `Select` method has an overload taking the current index as additional parameter
* The equivalent in the `List` module are the 2 functions `map` and `mapi`:\
  `List.map  (mapping:                 'T -> 'U) (items: 'T list) : 'U list`\
  &#x20;`List.mapi (mapping: (index: int) -> 'T -> 'U) (items: 'T list) : 'U list`\
  → See  [#map-vs-mapi](../collections/3-common-functions.md#map-vs-mapi "mention") 📍

## Template function

Create specialized "overloads"

→ **Example:** helper wrapping `String.Compare`:

```fsharp
type ComparisonResult = Bigger | Smaller | Equal // Union type 📍

let private compareTwoStrings (comparison: StringComparison) string1 string2 =
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

{% hint style="warning" %}
Note that `compareTwoStrings` is declared as `private`: to hide `implementation details`.
{% endhint %}

## Parameter order

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

> **Inline extansion** ou **inlining :** compiler optimization to replace a function call with the its body.\
> 🔗 [Inline expansion | Wikipedia](https://en.wikipedia.org/wiki/Inline_expansion)

* Performance gain 👍
* Longer compilation ⚠️

💡 Same as the refactoring [_Inline Method_](https://refactoring.guru/inline-method)

### `inline` keyword

* Tells the compiler to _"inline"_ the function
* Typical usage: small "syntactic sugar" function/operator

**Examples from** [**FSharp.Core**](https://github.com/dotnet/fsharp/blob/main/src/fsharp/FSharp.Core/prim-types.fs)**:**

```fsharp
let inline ignore _ = ()
let inline (|>) x f = f x

let t = true |> ignore
//   ~= ignore true // After the inlining of |>
//   ~= ()          // After the inlining of ignore
```

The other uses of `inline` functions relate to SRTP 📍
