# Syntax: Functions

## Named functions

* Declared with`let` _(like a variable)_
* Naming convention: **camelCase**
* No `return` keyword : the function always returns the last expression in its body
* No `()` around all parameters, no `,` between parameters\
  But `()` are necessary around each parameter to:\
  \- add the type annotation (1) \
  \- deconstruct an object (2)

```fsharp
let square x = x * x  // Function with 1 parameter
let res = square 2    // Returns 4

// (1) Parentheses required for annotations of type
let square' (x: int) : int = x * x

// (2) Brackets required when deconstructing an object
//     (here it's a single-case discriminated union 📍
let hotelId (HotelId value) = value
```

### Functions of 0 parameter and more than 1 parameter

For a function of 2 or more parameters, separate the parameters with a **space**. \
Same syntax to call it with the arguments.

<pre class="language-fsharp"><code class="lang-fsharp"><strong>// Function with 2 parameters
</strong>let add x y = x + y  // val add: x: int -> y: int -> int

// Call with the 2 arguments
let res = add 1 2    // val res: int = 3
</code></pre>

:warning: The inference of the `add` function can be confusing: the `+` works for any numbers and for strings too, but `add` is limited to `int`! To get it work, we need to write `let inline add ...`\
→ Leads to a special kind of generics: statically resolved type parameters (SRTP) 📍

:warning:️ Trap: `,` is used to instantiate/deconstruct a tuple that is seen as a single parameter

```fsharp
let addByPair (x, y) = x + y
```

For a function without parameter, use`()` as parameter _(like in C#)_

```fsharp
let printHello () = printfn "Hello"
printHello ()
// Displays "Hello" in the console

let notAFunction = printfn "Hello"
// Directly displays "Hello" in the console and returns "nothing" 📍
```

### Multi-line function

**Indentation** required, but no need for `{}` \
Can contain sub-function

```fsharp
let evens list =
    let isEven x =  // 👈 Sub-function
        x % 2 = 0   // 💡 `=` equality operator - No `==` operator in F#
    List.filter isEven list
// val evens: list: int list -> int list

let res = evens [1;2;3;4;5]
// val res: int list = [2; 4]
```

### Anonymous function

A.k.a. **Lambda**, arrow function

* Syntax: \
  `fun {parameters} -> body`  \
  &#xNAN;_(`{parameters} ⇒ body` in C#)_
* In general, we need `()` around for precedence reason

```fsharp
let evens' list = List.filter (fun x -> x % 2 = 0) list
```

### \_.Member shorthand (F# 8)

```fsharp
type Person = { Name: string; Age: int }
let people = [{ Name = "Alice"; Age = 30 }; { Name = "Bob"; Age = 5 }]

// Regular lambda (Shorthand not possible)
let adults = people |> List.filter (fun person -> person.Age >= 18)
// val adults: Person list = [{ Name = "Alice"; Age = 30 }]

// Member chain shorthand
let uppercaseNames = people |> List.map _.Name.ToUpperInvariant()
// val uppercaseNames: string list = ["ALICE"; "BOB"]
```

### Naming convention related to functions

It's usual in F# to use short names:

* `x`, `y`, `z` : parameters for values of the same type
* `f`, `g`, `h` : parameters for input functions
* `xs` : list of `x`&#x20;
* `_` : _discard_ an element not used _(like in C♯ 7.0)_

☝️ Suited for a short function body or for a generic function:

```fsharp
// Function that simply returns its input paameter, whatever its type
let id x = x

// Composition of 2 functions
let compose f g = fun x -> g (f x)
```

🔗 [When x, y, and z are great variable names](https://blog.ploeh.dk/2015/08/17/when-x-y-and-z-are-great-variable-names/) by Mark Seemann

## Piping

_Pipe_ operator `|>` : same idea that in UNIX with `|` \
→ `value |> function` Send a value to a function\
→ Can fit natural reading order: "subject verb"\
→ Same order than with OOP: object.method

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

### _Pipeline: chain of pipings_

Can be a way to have a code closed to a natural way of thinking, where the input data is passed from functions to functions, in a data flow, without intermediary variable👍

Similar to a fluent API in C♯ but native: in C♯ we need to return the object at the end of each method👍

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

## Expression `if/then/else`

In F♯, `if/then(/else)`  is an expression, not a statement, so every branch (`if-then` and `else`) should return a value and both returned values should be type-compatible.

💡 `if b then x else y` ≃ C♯ ternary operator `b ? x : y`

```fsharp
let isEven n =
    if n % 2 = 0 then
        "Even"
    else
        "Odd"
```

☝ When `then` returns "noting", `else` is optional:

```fsharp
let printIfEven n msg =
    if n |> isEven then
        printfn msg
```

## Expression `match`

```fsharp
let translate civility =
    match civility with
    | "Mister" -> "Monsieur"
    | "Madam"  -> "Madame"
    | "Miss"   -> "Mademoiselle"
    | _        -> ""   // 👈 wilcard `_`
```

Equivalent in C♯ 8 :

```csharp
public static string Translate(string civility) =>
    civility switch {
        "Mister" => "Monsieur"
        "Madam"  => "Madame"
        "Miss"   => "Mademoiselle"
        _        => ""
    }
```

## Exception

### Handling Exception

→  `try/with` expression

```fsharp
let tryDivide x y =
   try
       Some (x / y)
   with :? System.DivideByZeroException ->
       None
```

:warning: **Trap**: it's `with` keyword, not `catch` keyword, contrary to C#.

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
