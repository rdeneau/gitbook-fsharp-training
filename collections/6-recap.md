# 📜 Recap

## Types

5 collections including 4 functional/immutable

`List`: default choice / _Versatile, Practical_ \
→ `[ ]`, Operators: _Cons_ `::`, _Append_ `@`, Pattern matching

`Array`: fixed-size, mutable, performance-oriented (e.g. indexer) \
→ `[| |]` less handy to write than `[ ]`

`Seq`: deferred evaluation (_Lazy_), infinite sequence

`Set`: unique elements

`Map`: values by unique key

## API

**Rich** \
→ Hundreds of functions >> Fifty for LINQ

**Homogeneous** \
→ Common syntax and functions
→ Functions preserve the collection type (≠ LINQ sticked to `IEnumerable<>`)

**Semantic** \
→ Function names closer to JS

### Comparison C♯, F♯, JavaScript

| C♯ LINQ                       | F♯                    | JS `Array`           |
| ----------------------------- | --------------------- | -------------------- |
| `Select()`, `SelectMany()`    | `map`, `collect`      | `map()`, `flatMap()` |
| `Any(predicate)`, `All()`     | `exists`, `forall`    | `some()`, `every()`  |
| `Where()`, ×                  | `filter`, `choose`    | `filter()`, ×        |
| `First()`, `FirstOrDefault()` | `find`, `tryFind`     | ×, `find()`          |
| ×                             | `pick`, `tryPick`     | ×                    |
| `Aggregate([seed]])`          | `fold`, `reduce`      | `reduce()`           |
| `Average()`, `Sum()`          | `average`, `sum`      | ×                    |
| `ToList()`, `AsEnumerable()`  | `List.ofSeq`, `toSeq` | ×                    |
| `Zip()`                       | `zip`                 | ×                    |

## BCL types

- `Array`
- `ResizeArray` for C# List
- _Dictionaries:_ `dict`, `readOnlyDict`

For interop or performance

## 🕹️ Exercises

On [Exercism.io / F# track](https://exercism.io/tracks/fsharp)

| Exercise            | Level   | Topics                 |
|---------------------|---------|------------------------|
| High Scores         | Easy    | `List`                 |
| Protein Translation | Medium+ | `Seq`/`List` 💡        |
| ETL                 | Medium  | `Map` of `List`, Tuple |
| Grade School        | Medium+ | `Map` of `List`        |

☝ **Pre-requisites:** \
→ Create an account, with GitHub for instance \
→ Solve the first exercises to unlock the access to the one above

💡 **Tips:**
→ `string` is a `Seq<char>`
→ What about `Seq.chunkBySize`?

## 🔗 Additional resources

- [All functions, with their cost in O(?)](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/fsharp-collection-types#table-of-functions)
- [Choosing between collection functions](https://fsharpforfunandprofit.com/posts/list-module-functions/), F# for fun and profit (2015)
- [An F# Primer for curious C# developers - Work with collections](https://laenas.github.io/posts/01-fs-primer.html#work-with-collections), 2020
- [Formatage des collections](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#formatting-lists-and-arrays)
