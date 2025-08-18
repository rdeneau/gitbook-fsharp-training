---
description: F♯ Training, covering versions 5.0 to 9.0 (2021-2024)
---

# Presentation

## Slides

This git-book is here to supplement training sessions I've conducted using slides located in this repository:

{% embed url="https://github.com/rdeneau/formation-fsharp/tree/main/en" %}

* Written in markdown with marp / marpit
* Outputs available in HTML and PDF files
* The theme can be customized: it's "just" CSS and PNG files.

## Author

Romain DENEAU

* Senior Developer F♯ C♯ TypeScript
* Github [/rdeneau](https://github.com/rdeneau)
* Linked-in [romain-deneau](https://www.linkedin.com/in/romain-deneau-95481143/)
* ~~Twitter/X~~ [~~@DeneauRomain~~](https://x.com/DeneauRomain)

## Why F♯

[Answer in a single tweet](https://nitter.net/MokoSharma/status/1458151277343379457) :

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

In detail :

* Multi-paradigm language with a strong functional orientation
  * Functional programming _(FP)_ principles: immutability and composition
  * Building blocks: functions and algebraic data types _(ADT)_
* "Fun" ! Very pleasant to write and read
  * Expressive and concise (not very verbose syntax)
  * Sensitive to indentation → easy to read
  * Strong static typing but it's almost seamless thanks to type inference
* Language entreprise-friendly
  * Runs on the .NET platform → high performance, easy interop with C# projects, .NET ecosystem and libraries
  * Very good tooling: Visual Studio, VsCode, Rider
  * Robust code: predictable and reproducible results (immutability, structural equality, no nulls, exhaustive pattern matching)
  * Interactive programming: check code by evaluating it in the FSI console
  * Narrowed community but strong and friendly
* F♯ compared to other back-end functional languages
  * Its syntax is not hybrid, unlike **Scala** and **Kotlin** (mixing OOP and FP)
  * Easier to learn than **Haskell** or **OCaml** _(but with fewer FP features, partially compensated for by OOP features)_
  * Static typing, unlike **Closure** _(and far fewer brackets 😜)_

## Convention

📍 → concept addressed later, generally with the associated link

💥 → compilation error or runtime exception

🚀 → more advanced chapters

🍔 or 🎲 → Quiz

📜 → Summary

## Target audience

C# Developers - .NET SDK installed on your machine\
Notions in JavaScript/TypeScript can be helpful when we compare the languages

{% hint style="success" %}
#### Perform the training yourself

As a F# developer in a .NET team using only C#, feel free to use these materials (both git-book and slides) to train your team.

💡 The theme can be customized: it's "just" CSS and PNG files.
{% endhint %}

## Changelog

### 2025-08-18

* Improve [4-functional-pattern.md](types-monadic/4-functional-pattern.md "mention")
* Improve [5-computation-expressions.md](types-monadic/5-computation-expressions.md "mention")

### 2025-05-13

* Dedicated page to cover everything regarding [exceptions.md](fundamentals/2-syntax/exceptions.md "mention")
* Misc. precisions regarding [Broken link](broken-reference "mention")

### 2025-04-02

* Translation from [French](https://rdeneau.gitbook.io/formation-fsharp-fr)
