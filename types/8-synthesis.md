# Synthesis

## 🕹️ Quiz wrap up

```fsharp
// Match types with concepts (1 to many)
type Color1 = int * int * int
type Color2 = Red | Green | Blue
type Color3 = Red=1 | Green=2 | Blue=3
type Color4 = { Red: int; Green: int; Blue: int }
type Color5 = {| Red: int; Green: int; Blue: int |}
type Color6 = Color of Red: int * Green: int * Blue: int
type Color7 =
    | RGB of {| Red: int; Green: int; Blue: int |}
    | HSL of {| Hue: int; Saturation: int; Lightness: int |}

// A. Alias
// B. Enum
// C. Record
// D. Record anonyme
// E. Single-case union
// F. Union
// G. Union enum-like
// H. Tuple
```

***

### Answer

| Types                                       | Concepts                                |
| ------------------------------------------- | --------------------------------------- |
| `type Color1 = int * int * int`             | **H.** Tuple + **A.** Alias             |
| `type Color2 = Red ∣ Green ∣ Blue`          | **G.** Union enum-like                  |
| `type Color3 = Red=1 ∣ Green=2 ∣ Blue=3`    | **B.** Enum                             |
| `type Color4 = { Red: int; Green: int… }`   | **C.** Record                           |
| `type Color5 = {∣ Red: int; Green: int… ∣}` | **D.** Anonymous Record + **A.** Alias  |
| `type Color6 = Color of Red: int * …`       | **E.** Single-case union + **H.** Tuple |
| `type Color7 = RGB of {∣…∣} ∣ HSL of {∣…∣}` | **F.** Union + **D.** Anonymous Record  |

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

* By flattening, as in TypeScript?

```fsharp
type Color = Black | Red
```

❌ It's not the expected result: `Color` is an enum-style union, with 2 cases `Black` and `Red`, totally disconnected from the `Black` and `Red` types!

* By creating a brand new union?

```fsharp
type Color = Pike | Club | Hear | Tile
let pike = Pike
```

⚠️ Can create confusion: `Pike` refers to `Color.Pike` or `Black.Pike`?

⚠️ Need to write mappers between the 2 models.

* By composition?

```fsharp
type Color = Black of Black | Red of Red
let pike = Black Pike
```

✅ It works!

## Conclusion

Lots of ways to model!

💡 Opportunity for:

* Team discussions
* Business domain encoding in types
