# Syntax: Rules

## Declarations order

In a file, the declarations are ordered, from top to bottom.\
→ Declaration comes before the usages.

In a `.fsproj`, the files are ordered too.\
→ We can import only something previously declared.

Type inference works by proximity: the closest match will be used.

Balance:

* Pros:
  * no cyclic dependencies
  * faster and more predictable compilation
  * code easier de reason about
* Cons:
  * need more coding discipline

Example: we try to use the `fn` before its declaration:

```fsharp
let result = fn 2
//           ~~ 💥 Error FS0039: The value or constructor 'fn' is not defined.

let fn i = i + 1
```

## Indentation

* In general, indentation is very important for code readability:\
  → It creates visual structures that match the logical structure, the hierarchy.
* The indentation is optional in C♯ because it's the combination of curly braces `{ }` and semi-colons `;` that defines the logical blocks.\
  → These characters don't help the readability by themselves. It's the indentation that matters, then the curly braces can help to improve the block visual separation.\
  → More importantly, a code not properly indented can be mis-interpreted, that can lead to bugs!
* Indentation in F♯ is required:
  * It's the only way to define code blocks
  * Compiler ensures that the indentation is correct so the reader can really trust the indentation to understand the code structure.

👉 **Conclusion:** F♯ forces us to do what matters the most for the code readability 👍

### Vertical line of indentation

It's a concept a little more tricky, related to the way F♯ understands the indentation.

* In general, a block starts in a new line, at a greater indentation level.
* But sometimes a block can start in a middle of a line. In this case, this position defines the expected vertical indentation line.

```fsharp
let f =
   let x=1
   x+1
// ^ Vertical line here

let f = let x=1
        x+1
//      ^ Vertical line here!
```

There are some exceptions to this rule, for instance with operators.\
🔗 [F♯ syntax: indentation and verbosity](https://fsharpforfunandprofit.com/posts/fsharp-syntax/), by Scott Wlaschin | F♯ for fun and profit.

☝️ Indentation rules have been relaxed in [F♯ 6](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-6#indentation-syntax-revisions).

### Recommendations

* In temporary `fsx`, speed to write code is usually more important than code readability. F♯ allows us some inconsistency and that's fine.
* In `fsproj`,  proper and consistent formatting is important for maintainability.\
  → Use consistently only spaces (no ~~tabulations~~)\
  → Use the same number of spaces for all indentation level, 4 spaces being idiomatic. The number spaces can vary only to fit the vertical indentation line _(see details below)._ \
  → Avoid naming-sensible indentation a.k.a _Vanity Alignment_ (\*) :\
     •  They can break compilation after a renaming.\
     •  Blocks too far at the right is less readable for a left-to-right language reader.\
  _→_ Use a code formatter like [**Fantomas**](https://github.com/fsprojects/fantomas) to ensure this consistency.

(\*) _Vanity alignment_ example:

```fsharp
// 👌 OK
let myLongValueName =
    someExpression
    |> anotherExpression

// ⚠️ To avoid
let myLongValueName = someExpression
                      |> anotherExpression  // 👈 Depend on the length of `myLongValueName`
```

## Attributes

Attributes works like in C# to decorate a class, a method... with little character differences:

<table><thead><tr><th width="73">Lang</th><th width="246">Separated attributes</th><th>Merged attributes</th></tr></thead><tbody><tr><td>C#</td><td><code>[AttributeOne]</code><br><code>[SecondAttribute]</code></td><td><code>[AttributeOne, SecondAttribute</code></td></tr><tr><td>F♯</td><td><code>[&#x3C;AttributeOne>]</code><br><code>[&#x3C;SecondAttribute>]</code></td><td><code>[&#x3C;AttributeOne>; &#x3C;SecondAttribute>]</code></td></tr></tbody></table>

## Addendum

🔗 [F♯ Cheatsheet](https://fsprojects.github.io/fsharp-cheatsheet/)

🔗 [Troubleshooting F♯ - Why won't my code compile?](https://fsharpforfunandprofit.com/troubleshooting-fsharp/)
