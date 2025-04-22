# Type extensions

## Definition

Members of a type defined outside its main `type` block.

Each of these members is called **augmentation** or **extension**.

3 categories of extension :

* Intrinsic extension
* Optional extension
* Extension methods

## Intrinsic extension

> Declared in the same file and namespace as the type

**Use case:** Features available in both companion module and type\
→ E.g. `List.length list` function and `list.Length` member

_How to implement it following top-down declarations?_

**1. Implement in type**, Redirect module functions to type members\
→ More straightforward

**2. Intrinsic extensions:**\
→ Declare type "naked", Implement in module, Augment type after\
→ Favor FP style, Transparent for Interop

**Example:**

```fsharp
namespace Example

type Variant =
    | Num of int
    | Str of string

module Variant =
    let print v =
        match v with
        | Num n -> printf "Num %d" n
        | Str s -> printf "Str %s" s

// Add a member as an extension - see `with` required keyword
type Variant with
    member x.Print() = Variant.print x
```

## Optional extension

Extension defined outside the type module/namespace/assembly

**Use cases:**

1. Types we can't modify, for instance coming from a library
2. Keep types naked - e.g. Elmish MVU pattern

```fsharp
module EnumerableExtensions

open System.Collections.Generic

type IEnumerable<'T> with
    /// Repeat each element of the sequence n times
    member xs.RepeatElements(n: int) =
        seq {
            for x in xs do
                for _ in 1 .. n -> x
        }

// Usage: in another file ---
open EnumerableExtensions

let x = [1..3].RepeatElements(2) |> List.ofSeq
// [1; 1; 2; 2; 3; 3]
```

**Compilation:** into static methods\
→ Simplified version of the previous example:

```csharp
public static class EnumerableExtensions
{
    public static IEnumerable<T> RepeatElements<T>(IEnumerable<T> xs, int n) {...}
}
```

**Another example:**

```fsharp
// Person.fs ---
type Person = { First: string; Last: string }

// PersonExtensions.fs ---
module PersonExtensions =
    type Person with
        member this.FullName =
            $"{this.Last.ToUpper()} {this.First}"

// Usage elsewhere ---
open PersonExtensions
let joe = { First = "Joe"; Last = "Dalton" }
let s = joe.FullName  // "DALTON Joe"
```

### Limits

* Must be declared in a module
* Not compiled into the type, not visible to Reflection
* Usage as pseudo-instance members only in F♯\
  ≠ in C♯: as static methods

## Type extension _vs_ virtual methods

☝ Override virtual methods:

* in the initial type declaration ✅
* not in a ~~type extension~~ ⛔

```fsharp
type Variant = Num of int | Str of string with
    override this.ToString() = ... ✅

module Variant = ...

type Variant with
    override this.ToString() = ... ⚠️
    // Warning FS0060: Override implementations in augmentations are now deprecated...
```

## Type extension _vs_ type alias

Incompatible❗

```fsharp
type i32 = System.Int32

type i32 with
    member this.IsEven = this % 2 = 0
// 💥 Error FS0964: Type abbreviations cannot have augmentations
```

💡 **Solution:** use the real type name

```fsharp
type System.Int32 with
    member this.IsEven = this % 2 = 0
```

☝ **Corollary:** F♯ tuples such as `int * int` cannot be augmented, but they can be extended using C♯-style extension methods 📍

## Type extension _vs_ Generic type constraints

Extension allowed on generic type except when constraints differ:

```fsharp
open System.Collections.Generic

type IEnumerable<'T> with
//   ~~~~~~~~~~~ 💥 Error FS0957
//   One or more of the declared type parameters for this type extension
//   have a missing or wrong type constraint not matching the original type constraints on 'IEnumerable<_>'
    member this.Sum() = Seq.sum this

// ☝ This constraint comes from `Seq.sum`.
```

**Solution:** C♯-style extension method 📍

## Extension method (C♯-style)

Static method:

* Decorated with `[<Extension>]`
* In F♯ < 8.0: Defined in class decorated with `[<Extension>]`
* Type of 1st argument = extended type _(`IEnumerable<'T>` below)_

**Example:**

```fsharp
open System.Runtime.CompilerServices

[<Extension>] // 💡 Not required anymore since F♯ 8.0
type EnumerableExtensions =
    [<Extension>]
    static member inline Sum(xs: seq<_>) = Seq.sum xs

let x = [1..3].Sum()
//------------------------------
// Output in FSI console (verbose syntax):
type EnumerableExtensions =
  class
    static member
      Sum : xs:seq<^a> -> ^a
              when ^a : (static member ( + ) : ^a * ^a -> ^a)
              and  ^a : (static member get_Zero : -> ^a)
  end
val x : int = 6
```

C♯ equivalent:

```csharp
using System.Collections.Generic;

namespace Extensions
{
    public static class EnumerableExtensions
    {
        public static TSum Sum<TItem, TSum>(this IEnumerable<TItem> source) {...}
    }
}
```

☝ **Note:** The actual implementations of `Sum()` in LINQ are different,\
one per type: `int`, `float`... → [_Source code_](https://github.com/dotnet/runtime/blob/main/src/libraries/System.Linq/src/System/Linq/Sum.cs)

### Tuples

An extension method can be added to any F♯ tuple:

```fsharp
open System.Runtime.CompilerServices

[<Extension>]
type EnumerableExtensions =
    [<Extension>]
    // Signature : ('a * 'a) -> bool when 'a : equality
    static member inline IsDuplicate((x, y)) = // 👈 Double () required
        x = y

let b1 = (1, 1).IsDuplicate()  // true
let b2 = ("a", "b").IsDuplicate()  // false
```

## Comparison

| Feature            | Type extension          | Extension method     |
| ------------------ | ----------------------- | -------------------- |
| Methods            | ✅ instance, ✅ static    | ✅ instance, ❌ static |
| Properties         | ✅ instance, ✅ static    | ❌ _Not supported_    |
| Constructors       | ✅ intrinsic, ❌ optional | ❌ _Not supported_    |
| Extend constraints | ❌ _Not supported_       | ✅ _Support SRTP_     |

## Limits

Type extensions do not support (sub-typing) polymorphism:

* Not in the virtual table
* No `virtual`, `abstract` member
* No `override` member _(but overloads 👌)_

## Extensions _vs_ C♯ partial class

| Feature             | Multi-files | Compiled into the type | Any type             |
| ------------------- | ----------- | ---------------------- | -------------------- |
| C♯ partial class    | ✅ Yes       | ✅ Yes                  | Only `partial class` |
| Extension intrinsic | ❌ No        | ✅ Yes                  | ✅ Yes                |
| Extension optional  | ✅ Yes       | ❌ No                   | ✅ Yes                |
