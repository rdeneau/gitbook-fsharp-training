# Functions

## Named functions

* Declared a `let` binding _(like a variable)_
* Naming convention: **camelCase**
* No `return` keyword: the function always returns the last expression in its body
* No `()` around all parameters, no `,` between parameters
* `()` required around parameter with type annotation (1) or deconstruction (2)

```fsharp
let square x = x * x  // Function with 1 parameter
let res = square 2    // Returns 4

// (1) Parentheses required for annotations of type
let square' (x: int) : int = x * x

// (2) Brackets required when deconstructing an object
//     (here it's a single-case discriminated union 📍
let hotelId (HotelId value) = value
```

### Functions of 2 or more parameters

Separate parameters and arguments with **spaces**:

```fsharp
// Function with 2 parameters
let add x y = x + y  // val add: x: int -> y: int -> int

// Call with the 2 arguments
let res = add 1 2    // val res: int = 3
```

{% hint style="warning" %}
### Inference

The inference of the `add` function can be confusing: the `+` works for any numbers and for strings too, but `add` is limited to `int`! To get it work, we need to write `let inline add ...`\
→ Related to a special kind of generics: statically resolved type parameters (SRTP) 📍
{% endhint %}

⚠️️ `,` creates another kind of functions using tuples 📍

```fsharp
let addByPair (x, y) = x + y
// val addByPair: x: int * y: int -> int
```

## Functions without parameter

Use `()` _(like in C#)_

```fsharp
let printHello () = printfn "Hello"
// val printHello: unit -> unit
printHello ();;
// Hello

let notAFunction = printfn "Hello"
// Hello
// val notAFunction: unit = ()
```

☝️ `unit` means "nothing" 📍

### Multi-line function

**Indentation** required, but no need for `{}`\
Can contain sub-function

```fsharp
let evens list =
    let isEven x =  // 👈 Sub-function
        x % 2 = 0   // 💡 `=` equality operator - No `==` operator in F♯
    List.filter isEven list
// val evens: list: int list -> int list

let res = evens [1;2;3;4;5]
// val res: int list = [2; 4]
```

## Anonymous function

A.k.a. **Lambda**, arrow function

* Syntax: `fun {parameters} -> body` _(≠ in C♯ `{parameters} ⇒ body`)_
* In general, `()` required all around, for precedence reason

```fsharp
let evens' list = List.filter (fun x -> x % 2 = 0) list
```

### \_.Member shorthand (F♯ 8)

```fsharp
type Person = { Name: string; Age: int }

let people =
    [ { Name = "Alice"; Age = 30 }
      { Name = "Billy"; Age =  5 } ]

// Regular lambda (Shorthand not possible)
let adults = people |> List.filter (fun person -> person.Age >= 18)
// val adults: Person list = [{ Name = "Alice"; Age = 30 }]

// Member chain shorthand
let uppercaseNames = people |> List.map _.Name.ToUpperInvariant() // 👈👈
// val uppercaseNames: string list = ["ALICE"; "BILLY"]
```

## Naming convention related to functions

It's usual in F♯ to use short names:

* `x`, `y`, `z` : parameters for values of the same type
* `f`, `g`, `h` : parameters for input functions
* `xs` : list of `x`
* `_` : _discard_ an element not used _(like in C♯ 7.0)_

☝️ Suited for a short function body or for a generic function:

```fsharp
// Function that simply returns its input parameter, whatever its type
let id x = x

// Composition of 2 functions
let compose f g = fun x -> g (f x)
```

🔗 [When x, y, and z are great variable names](https://blog.ploeh.dk/2015/08/17/when-x-y-and-z-are-great-variable-names/) by Mark Seemann

## Piping

_Pipe_ operator `|>` : same idea that in UNIX with `|`\
→ `value |> function` send a value to a function\
→ match left-to-right reading order: "subject verb"\
→ same order than with OOP: object.method

```fsharp
let a = 2 |> add 3  // to read "2 + 3"

// We pipe a list to the "List.filter predicate" function
let evens = [1;2;3;4;5] |> List.filter (fun x -> x % 2 = 0)
```

```csharp
// ≃ C♯
var a = 2.Add(3);
var nums = new[] { 1, 2, 3, 4, 5 };
var evens = nums.Where(x => x % 2 == 0);
```

### Pipeline: chain of pipings

Style of coding to emphasize the data flowing from functions to functions\
→ without intermediary variable 👍

Similar to a built-in _fluent API_\
→ no need to return the object at the end of each method 👍

```fsharp
// Short syntax: in a single line fitting the screen width
let res = [1;2;3;4;5] |> List.filter (fun x -> x % 2 = 0) |> List.sum

// More readable with line breaks
let res' =
    [1; 2; 3; 4; 5]
    |> List.filter isOdd  // With `let isOdd x = x % 2 <> 0`
    |> List.map square    //      `let square x = x * x`
    |> List.map addOne    //      `let addOne x = x + 1`
```

## If/then/else expression

In F♯, `if/then(/else)` is an expression, not a statement, so every branch (`then` and `else`) should return a value and both returned values should be type-compatible.

```fsharp
let isEven n =
    if n % 2 = 0 then
        "Even"
    else
        "Odd"
```

💡 `if b then x else y` ≃ C♯ ternary operator `b ? x : y`

☝ When `then` returns "nothing", `else` is optional:

```fsharp
let printIfEven n msg =
    if n |> isEven then
        printfn msg
```

💡 We can use `elif` keyword instead of `else if`.

## Match expression

```fsharp
let translateInFrench civility =
    match civility with
    | "Mister" -> "Monsieur"
    | "Madam"  -> "Madame"
    | "Miss"   -> "Mademoiselle"
    | _        -> ""   // 👈 wilcard `_`
```

Equivalent in C♯ 8 :

```csharp
public static string TranslateInFrench(string civility) =>
    civility switch {
        "Mister" => "Monsieur"
        "Madam"  => "Madame"
        "Miss"   => "Mademoiselle"
        _        => ""
    }
```

## Exception

### Handling Exception

→ `try/with` expression

```fsharp
let tryDivide x y =
   try
       Some (x / y)
   with :? System.DivideByZeroException ->
       None
```

⚠️ **Trap**: the keyword used is `with`, not `catch`, contrary to C#.

💡 There is no `try/with/finally` expression, only `try/finally` that we can nest in another `try/with`.

### Throwing Exception

→ Helpers `failwith`, `invalidArg` and `nullArg`

```fsharp
let fn arg =
    if arg = null then nullArg (nameof arg)
    failwith "Not implemented"

let divide x y =
    if y = 0
    then invalidArg (nameof y) "Divisor cannot be zero"
    else x / y
```

🔗 Handling Errors Elegantly [https://devonburriss.me/how-to-fsharp-pt-8/](https://devonburriss.me/how-to-fsharp-pt-8/)
