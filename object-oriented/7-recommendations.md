---
description: Recommendations for object-oriented programming
---

# Recommendations

## No object orientation where F♯ is good

Type inference works better with `function object` than `object.Member`

### Simple object hierarchy

❌ Avoid inheritance

✅ Prefer type _Union_ and exhaustive _pattern matching_

{% hint style="info" %}
#### Recursive types

This is particularly true for recursive types. You can define a `fold` function for them.

🔗 [The "Recursive types and folds" series](https://fsharpforfunandprofit.com/series/recursive-types-and-folds/), _F# for fun and profit_
{% endhint %}

### Structural equality

❌ Avoid class _(reference equality by default)_

✅ Prefer a _Record_ or a _Union_

👌 Alternatively, consider _Struct_ for performance purposes

❓ Consider custom structural equality for performance purposes\
&#x20;     🔗 [Custom Equality and Comparison in F#](https://www.compositional-it.com/news-blog/custom-equality-and-comparison-in-f/), _Compositional IT_

## Object-oriented recommended use-cases

1. Encapsulate mutable state → in a class
2. Group features → in an interface
3. Expressive, user-friendly API → tuplified methods
4. F♯ API consumed in C♯ → member extensions
5. Dependency management → injection into the constructor
6. Tackle higher-order functions limits

### Class to encapsulate mutable state

```fsharp
// 😕 Encapsulate mutable state in a closure → impure function → counter-intuitive ⚠️
let counter =
    let mutable count = 0
    fun () ->
        count <- count + 1
        count

let x = counter ()  // 1
let y = counter ()  // 2

// ✅ Encapsulate mutable state in a class
type Counter() =
    let mutable count = 0 // Private field
    member _.Next() =
        count <- count + 1
        count
```

### Interface grouping features

```fsharp
let checkRoundTrip serialize deserialize value =
    value = (value |> serialize |> deserialize)
// val checkRoundTrip :
//   serialize:('a -> 'b) -> deserialize:('b -> 'a) -> value:'a -> bool
//     when 'a : equality
```

`serialize` and `deserialize` form a consistent group\
→ Grouping them in an object makes sense

```fsharp
let checkRoundTrip serializer data =
    data = (data |> serializer.Serialize |> serializer.Deserialize)
```

💡 Prefer an interface to a _Record_ _(not possible with `Fable.Remoting`)_

```fsharp
// ❌ Avoid: not a good use of a Record: unnamed parameters, structural comparison lost...
type Serializer<'T> = {
    Serialize: 'T -> string
    Deserialize: string -> 'T
}

// ✅ Recommended
type Serializer =
    abstract Serialize<'T> : value: 'T -> string
    abstract Deserialize<'T> : data: string -> 'T
```

* Parameters are named in the methods
* Object easily instantiated with an object expression

### User-friendly API

```fsharp
// ✖️ Avoid                         // ✅ Favor
module Utilities =                  type Utilities =
    let add2 x y = x + y                static member Add(x,y) = x + y
    let add3 x y z = x + y + z          static member Add(x,y,z) = x + y + z
    let log x = ...                     static member Log(x, ?retryPolicy) = ...
    let log' x retryPolicy = ...
```

Advantages of OO implementation:

* `Add` method overloaded _vs_ `add2`, `add3` functions _(`2` and `3` = args count)_
* Single `Log` method with `retryPolicy` optional parameter

🔗 [F♯ component design guidelines - Libraries used in C♯](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/component-design-guidelines#guidelines-for-libraries-for-use-from-other-net-languages)

### F♯ API consumed in C♯

Do not expose this type as is:

```fsharp
type RadialPoint = { Angle: float; Radius: float }

module RadialPoint =
    let origin = { Angle = 0.0; Radius = 0.0 }
    let stretch factor point = { point with Radius = point.Radius * factor }
    let angle (i: int) (n: int) = (float i) * 2.0 * System.Math.PI / (float n)
    let circle radius count =
        [ for i in 0..count-1 -> { Angle = angle i count; Radius = radius } ]
```

💡 To make it easier to discover the type and use its features in C♯

* Put everything in a namespace
* Augment type with the functionalities implemented in the companion module

```fsharp
namespace Fabrikam

type RadialPoint = {...}
module RadialPoint = ...

type RadialPoint with
    static member Origin = RadialPoint.origin
    static member Circle(radius, count) = RadialPoint.circle radius count |> List.toSeq
    member this.Stretch(factor) = RadialPoint.stretch factor this
```

👉 The API consumed in C♯ is \~equivalent to:

```csharp
namespace Fabrikam
{
    public static class RadialPointModule { ... }

    public sealed record RadialPoint(double Angle, double Radius)
    {
        public static RadialPoint Origin => RadialPointModule.origin;

        public static IEnumerable<RadialPoint> Circle(double radius, int count) =>
            RadialPointModule.circle(radius, count);

        public RadialPoint Stretch(double factor) =>
            new RadialPoint(Angle@, Radius@ * factor);
    }
}
```

## Dependency management

### FP based technique

> **Parametrization of dependencies + partial application**

* Small-dose approach: few dependencies, few functions involved
* Otherwise, quickly tedious to implement and to use

```fsharp
module MyApi =
    let function1 dep1 dep2 dep3 arg1 = doStuffWith dep1 dep2 dep3 arg1
    let function2 dep1 dep2 dep3 arg2 = doStuffWith' dep1 dep2 dep3 arg2
```

### OO technique

> **Dependency injection**

* Inject dependencies into the class constructor
* Use these dependencies in methods

👉 Offers a user-friendly API 👍

```fsharp
type MyParametricApi(dep1, dep2, dep3) =
    member _.Function1 arg1 = doStuffWith dep1 dep2 dep3 arg1
    member _.Function2 arg2 = doStuffWith' dep1 dep2 dep3 arg2
```

✅ Particularly recommended for encapsulating **side-effects** :\
→ Connecting to a DB, reading settings...

{% hint style="warning" %}
#### Trap

Dependencies injected in the constructor make sense only if they are used throughout the class.

A dependency used in a single method indicates a design smell.
{% endhint %}

### Advanced FP techniques

_Dependency rejection_ = sandwich pattern

* Reject dependencies in Application layer, out of Domain layer
* Powerful and simple 👍
* ... when suitable ❗

_Free_ monad + interpreter patter

* Pure domain
* More complex than the sandwich pattern but working in any case
* User-friendly through a dedicated computation expression

_Reader_ monad

* Only if hidden inside a computation expression

...

🔗 [Six approaches to dependency injection](https://fsharpforfunandprofit.com/posts/dependencies/), F# for Fun and Profit, Dec 2020

## Higher-order function limits

☝️ It's better to pass an object than a lambda as a parameter to a higher-order function when:

### 1. Lambda arguments are not explicit

❌ `let test (f: float -> float -> string) =...`

✅ Solution 1: type wrapping the 2 args `float`\
→ `f: Point -> string` with `type Point = { X: float; Y: float }`

✅ Solution 2: interface + method for named parameters\
→ `type IPointFormatter = abstract Execute : x:float -> y:float -> string`

### 2. Lambda is a **command** `'T -> unit`

✅ Prefer to trigger a side-effect via an object\
→ `type ICommand = abstract Execute : 'T -> unit`

### 3. Lambda: "really" generic!?

```fsharp
let test42 (f: 'T -> 'U) =
    f 42 = f "42"
// ❌ ^^     ~~~~
// ^^ Warning FS0064: This construct causes code to be less generic than indicated by the type annotations.
//                    The type variable 'T has been constrained to be type 'int'.
// ~~ Error FS0001: This expression was expected to have type 'int' but here has type 'string'
// 👉 `f: int -> 'U'` expected
```

✅ Solution: wrap the function in an object

```fsharp
type Func2<'U> =
    abstract Invoke<'T> : 'T -> 'U

let test42 (f: Func2<'U>) =
    f.Invoke 42 = f.Invoke "42"
```
