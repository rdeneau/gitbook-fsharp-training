---
description: A.k.a. Discriminated Unions (DU)
---

# Unions

## Key points

- Exact term: *Discriminated Union (DU)*
- Sum type: represents an **OR**, a **choice** between several *Cases*
  - Same principle as for an `enum`, but on steroids 💪
- Each *case* must have a *Tag* *(a.k.a Label, Discriminator)* -- in `PascalCase` ❗
- Each *case* **may** contain data
  - As Tuple: its elements can be named -- in camelCase 🙏

```fs
type Ticket =
    | Adult                  // no data -> ≃ singleton stateless
    | Senior of int          // holds an 'int' (w/o more precision)
    | Child of age: int      // holds an 'int' named 'age'
    | Family of Ticket list  // holds a list of tickets
                             // recursive type by default (no 'rec' keyword)
```

### Cases naming

*Cases* can be used without **qualification**: `Int32` *vs* `IntOrBool.Int32`

Qualification can be forced with `RequireQualifiedAccess` attribute: \
• Cases using common terms (e.g. `None`) → to avoid name collision \
• Cases names are designed to read better/more explicitly with qualification

*Cases* must be named in **PascalCase** ❗ \
• Since F# 7.0, camelCase is allowed for `RequireQualifiedAccess` unions 💡

### Field labels

Helpful for:

- Adding meaning to a primitive type \
  → See `Ticket` previous example: `Senior of int` vs `Child of age: int`
- Distinguish between two fields of the same type \
  → See example below:

```fsharp
type ComplexNumber =
    | Cartesian of Real: float * Imaginary: float
    | Polar of Magnitude: float * Phase: float
```

## Instanciation

*Case* ≃ **constructor** \
→ Function called with any *case* data

```fs
type Shape =
    | Circle of radius: int
    | Rectangle of width: int * height: int

let circle = Circle 12         // Type: 'Shape', Value: 'Circle 12'
let rect = Rectangle(4, 3)     // Type: 'Shape', Value: 'Rectangle (4, 3)'

let circles = [1..4] |> List.map Circle     // 👈 Case used as function
```

## Name conflict

When 2 unions have tags with the same name
→ Qualify the tag with the union name

```fs
type Shape =
    | Circle of radius: int
    | Rectangle of width: int * height: int

type Draw = Line | Circle    // 'Circle' will conflict with the 'Shape' tag

let draw = Circle            // Type='Draw' (closest type) -- ⚠️ to be avoided as ambiguous

// Tags qualified by their union type
let shape = Shape.Circle 12
let draw' = Draw.Circle
```

## Unions: get the data out

- Only via *pattern matching*.
- Matching a union type is **exhaustive**.

```fs
type Shape =
    | Circle of radius: float
    | Rectangle of width: float * height: float

let area shape =
    match shape with
    | Circle r -> Math.PI * r * r   // 💡 Same syntax as instantiation
    | Rectangle (w, h) -> w * h

let isFlat = function
    | Circle 0.
    | Rectangle (0., _)
    | Rectangle (_, 0.) -> true
    | Circle _
    | Rectangle _ -> false
```

## Single-case unions

Unions with a single case encapsulating a type (usually primitive)

```fs
type CustomerId = CustomerId of int
type OrderId = OrderId of int

let fetchOrder (OrderId orderId) =    // 💡 Direct deconstruction without 'match' expression
    ...
```

- **Benefits** 👍
  - Ensures *type safety* unlike simple type alias <br/>      → Impossible to pass a `CustomerId` to a function waiting for an `OrderId`
  - Prevents *Primitive Obsession* at a minimal cost
- **Trap** ⚠️
  - `OrderId orderId` looks like C# parameter definition

## Enum style unions

All *cases* are empty = devoid of data \
→ ≠ .NET `enum` based on numeric values 📍

Instantiation and pattern matching are done just with the *Case*. \
→ The *Case* is no longer a ~~function~~ but a *singleton* value.

```fs
type Answer = Yes | No | Maybe
let answer = Yes

let print answer =
    match answer with
    | Yes   -> printfn "Oui"
    | No    -> printfn "Non"
    | Maybe -> printfn "Peut-être"
```

🔗 [“Enum” style unions | F# for fun and profit](https://fsharpforfunandprofit.com/posts/fsharp-decompiled/#enum-style-unions)

## Unions .Is* properties

The compiler generates `.Is{Case}` properties for each case in a union

- Before F♯ 9: not accessible + we cannot add them manually 😒
- Since F♯ 9: accessible 👍

```fsharp
type Contact =
    | Email of address: string
    | Phone of countryCode: int * number: string

type Person = { Name: string; Contact: Contact }

let canSendEmailTo person =  // Person -> bool
    person.Contact.IsEmail   // `.IsEmail` is auto-generated
```

## Union (FP) *vs* Object Hierarchy (OOP)

👉 A union can usually replace a small *object hierarchy.*

### Explanations

Behaviors/operations implementation:

- **OO:** *virtual methods* in separated classes
- **FP:** *functions* relying on **pattern matchings**
  - exhaustivity
  - avoid duplication by grouping cases
  - improve readability by flattening split cases in a single `match..with`

### FP *vs* OOP

**How we reason about the code** *(at both design and reading time)*

- **FP: by functions** → how an operation is performed for the different cases
- **OOP: by objects** → how all operations are performed for a single case

**Abstraction**

- Objects are more abstract than functions
- Good abstraction is difficult to design
- The more abstract a thing is, the more stable it should be

**👉 FP is usually easier to write, to understand, to evolve**

### FP *vs* OOP: Open-Closed Principle

It's easier to extend what's **Open.**

**OOP:** open hierarchy, closed operations

- Painful to add an operation: in all classes 😓
- Easy to add a class in the hierarchy 👍

**FP:** open operations, closed cases

- Easy to add an operation 👍
- Painful to add a case: in all functions 😓
  - Still, it's usually easier in F♯: only 1 file to change

☝️ **Notes:**

- Adding a class = new concept in the domain → always tricky ⚠️
- Adding an operation = new behavior for the existing domain concepts
