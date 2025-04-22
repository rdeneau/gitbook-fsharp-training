# Interface

## Syntax

Similar to an abstract class but with only abstract members.

**Atttribute:**

* `[<AbstractClass>]` attribute can not be used to declare an interface
* `[<Interface>]` attribute is optional but recommended in most cases

```fsharp
type [accessibility-modifier] interface-name =
    abstract memberN : [ argument-typesN -> ] return-typeN
```

* Interface name begins with `I` to follow .NET convention
* Arguments can be named _(without parentheses otherwise 💥)_

```fsharp
[<Interface>]
type IPrintable =
    abstract member Print : format: string -> unit
```

You can also use verbose syntax with an `interface ... end` block.\
→ Not idiomatic except in the case of a member-less interface a.k.a _marker interface_.

```fsharp
type IMarker = interface end
```

## Implementation

2 ways of implementing an interface:

1. In an object expression 📍
2. In a type _(as in C♯)_

```fsharp
type IPrintable =
    abstract member Print : unit -> unit

type Range = { Min: int; Max: int } with
    interface IPrintable with
        member this.Print() = printfn $"[{this.Min}..{this.Max}]"
```

{% hint style="warning" %}
#### Keyword

This `interface` F♯ keyword matches the `implements` keyword in Java and TypeScript.
{% endhint %}

## Default implementation

F♯ 5.0 supports interfaces defining methods with default implementations written in C♯ 8+ but does not allow them to be defined directly in F♯.

{% hint style="warning" %}
#### Keyword

Don't confuse with `default` keyword: it's supported only in classes!
{% endhint %}

## F♯ interface is explicit

F♯ interface implementation\
≡ Explicit implementation of an interface in C♯

→ Interface methods are accessible only by _upcasting_:

```fsharp
[<Interface>]
type IPrintable =
    abstract member Print : unit -> unit

type Range = { Min: int; Max: int } with
    interface IPrintable with
        member this.Print() = printfn $"[{this.Min}..{this.Max}]"

let range = { Min = 1; Max = 5 }
(range :> IPrintable).Print()  // upcast operator
// [1..5]
```

## Implementing a generic interface

```fsharp
[<Interface>]
type IValue<'T> =
    abstract member Get : unit -> 'T

// Contrived example for demo purpose
type DoubleValue(i, s) =
    interface IValue<int> with
        member _.Get() = i

    interface IValue<string> with
        member _.Get() = s

let o = DoubleValue(1, "hello")
let i = (o :> IValue<int>).Get() // 1
let s = (o :> IValue<string>).Get() // "hello"
```

## Inheritance

Defined with `inherit` keyword

```fsharp
type A(x: int) =
    do
        printf "Base (A): "
        for i in 1..x do printf "%d " i
        printfn ""

type B(y: int) =
    inherit Base(y * 2) // 👈
    do
        printf "Child (B): "
        for i in 1..y do printf "%d " i
        printfn ""

let child = B(1)
// Base: 1 2
// Child: 1
// val child: B
```
