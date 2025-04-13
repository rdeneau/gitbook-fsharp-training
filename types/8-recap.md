# 📜 Recap

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

<details>

<summary>Answer</summary>

| Types                                       | Concepts                                |
| ------------------------------------------- | --------------------------------------- |
| `type Color1 = int * int * int`             | **H.** Tuple + **A.** Alias             |
| `type Color2 = Red ∣ Green ∣ Blue`          | **G.** Union enum-like                  |
| `type Color3 = Red=1 ∣ Green=2 ∣ Blue=3`    | **B.** Enum                             |
| `type Color4 = { Red: int; Green: int… }`   | **C.** Record                           |
| `type Color5 = {∣ Red: int; Green: int… ∣}` | **D.** Anonymous Record + **A.** Alias  |
| `type Color6 = Color of Red: int * …`       | **E.** Single-case union + **H.** Tuple |
| `type Color7 = RGB of {∣…∣} ∣ HSL of {∣…∣}` | **F.** Union + **D.** Anonymous Record  |

</details>

## Conclusion

Lots of ways to model!

💡 Opportunity for:

* Team discussions
* Business domain encoding in types
