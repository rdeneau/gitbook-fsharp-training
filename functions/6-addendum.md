# Addendum

## Memoization

💡 **Idea:** reduce the calculation time of a function

❓ **How to:** cache results\
→ next call with same arguments, return cached result

👉 **In practice :** function `memoizeN` from library [FSharpPlus](https://fsprojects.github.io/FSharpPlus/reference/fsharpplus-memoization.html#memoizeN)

⚠️ **Caution :** As with any optimization, use it when you need it and check (measure) that it works without any additional inconvenience.

☝ Not to be confused with `lazy` expression...

## Lazy expression

Syntax sugar to create a .NET `Lazy<'T>` object from an expression

* Expression not evaluated immediately but on 1st request _(_[_Thunk_](https://en.wikipedia.org/wiki/Thunk)_)_
* Interesting for improving performance without overcomplexifying the code

```fsharp
let printAndForward x = printfn $"{x}"; x

let a = lazy (printAndForward "a")

let b = printAndForward "b"
// > b

printfn $"{a.Value} and {b}"
// > a
// > a and b

printfn $"{a.Value} and c"
// > a and c
```

### `Lazy` active pattern

Extracts the value from a `Lazy` object

```fsharp
let triple (Lazy i) = 3 * i // Lazy<int> -> int

let v =
    lazy
        (printf "eval!"
         5 + 5) // Lazy<int>

triple v
// eval!val it: int = 30

triple v
// val it: int = 30
```

## Organizing functions

3 ways to organize functions = 3 places to declare them:

* _Module_: function declared within a module 📍
* _Nested_ : function declared inside a value or inside another function
  * 💡 Encapsulate helpers used just locally
  * ☝ Parent function parameters accessible to function _nested_
* _Method_ : function defined as a method in a type...

## Methods

* Defined with the `member` keyword rather than `let`
* Choice of the _self-identifier_: `this`, `me`, `self`, `_`...
* Choice of the parameters style:
  * Tuplified: OOP style
  * Curried: FP style

### Method Examples

```fsharp
type Product =
    { SKU: string; Price: float }

    // Tuple style, `this`
    member this.TupleTotal(qty, discount) =
        (this.Price * float qty) - discount

    // Curried style, `me`
    member me.CurriedTotal qty discount =
        (me.Price * float qty) - discount
```

## Function _vs_ Method

| Feature                           | Function         | Method                                                |
| --------------------------------- | ---------------- | ----------------------------------------------------- |
| Naming convention                 | camelCase        | PascalCase                                            |
| Currying                          | ✅ yes            | ✅ if not tuplified nor overridden                     |
| Named parameters                  | ❌ no             | ✅ if tuplified                                        |
| Optional parameters               | ❌ no             | ✅ if tuplified                                        |
| Overload                          | ❌ no             | ✅ if tuplified                                        |
| Parameter inference (declaration) | ➖ Possible       | ➖ yes for `this`, possible for the other parameters   |
| Argument inference (usage)        | ✅ yes            | ❌ no, object type annotation needed                   |
| High-order function argument      | ✅ yes            | ➖ yes with shorthand member, no with lambda otherwise |
| `inline` supported                | ✅ yes            | ✅ yes                                                 |
| Recursive                         | ✅ yes with `rec` | ✅ yes                                                 |

## Interop with the BCL[^1]

### void method

A .NET `void` method is seen in F# as returning `unit`.

```fsharp
let list = System.Collections.Generic.List<int>()
list.Add
(* IntelliSense Ionide:
abstract member Add:
   item: int
      -> unit
*)
```

Conversely, an F# function returning `unit` is compiled into a `void` method.

```fsharp
let ignore _ = ()
```

Equivalent C# based on [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfYBTALgAgJYHMB2A9gE4oYD6GAvBgBQCUQA==):

```csharp
public static void ignore<T>(T _arg) {}
```

### Calling a BCL method with N arguments

A .NET method with several arguments is "pseudo-tuplified":

* All arguments must be specified (1)
* Partial application of parameters is not supported (2)
* Calls don't work with a real F♯ tuple ⚠️ (3)

```fsharp
System.String.Compare("a", "b") // ✅ (1)
System.String.Compare "a","b"   // ❌
System.String.Compare "a"       // ❌ (2)

let tuple = ("a","b")
System.String.Compare tuple     // ❌ (3)
```

### `out` Parameter - In C♯

`out` used to have multiple output values from a method\
→ Ex : `Int32.TryParse`, `Dictionary<,>.TryGetValue` :

```csharp
if (int.TryParse(maybeInt, out var value))
    Console.WriteLine($"It's the number {value}.");
else
    Console.WriteLine($"{maybeInt} is not a number.");
```

### `out` Parameter - In F♯

Output can be consumed as a tuple 👍

```fsharp
  match System.Int32.TryParse maybeInt with
  | true, i  -> printf $"It's the number {value}."
  | false, _ -> printf $"{maybeInt} is not a number."
```

### Instantiate a class with `new`?

```fsharp
// (1) new allowed but not recommended
type MyClass(i) = class end

let c1 = MyClass(12)      // 👍
let c2 = new MyClass(234) // 👌 mais pas idiomatique

// (2) IDisposable => `new` required, `use` replaces `let` (otherwise it's a compiler warning)
open System.IO
let fn () =
    use f = new FileStream("hello.txt", FileMode.Open)
    f.Close()
```

### Calling an overloaded method

* Compiler may not understand which overload is being called
* Tips: call with named argument

```fsharp
let createReader fileName =
    new System.IO.StreamReader(path = fileName)
    // ☝️ Param `path` → `filename` inferred as `string`

let createReaderByStream stream =
    new System.IO.StreamReader(stream = stream)
    // ☝️ Param `stream` of type `System.IO.Stream`
```

[^1]: .NET Base Class Library
