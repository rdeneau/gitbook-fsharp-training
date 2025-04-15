# 🍔 Quiz

## Question 1 - ⏱ 10’’

```fsharp
type Address = { City: string; Country: string }

let format address = $"{address.City}, {address.Country}"

let addresses: Address list = ...
```

What function should you use to apply the `format` function to `addresses`?

**A.** `List.iter()`

**B.** `List.map()`

**C.** `List.sum()`

<details>

<summary>Answer</summary>

**A.** `List.iter()` ❌

**B.** `List.map()` ✅

**C.** `List.sum()` ❌

</details>

***

## Question 2 - ⏱ 10’’

**Que vaut `[1..4] |> List.head` ?**

**A.** `[2; 3; 4]`

**B.** `1`

**C.** `4`

<details>

<summary>Answer</summary>

**A.** `[2; 3; 4]` ❌ _(This is the result of `List.tail`)_

**B.** `1` ✅

**C.** `4` ❌ _(This is the result of `List.last`)_

</details>

***

## Question 3 - ⏱ 10’’

**Quelle est la bonne manière d'obtenir la moyenne d'une liste ?**

**A.** `[2; 4] |> List.average`

**B.** `[2; 4] |> List.avg`

**C.** `[2.0; 4.0] |> List.average`

<details>

<summary>Answer</summary>

**A.** `[2; 4] |> List.average` ❌

💥 **Error FS0001:** `List.average` does not support the type `int`, because the latter lacks the required (real or built-in) member `DivideByInt`

**B.** `[2; 4] |> List.avg`

💥 **Error FS0039:** The value, constructor, namespace or type `avg` is not defined.

**C.** `[2.0; 4.0] |> List.average` ✅

</details>
