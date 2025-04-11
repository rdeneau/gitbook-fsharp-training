# Types Overview

## .NET type classifications

1. Value types *vs* reference types
2. Primitive types *vs* composite types
3. Generic types
4. Types created from literal values
5. Algebraic types: sum *vs* product

## Composite types

• Created by combining other types \
• Can be generic (except `enum`)

| Types          | *Version* | Name                   | *Ref. type*  | *Value type* |
|----------------|-----------|------------------------|--------------|--------------|
| Types .NET     |           | `class`                | ✅           | ❌           |
|                |           | `struct`, `enum`       | ❌           | ✅           |
| Specific to C♯ | C♯ 3.0    | Anonymous type         | ✅           | ❌           |
|                | C♯ 7.0    | *Value tuple*          | ❌           | ✅           |
|                | C♯ 9.0    | `record (class)`       | ✅           | ❌           |
|                | C♯ 10.0   | `record struct`        | ❌           | ✅           |
| Specific to F♯ |           | *Tuple, Record, Union* | ✅ (default) | ✅ (opt-in)  |
|                | F♯ 4.6    | Anonymous *Record*     | ✅ (default) | ✅ (opt-in)  |

👉 F♯ type features stable and mature

### Type Location

- *Top-level* : `namespace`, top-level `module` (F♯)
- *Nested* : `class` (C♯), `module` (F♯)
- Not definable in `let` bindings, `member`

In F♯, all type definitions are made with the `type` keyword \
→ including classes, enums and interfaces! \
→ but tuples don't need a type definition

## Particularity of F♯ types / .NET types

*Tuple, Record, Union* are:

- Immutable by default
- Non-nullable by default
- Equality and structural comparison *(except with fields of `function` type)*
- `sealed`: cannot be inherited
- Support deconstruction, with the same syntax than for construction

## Types with literal values

Literal values = instances whose type is inferred

- Primitive types: `true` (`bool`) - `"abc"` (`string`) - `1.0m` (`decimal`)
- Tuples C♯ / F♯ : `(1, true)`
- Anonymous types C♯ : `new { Name = "Joe", Age = 18 }`
- Records F♯ : `{ Name = "Joe"; Age = 18 }`

☝ **Note :**

- Types must be defined beforehand ❗
- Exception: tuples and C♯ anonymous types: implicit definition

## Algebraic data types (ADT)

> Composite types, combining other types by *product* or *sum.*

Let's take the types `A` and `B`, then we can create:

- The product type `A × B`:
  - Contains 1 component of type `A` AND 1 of type `B`.
  - Anonymous or named components
- Sum type `A + B`:
  - Contains 1 component of type `A` OR 1 of type `B`.

By extension, same for the product/sum types of N types.

### Why *Sum* and *Product* terms?

It's related to the *number of values*:

- `bool` → 2 values: `true` and `false`
- `unit` → 1 value `()`
- `int` → infinite number of values

The number of values in the composed type will be:

- The sum of numbers for a *sum type*: `N(A) + N(B)`
- The product of numbers for a *product type*: `N(A) * N(B)`

## Types algébriques _vs_ Types composites

| Type _custom_                    | Somme | Produit | Composantes nommées |
| -------------------------------- | ----- | ------- | ------------------- |
| `enum`                           | ✅     | ❌       | ➖                   |
| _Union_ F♯                       | ✅     | ❌       | ➖                   |
| `class` ⭐, `interface`, `struct` | ❌     | ✅       | ✅                   |
| _Record_ F♯                      | ❌     | ✅       | ✅                   |
| _Tuple_ F♯                       | ❌     | ✅       | ❌                   |

⭐ Classes + variations C♯ : type anonyme, _Value tuple_ et `record`

👉 En C♯, pas de type somme sauf `enum`, très limitée par rapport au type union 📍 [unions.md](unions.md "mention")

## Type abbreviation

**Alias** of another type: `type [name] = [existingType]`

Different use-cases:

```fs
// 1. Document code to avoid repetition
type ComplexNumber = float * float
type Addition<'num> = 'num -> 'num -> 'num // 👈 Also works with generics

// 2. Decouple (partially) usage / implementation
//    → Easier to change the implementation (for a stronger type)
type ProductCode = string
type CustomerId = int
```

⚠️ Deleted at compile time → no ~~*type safety*~~
→ Compiler allows `int` to be passed instead of `CustomerId` !

💡 It is also possible to create an alias for a module 📍 \
`module [name] = [existingModule]`

⚠️ It's NOT possible to create an alias for a namespace (≠ C♯)
