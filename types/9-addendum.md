# Addendum

## Types Composition

### Creating new types

Algebraic data types do not support ~~inheritance~~

* They cannot inherit from a base class.
* They are `sealed`: they cannot be inherited.
* But they can implement interfaces.

Algebraic types are created by composition:

* Aggregation into a _product type_ (_Record_, _Tuple_)
* "Choice" in a _sum type_ (_Union_)

### Combine 2 unions?

Consider the 2 unions below dealing with French-suited cards (♠ ♣ ♥ ♦):

```fsharp
type Black = Pike | Club
type Red = Heart | Tile
```

How do you combine them to create a union of `Pike`, `Club`, `Heart` or `Tile`?

#### By flattening unions, as in TypeScript ❌

```fsharp
type Color = Black | Red
```

→ It's not the expected result: `Color` is an enum-style union, with 2 cases `Black` and `Red`, totally disconnected from the `Black` and `Red` types!

#### By creating a brand new union ⚠️

```fsharp
type Color = Pike | Club | Hear | Tile
let pike = Pike
```

{% hint style="warning" %}
#### Warnings

* Can create confusion: `Pike` refers to `Color.Pike` or `Black.Pike`?
* Need to write mappers between the 2 models: `(Black, Red) <-> Color`
{% endhint %}

#### By composition ✅

```fsharp
type Color = Black of Black | Red of Red
let pike = Black Pike
```

→  The new union `Color` is composed based on the 2 previous types:

* `Black` union is used as data for the `Color.Black` case
* `Red` union is used as data for the `Color.Red` case

{% hint style="info" %}
### Note

It's common in F♯ to write union cases like `Black of Black` where the case name matches the case field type.
{% endhint %}

## Union (FP) _vs_ Object Hierarchy (OOP)

👉 A union can usually replace a small _object hierarchy._

### Explanations

Behaviors/operations implementation:

* **OO:** _virtual methods_ in separate classes
* **FP:** _functions_ relying on **pattern matchings**
  * exhaustivity
  * avoid duplication by grouping cases
  * improve readability by flattening split cases in a single `match..with`

### FP _vs_ OOP

**How we reason about the code** _(at both design and reading time)_

* **FP:&#x20;**_**by functions**_ → how an operation is performed for the different cases
* **OO:&#x20;**_**by objects**_ → how all operations are performed for a single case

**Abstraction**

* Objects are more abstract than functions
* Good abstraction is difficult to design
* The more abstract a thing is, the more stable it should be

**👉 FP is usually easier to write, to understand, to evolve**

### FP _vs_ OOP: Open-Closed Principle

It's easier to **extend** what's **open.**

**OO:** open hierarchy, closed operations

* Painful to add an operation: in all classes 😓
* Easy to add a class in the hierarchy 👍

**FP:** open operations, closed cases

* Easy to add an operation 👍
* Painful to add a case: in all functions 😓
  * Still, it's usually easier in F♯: only 1 file to change

☝️ **Notes:**

* Adding a class = new concept in the domain → always tricky ⚠️
* Adding an operation = new behavior for the existing domain concepts
