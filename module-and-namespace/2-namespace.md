# Namespace

## Syntax

`namespace [rec] [parent.]identifier`

* `rec` for recursive → see *slide next*
* `parent` for grouping namespaces

## Content

A `namespace` F♯ can only contain local types and modules

* Cannot contain values or functions ❗
* By comparison, it's the same in C♯ with `namespace` that contains classes / enums only

What about nested namespaces?

* Only happens declaratively `namespace [parent.]identifier`
* 2 namespaces declared in the same file = ~~not nested~~ but independent

## Scope

* Several files can share the same namespace
* Several namespaces can be declared in a single file
  * They will not be ~~nested~~
  * May cause confusion ❗

☝ **Recommendations** \

* **Only** one namespace per file, declared at the top
* Organize files by grouping them in directories with the same name as the namespace

<table>
<thead><tr><th width="160">Level</th><th>Folder</th><th>Namespace</th></tr></thead>
<tbody>
<tr><td>0 <em>(root)</em></td><td><code>/</code></td><td><code>Root</code></td></tr>
<tr><td>1</td><td><code>/Lev1</code></td><td><code>Root.Lev1</code></td></tr>
<tr><td>2</td><td><code>/Lev1/Lev2</code></td><td><code>Root.Lev1.Lev2</code></td></tr>
</tbody></table>

## 🚀 Recursive namespace

Extends the default unidirectional visibility: from bottom to top \
→ each element can see all the elements in a recursive namespace

```fsharp
namespace rec Fruit

type Banana = { Peeled: bool }
    member this.Peel() =
        BananaHelper.peel  // `peel` not visible here without the `rec`

module BananaHelper =
    let peel banana = { banana with Peeled = true }
```

⚠️ **Drawbacks:** slow compilation, risk of circular reference

☝ **Recommendation:** handy but only for very few use cases
