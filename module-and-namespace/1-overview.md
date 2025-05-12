# Overview

## Similarities

Modules and namespaces allow you to:

* **Organize code** into zones of related functionality
* Avoid name collisions

## Differences

| Property        | Namespace      | Module                   |
| --------------- | -------------- | ------------------------ |
| .NET equivalent | `namespace`    | `static class`           |
| Type            | _Top-level_    | _Top-level_ or local     |
| Contains        | Modules, Types | Idem + Values, Functions |
| Annotable       | ❌ No           | ✅ Yes                    |

**Scope:** Namespaces > Files > Modules

## Use a module or a namespace

1. Either qualify the elements individually
2. Or import everything with the `open` keyword
   * placed anywhere before the usage, at the top recommended
   * `open Name.Space` ≡ C♯ `using Name.Space`
   * `open My.Module` ≡ C♯ `using static My.Module`

```fsharp
// Option 1: Qualify usages
let result1 = Arithmetic.add 5 9

// Option 2: Import the entire module
open Arithmetic
let result2 = add 5 9
```

## Imports order

⚠️ **Warning:** the order of the imports is taken into account, from top to bottom.

* This allows _shadowing_ (see below).
* As a result, reordering imports in an existing file may cause compilation to fail!
* So, although it's advisable to have imports sorted alphabetically, this isn't always possible. It is therefore advisable to indicate this in the code by means of a comment.

## Shadowing at import

Imports are done without name conflicts but need **disambiguation:**

* Modules and static classes are merged ✅
* Types and functions are shadowed ❗\
  &#x20; \- Last-imported-wins mode: see example below\
  &#x20; \- Import order matters

```fsharp
module IntHelper =
    let add x y = x + y

module FloatHelper =
    let add x y : float = x + y

open IntHelper
open FloatHelper // 👈 IntHelper add is shadowed by FloatHelper add

let result = add 1 2 // 💥 Error FS0001: The type 'float' does not match the type 'int'
```

{% hint style="warning" %}
**☝ Recommendation**

_Shadowing_ is more likely to happen with common names such as `add` in the previous example, or `map`, `filter`... from the `List`, `Seq`... modules.

_Qualification_ is recommended in this case, and it can be made mandatory by decorating the module with `[<RequireQualifiedAccess>]`📍
{% endhint %}

## Keyword `global`

Because of _Shadowing,_ we may come across cases where we can't import the right namespace. For example, having done `open FsCheck` and referenced the `FsCheck.Xunit` library, when you do `open Xunit`, we don't import the `Xunit` namespace from the `Xunit` library but the `FsCheck.Xunit` namespace!

💡 We resolve fix issue by doing `open global.Xunit`.
