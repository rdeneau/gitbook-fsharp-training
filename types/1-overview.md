# Overview

## .NET type classifications

1. Value types _vs_ reference types
2. Primitive types _vs_ composite types
3. Generic types
4. Types created from literal values
5. Algebraic types: sum _vs_ product

## Composite types

* Created by combining other types
* Can be generic (except `enum`)

| Types          | _Version_ | Name                   | _Ref. type_ | _Value type_ |
| -------------- | --------- | ---------------------- | ----------- | ------------ |
| Types .NET     |           | `class`                | ✅           | ❌            |
|                |           | `struct`, `enum`       | ❌           | ✅            |
| Specific to C♯ | C♯ 3.0    | Anonymous type         | ✅           | ❌            |
|                | C♯ 7.0    | _Value tuple_          | ❌           | ✅            |
|                | C♯ 9.0    | `record (class)`       | ✅           | ❌            |
|                | C♯ 10.0   | `record struct`        | ❌           | ✅            |
| Specific to F♯ |           | _Tuple, Record, Union_ | ✅ (default) | ✅ (opt-in)   |
|                | F♯ 4.6    | Anonymous _Record_     | ✅ (default) | ✅ (opt-in)   |

👉 F♯ type features are stable and mature.

### Type Location

* _Top-level_ : `namespace`, top-level `module` (F♯)
* _Nested_ : `class` (C♯), `module` (F♯)
* Not definable in `let` bindings, `member`

In F♯, all type definitions are made with the `type` keyword\
→ including classes, enums and interfaces!\
→ but tuples don't need a type definition

## Particularity of F♯ types / .NET types

_Tuple, Record, Union_ are:

* Immutable by default
* Non-nullable by default
* Equality and structural comparison _(except with fields of `function` type)_
* `sealed`: cannot be inherited
* Support deconstruction, with the same syntax than for construction

## Types with literal values

Literal values = instances whose type is inferred

* Primitive types: `true` (`bool`) - `"abc"` (`string`) - `1.0m` (`decimal`)
* Tuples C♯ / F♯ : `(1, true)`
* Anonymous types C♯ : `new { Name = "Joe", Age = 18 }`
* Records F♯ : `{ Name = "Joe"; Age = 18 }`

☝ **Note :**

* Types must be defined beforehand ❗
* Exception: tuples and C♯ anonymous types: implicit definition

## Algebraic data types (ADT)

> Composite types, combining other types by _product_ or _sum._

Let's take the types `A` and `B`, then we can create:

* The product type `A × B`:
  * Contains 1 component of type `A` AND 1 of type `B`.
  * Anonymous or named components
* Sum type `A + B`:
  * Contains 1 component of type `A` OR 1 of type `B`.

By extension, same for the product/sum types of N types.

### Why _Sum_ and _Product_ terms?

It's related to the _number of values_:

* `bool` → 2 values: `true` and `false`
* `unit` → 1 value `()`
* `int` → infinite number of values

The number of values in the composed type will be:

* The sum of numbers for a _sum type_: `N(A) + N(B)`
* The product of numbers for a _product type_: `N(A) * N(B)`

## Algebraic types _vs_ Composite types

| `enum`                                | ✅ | ❌ |
| ------------------------------------- | - | - |
| F♯ _Union_                            | ✅ | ❌ |
| C♯ `class` (1), `interface`, `struct` | ❌ | ✅ |
| F♯ _Record_                           | ❌ | ✅ |
| F♯ _Tuple_                            | ❌ | ✅ |

(1) C♯ classes in the broadest sense:\
→ including modern variations like _anonymous type,_ _Value tuple_ and _Record_

👉 In C♯, only 1 sum type: `enum`, very limited / union type 📍

## Type abbreviation

**Alias** of another type: `type [name] = [existingType]`

Different use-cases:

```fsharp
// 1. Document code to avoid repetition
type ComplexNumber = float * float
type Addition<'num> = 'num -> 'num -> 'num // 👈 Also works with generics

// 2. Decouple (partially) usage / implementation
//    → Easier to change the implementation (for a stronger type)
type ProductCode = string
type CustomerId = int
```

⚠️ Deleted at compile time → no ~~_type safety_~~\
→ Compiler allows `int` to be passed instead of `CustomerId` !

💡 It is also possible to create an alias for a module 📍\
`module [name] = [existingModule]`

⚠️ It's NOT possible to create an alias for a namespace (≠ C♯)
