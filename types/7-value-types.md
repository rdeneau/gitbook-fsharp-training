# Value types

Regular tuple/record/union are reference-types by default, but it's possible to get them as value-types

* Instances stored on the _Stack_ rather than in the _Heap_
* Records, Unions: `[<Struct>]` attribute
* Tuples, Anonymous Records: `struct` keyword

## Struct tuples & anonymous records

```fsharp
// Struct tuple
let a = struct (1, 'b', "Three") // struct (int * char * string)

// Struct anonymous record
let b = struct {| Num = 1; Char = 'b'; Text = "Three" |}
```

## Struct records & unions

```fsharp
// Struct record
[<Struct>]
type Point = { X: float; Y: float }
let p = { X = 1.0; Y = 2.3 } // val p: Point = { X = 1.0; Y = 2.3 }

// Struct union: unique fields labels are required❗
[<Struct>]
type Multicase =
    | Int  of i: int
    | Char of c: char
    | Text of s: string
let t = Int 1 // val t: Multicase = Int 1
```

## ⚖️ Pros/Cons

* ✅ Efficient because no _garbage collection_
* ⚠️ Passed by value → memory pressure

**Recommendations:**

> Consider structs for small types with high allocation rates

🔗 [F# coding conventions / Performance](https://learn.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#performance)
