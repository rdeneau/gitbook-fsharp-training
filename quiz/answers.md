# Answers

## 1. Who is the father of the F♯ ?

**A.** Anders Hejlsberg ❌

→ Father of C♯ and TypeScript!

**B.** Don Syme ✅

→ [dsymetweets](https://twitter.com/dsymetweets) • 🎥 [F♯ Code I Love](https://www.youtube.com/watch?v=1AZA1zoP-II)

**C.** Scott Wlaschin ❌

→ Famous blog [F♯ for Fun and Profit](https://fsharpforfunandprofit.com/), a gold mine for F♯

## 2. What is the name of the `::` operator?

**A.** Append ❌

`List.append` : concatenation of 2 lists

**B.** Concat ❌

`List.concat` : concatenation of a set of lists

**C.** Cons ✅

`newItem :: list` is the fasted way to add an item at the top of a list

## 3. Find the intruder!

B et C are functions, while A is a simple value: a `string`.

**A.** `let a = "a"` ✅

**B.** `let a () = "a"` ❌

**C.** `let a = fun () -> "a"` ❌

## 4. What line does not compile?

<pre class="language-fsharp" data-line-numbers><code class="lang-fsharp"> let evens list =
     let isEven x =
<strong>     x % 2 = 0 // 💥 Error FS0058: Unexpected syntax or possible incorrect indentation
</strong>     List.filter isEven list
</code></pre>

Line **3.** `x % 2 = 0` : an indentation is missing

## 5. What is the name of `|>` operator?

**A.** Compose ❌ - Composition operator is `>>` 📍

**B.** Chain ❌

**C.** Pipeline ❌

**D.** Pipe ✅

## 6. Which expression compile?

**A.** `a == b && b != ""` ❌

**B.** `a == b && b <> ""` ❌

**C.** `a = b && b <> ""` ✅

**D.** `a = b && b != ""` ❌

☝ In F♯, the equality and inequality operators are respectively `=` and `<>`.
