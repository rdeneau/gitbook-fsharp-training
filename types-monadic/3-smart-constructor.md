# Smart constructor

## Purpose

🔗 [Making illegal states unrepresentable](https://kutt.it/MksmkG), *F♯ for fun and profit*

- Design to prevent invalid states
  - Encapsulate state *(all primitives)* in an object
- *Smart constructor* guarantees a valid initial state
  - Validates input data
  - If Ko, returns "nothing" (`Option`) or an error (`Result`)
  - If Ok, returns the created object wrapped in an `Option` / a `Result`

## Encapsulate the state in a type

→ *Single-case (discriminated) union*: `Type X = private X of a: 'a...` \
🔗 [Designing with types: Single case union types](https://fsharpforfunandprofit.com/posts/designing-with-types-single-case-dus/), *F♯ for fun and profit*

→ *Record*: `Type X = private { a: 'a... }` \
🔗 [You Really Wanna Put a Union There? You Sure?](https://kutt.it/cYP4gY), by *Paul Blasucci*

☝ `private` keyword:

- Hide object content
- Fields and constructor no longer visible from outside
- Smart constructor defined in companion module or static method

## Example #1

Smart constructor :
→ `tryCreate` function in companion module
→ Returns an `Option`

```fsharp
module Xxx.Types                            // 👈 Top-level module❗

type Latitude = private { Latitude: float } with
    member this.Value = this.Latitude       // 👈 Required because `.Latitude` is private too

[<RequireQualifiedAccess>]                  // 👈 Optional but recommended
module Latitude =
    let tryCreate (latitude: float) =
        if latitude >= -90. && latitude <= 90. then
            Some { Latitude = latitude }    // 👈 Constructor accessible here
        else
            None
```

Usages:

```fsharp
module Xxx.Usages

open Xxx.Types

let lat_ok = Latitude.tryCreate 45.  // Some { Latitude = 45.0 }
let lat_ko = Latitude.tryCreate 115. // None
```

{% hint style="warning" %}
### Access control

`private` keyword has not exactly the same meaning in F♯ as in C♯!

- In F♯, `private` indicates that the entity can be accessed only from the enclosing type or module.
- In our example, `private` is applied on the `Latitude` definition that is on the `Xxx.Types` module.
- `{ Latitude = ... }` and `latitude.Latitude` are accessible in `Xxx.Types` module as if there were no `private` keyword.
- In the 2nd code block, we are in another module. The `Latitude` definition is not accessible.
- We can use only `Latitude.tryCreate` and `latitude.Value`.
{% endhint %}

## Example #2

Smart constructor with:

- Static method `Of`
- Returns a `Result` with a `string` in the `Error` track.

```fsharp
type Tweet =
    private { Tweet: string }

    static member Of tweet =
        if System.String.IsNullOrEmpty tweet then
            Error "Tweet shouldn't be empty"
        elif tweet.Length > 280 then
            Error "Tweet shouldn't contain more than 280 characters"
        else Ok { Tweet = tweet }

let tweet1 = Tweet.Of "Hello world" // Ok { Tweet = "Hello world" }
```
