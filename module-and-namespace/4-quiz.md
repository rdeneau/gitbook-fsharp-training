# 🍔 Quiz

## Question 1

Is the following code valid?

```fsharp
namespace A

let a = 1
```

**A.** Yes

**B.** No

<details>

<summary>Answer</summary>

**A.** ~~Yes~~ ❌

**B.** No ✅

→ A namespace cannot contain values!

</details>

---

## Question 2

Is the following code valid?

```fsharp
namespace A

module B

let a = 1
```

**A.** Yes

**B.** No

<details>

<summary>Answer</summary>

**A.** ~~Yes~~ ❌

**B.** No ✅

→ module B is declared as top-level \
→ forbidden after a namespace

**Valid equivalent code:**

- Option 1: top-level module

```fsharp
module A.B

let a = 1
```

- Option 2: namespace + local module

```fsharp
namespace A

module B =
    let a = 1
```

</details>

---

## Question 3

Give the fully-qualified name for `add`?

```fsharp
namespace Common.Utilities

module IntHelper =
    let add x y = x + y
```

**A.** `add`

**B.** `IntHelper.add`

**C.** `Utilities.IntHelper.add`

**D.** `Common.Utilities.IntHelper.add`

<details>

<summary>Answer</summary>

**A.** `add` ❌

**B.** `IntHelper.add` ❌

**C.** `Utilities.IntHelper.add` ❌

**D.** `Common.Utilities.IntHelper.add` ✅

→ `IntHelper` for the parent module \
→ `Common.Utilities` for the root namespace

</details>
