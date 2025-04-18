# Module

## Syntax

```fsharp
// Top-level module
module [accessibility-modifier] [qualified-namespace.]module-name
declarations

// Local module
module [accessibility-modifier] module-name =
    declarations
```

`accessibility-modifier`: restrict accessibility \
→ `public` *(default)*, `internal` *(assembly)*, `private` *(parent)*

Full name (`[namespace.]module-name`) must be unique \
→ 2 files cannot declare modules with the same name

## Top-level module

- Only one top-level module per file \
  → Declared on very top of the file

- Can (should?) be qualified \
  → Attached to a parent namespace *(already declared or not)*

- Contains all the rest of the file \
  → Unindented content 👍

## Implicit top-level module

- For a file without top-level module/namespace
- Module name = file name
  - Without extension
  - With 1st letter in uppercase
  - E.g.: `program.fs` → `module Program`

☝️ Not recommended in `.fsproj`

## Local module

Syntax similar to `let`

- The `=` sign after the local module name ❗
- Indent the entire content

## Content

A module, *local as top-level*, can contain:

- local types and sub-modules
- values, functions

**Key difference:** \
→ content indentation

| Module    | Indentation |
|-----------|-------------|
| top-level | No          |
| local     | Yes         |

## Module/static class equivalence

```fsharp
module MathStuff =
    let add x y  = x + y
    let subtract x y = x - y
```

This F# module is equivalent to the following static class in C♯:

```csharp
public static class MathStuff
{
    public static int add(int x, int y) => x + y;
    public static int subtract(int x, int y) => x - y;
}
```

See [sharplab.io](https://sharplab.io/#v2:DYLgZgzgPgtg9gEwK7AKYAICyBDALgCwGVckwx0BeAWAChb0H01d1sEF0APdATwYq7oA1L3qNm6CEgBGuAE7YAxi259KggLS8gA=)

## Module nesting

As with C♯ classes, F♯ modules can be nested.

```fsharp
module Y =
    module Z =
        let z = 5

printfn "%A" Y.Z.z
```

☝ **Notes :**

- Interesting with private nested module to isolate/group
- Otherwise, prefer a *flat view*
- F♯ classes cannot be nested

## Top-level *vs* local module

| Property                    | Top-level | Local |
|-----------------------------|-----------|-------|
| Qualifiable                 | ✅        | ❌    |
| `=` sign + indented content | ❌        | ✅ ❗ |

- *Top-level* module → 1st element declared in a file
- Otherwise *(after a top-level module/namespace)* → local module

## Recursive module

Same principle as recursive namespace \
→ Convenient for a type and a related module to see each other

☝ **Recommendation:** limit the size of recursive zones as much as possible

## Module annotation

2 opposite attributes impact the module usage

### `[<RequireQualifiedAccess>]`

Prevents the module import hence any unqualified use of its elements \
→ 💡 Useful for avoiding *shadowing* for common names: `add`, `parse`...

### `[<AutoOpen>]`

Import module at same time as the parent namespace/module \
→ 💡 Handy for "mounting" values/functions at namespace level \
→ ⚠️ Pollutes the current *scope*

⚠️ **Scope** \
When the parent module/namespace has not been imported, `AutoOpen` has no effect: to access the contents of the child module, the qualification includes not only the parent module/namespace, but also the child module: \
→ `Parent.childFunction` ❌ \
→ `Parent.Child.childFunction` ✅

💡 `AutoOpen` is commonly used to better organize a module, by grouping elements into child modules \
→ provided they remain of a reasonable size, otherwise it would be better to consider extracting them to different files.

This can be combined with making some modules `private` to hide all their contents from the calling code, while keeping these contents directly accessible to the rest of the current module.

👉 Having an `AutoOpen` module inside a `RequireQualifiedAccess` module only makes sense if the module is `private`.

### `AutoOpen`, `RequireQualifiedAccess` or nothing?

Let's consider a `Cart` type with its `Cart` companion module.

**How do we call the function that adds an item to the cart?** \
→ It depends on the function name.

- `addItem item cart`: \
  → `[<RequireQualifiedAccess>]` to consider \
  → to be compelled to use `Cart.addItem`
- `addItemToCart item cart`: \
  → function name is *self-explicit* \
  → `[<AutoOpen>]` interesting to prevent `Cart.addItemToCart` \
  → Works only if `Cart` parent *(if any)* is not `RequireQualifiedAccess` and opened

If the `Cart` module contains other functions like this, it's probably better to apply the same naming convention to all of them.

## Types-Modules main typologies

*(Not exhaustive)*

- Type + Companion module containing function dedicated to this type
- Multi-type module: several small types + related functions
- Mapper modules: to map between 2 types sets

### Type + Companion module

FSharp.Core style - see `List`, `Option`, `Result`...

Module can have the same name as the type
→ BCL interop: module compiled name = `{Module}Module`

```fsharp
type Person = { FirstName: string; LastName: string }

module Person =
    let fullName person = $"{person.FirstName} {person.LastName}"

let person = { FirstName = "John"; LastName = "Doe" }   // Person
person |> Person.fullName // "John Doe"
```

### Multi-type module

Contains several small types + related functions *(eventually)*

```fsharp
module Common.Errors

type OperationNotAllowedError = { Operation: string; Reason: string }

type Error =
    | Bug of exn
    | OperationNotAllowed of OperationNotAllowedError


let bug exn = Bug exn |> Error

let operationNotAllowed operation reason =
    { Operation = operation
      Reason = reason }
    |> Error
```

### Mapper modules

To map between 2 types sets

```fsharp
// Domain/Types/Mail.fs ---
module Domain.Types.Mail

[ types... ]

// Data/Mail/Entities.fs ---
module Data.Mail.Entities

[ DTO types... ]

// Data/Mail.Mappers ---
module Data.Mail.Mappers

module DomainToEntity =
    let mapXxx x : XxxDto = ...
```

## Module *vs* namespace

If a file contains a single module

- Prefer top-level module in general
- Prefer namespace + local module for BCL interop

## Open type *(Since F♯ 5)*

Use cases:

### **1.** Import static classes to get direct access to methods

```fsharp
open type System.Math
let x = Max(123., 456.)
```

*vs*

```fsharp
open System
let x = Math.Max(123., 456.)
```

☝️ In general, use case only recommended for classes designed for this usage.

### **2.** Cherry-pick imports

→ Import only the types needed in a module

```fsharp
// Domain/Sales.fs ---
module Domain.Sales =
    type Balance = Overdrawn of decimal | Remaining of decimal
    // Other types, functions...

// Other/Module.fs ---
open type Sales.Balance

let myBalance = Remaining of 500. // myBalance is of type Balance.
```
