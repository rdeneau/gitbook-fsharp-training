---
description: 6 questions to test your memory
---

# 🍔 Quiz

## 1. Who is the father of the F♯? ⏱ 10’’

**A.** Anders Hejlsberg

**B.** Don Syme

**C.** Scott Wlaschin

<details>

<summary>Answer</summary>

**A.** Anders Hejlsberg ❌

→ Father of C♯ and TypeScript!

**B.** Don Syme ✅

→ [dsymetweets](https://twitter.com/dsymetweets) • 🎥 [F♯ Code I Love](https://www.youtube.com/watch?v=1AZA1zoP-II)

**C.** Scott Wlaschin ❌

→ Famous blog [F♯ for Fun and Profit](https://fsharpforfunandprofit.com/), a gold mine for F♯

</details>

***

## 2. What is the name of the `::` operator? ⏱ 10’’

**A.** Append

**B.** Concat

**C.** Cons

<details>

<summary>Answer</summary>

**A.** Append ❌

`List.append` : concatenation of 2 lists

**B.** Concat ❌

`List.concat` : concatenation of a set of lists

**C.** Cons ✅

`newItem :: list` is the fasted way to add an item at the top of a list

</details>

***

## 3. Find the intruder! ⏱ 15’’

**A.** `let a = "a"`

**B.** `let a () = "a"`

**C.** `let a = fun () -> "a"`

<details>

<summary>Answer</summary>

B and C are functions, while A is a simple value: a `string`.

**A.** `let a = "a"` ✅

**B.** `let a () = "a"` ❌

**C.** `let a = fun () -> "a"` ❌

</details>

***

## 4. What line does not compile? ⏱ 20’’

{% code lineNumbers="true" %}
```fsharp
 let evens list =
     let isEven x =
     x % 2 = 0
     List.filter isEven list
```
{% endcode %}

<details>

<summary>Answer</summary>

<pre class="language-fsharp" data-line-numbers><code class="lang-fsharp"> let evens list =
     let isEven x =
<strong>     x % 2 = 0 // 💥 Error FS0058: Unexpected syntax or possible incorrect indentation
</strong>     List.filter isEven list
</code></pre>

Line **3.** `x % 2 = 0` : an indentation is missing

</details>

***

## 5. What is the name of `|>` operator? ⏱ 10’’

**A.** Compose

**B.** Chain

**C.** Pipeline

**D.** Pipe

<details>

<summary>Answer</summary>

**A.** Compose ❌ - Composition operator is `>>` 📍

**B.** Chain ❌

**C.** Pipeline ❌

**D.** Pipe ✅

</details>

***

## 6. Which expression compiles? ⏱ 20’’

**A.** `a == "a" && b != "*"`

**B.** `a == "a" && b <> "*"`

**C.** `a = "a" && b <> "*"`

**D.** `a = "a" && b != "*"`

<details>

<summary>Answer</summary>

**C.** `a = "a" && b <> ""` ✅

| Operator   | C♯             | F♯             |
|------------|----------------|----------------|
| Equality   | `==`           | `=`            |
| Inequality | `!=` (`!` `=`) | `<>` (`<` `>`) |

</details>
