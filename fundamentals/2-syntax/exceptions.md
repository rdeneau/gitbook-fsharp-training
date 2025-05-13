# Exceptions

In F♯, thanks to [4-unions.md](../../types/4-unions.md "mention") 📍, we can deal with expected errors using the [2-result.md](../../types-monadic/2-result.md "mention") and optional union types for custom error - see 🔗 [Railway oriented programming](https://fsharpforfunandprofit.com/rop/).

Still, as exceptions are quite common in .NET, F♯ supports them fully and elegantly.

## Type alias

F♯ provides the convenient`exn` alias for the `System.Exception` type.

## Defining exceptions

To define our own exceptions, we can use the `exception` keyword:

```fsharp
// Without inner data
exception ArgumentNullOrWhiteSpaceException

// With inner data
exception MustBePositiveException of int

// With inner data with labels
exception ArgumentMustBePositiveException of name: string * value: int
```

**☝️ Notes:**

* Naming convention: the exception name should end with `Exception`
* Data labels are recommended for code clarity.
* The syntax is close to a union type case for the exception's definition and its handling with pattern matching - see [#full-example](exceptions.md#full-example "mention") below.
* We can also define exceptions by inheriting from `System.Exception`. This is the C♯ style, but it is not recommended because it's less elegant for both definition and handling.

## Throwing exceptions

F♯ provides a set of convenient helpers:

* `raise (exn: exn)`&#x20;
  * throw a custom exception object
  * equivalent of `throw exn;` in C♯
* `reraise ()`&#x20;
  * used in a `(try...) with` block to propagate a handled exception up the call chain
  * equivalent of `throw;` in C♯
  * preserve the stack trace, contrary to `raise ex`
  * to do the same outside of a `(try...) with` block, it's more complicated\
    → 🔗 [https://stackoverflow.com/a/41202215/8634147](https://stackoverflow.com/a/41202215/8634147)
* `failwith (message: string)` and `failwithf (messageTemplate: string) (...args)`&#x20;
  * generate a general F♯ exception with the given message
  * `failwith` is the shortcut for `raise (Exception message)`
  * `failwithf` is less used since F♯ supports string interpolation.
* `invalidArg (argumentName: string) (message: string)`
  * generate a `System.ArgumentException`  for the given argument name and with the specified message
  * shortcut for `raise (ArgumentException(message, argumentName))`
* `nullArg (argumentName: string)`&#x20;
  * generate a `System.NullArgumentException`  for the given argument name
  * shortcut for `raise (ArgumentNullException argumentName)`
* `invalidOp (message: string)`
  * generate a `System.InvalidOperationException`  with the given message
  * shortcut for `raise (InvalidOperationException message)`

Examples:

```fsharp
let notImplemented () =
    failwith "Not implemented"

let notNull argumentName value =
    if isNull value then
        nullArg argumentName

let divide x y =
    match y with
    | 0.0 -> invalidArg (nameof y) "Divisor cannot be zero"
    | _ -> x / y
```

## Handling exceptions

→ `try/with` expression

```fsharp
let tryDivide x y =
   try
       Some (x / y)
   with :? System.DivideByZeroException ->
       None
```

**☝️ Notes:**

* The keyword used is `with`, not `catch`, contrary to C♯.
* There is no `try/with/finally` expression, only `try/finally` that we can nest in another `try/with`.
* The `with` block is using pattern matching:
  * `:? ExceptionType` to check if the handled exception has or derives from the given type.
  * `Failure message` is an active pattern to catch low-level exceptions that have exactly the`System.Exception` type. \
    → useful to handle exceptions raised by `failwith(f)`  helpers.
  * Exceptions defined using the `exception` keyword can be deconstructed\
    → see [#full-example](exceptions.md#full-example "mention") below.

## Full example

The following example demonstrates almost every topic we studied:

* How to define an exception type
* How to raise an exception using different helpers
* How to catch the exception and pattern match it

```fsharp
open System

exception ArgumentMustBePositiveOrZeroException of name: string * value: int

type Test =
    | FailWith of message: string
    | NotANumber of value: string
    | NotImplemented
    | NotPositive of name: string * value: int
    | Null of name: string
    | Valid of name: string * value: int

printfn "---"

[ FailWith "Unknown error"
  NotImplemented
  NotANumber "not a number"
  NotPositive("userId", -1)
  Null "userId"
  Valid("userId", 3) ]
|> List.iter (fun input ->
    try
        try
            match input with
            | FailWith message -> failwith message
            | NotANumber value -> Int32.Parse value |> ignore
            | NotImplemented -> raise (NotImplementedException())
            | NotPositive (name, value) -> raise (ArgumentMustBePositiveOrZeroException(name, value))
            | Null name -> nullArg name
            | Valid (name, value) -> printfn $"✅ %s{name} %d{value} is valid"
        with
        | :? ArgumentNullException as exn ->
            printfn $"💣 ArgumentNullException(ParamName = %s{exn.ParamName}, Message = %s{exn.Message})"
        | :? ArgumentException as exn ->
            printfn $"💣 ArgumentException(ParamName = %s{exn.ParamName}, Message = %s{exn.Message})"
        | :? FormatException as exn -> printfn $"💣 FormatException(Message = {exn.Message})"
        | ArgumentMustBePositiveOrZeroException (name, value) ->
            printfn $"💣 ArgumentMustBePositiveOrZeroException(Name = %s{name}, Value = %d{value})"
        | Failure message ->
            printfn $"💣 Failure(Message = %s{message})"
        | exn ->
            printfn $"💣 Exception(Type = %s{exn.GetType().Name}, Message = %s{exn.Message})"
    finally
        printfn "Input was: %A" input
        printfn "---"
)
```

<details>

<summary>Outputs</summary>

```
---
💣 Failure(Message = Unknown error)
Input was: FailWith "Unknown error"
---
💣 Exception(Type = NotImplementedException, Message = The method or operation is not implemented.)
Input was: NotImplemented
---
💣 FormatException(Message = The input string 'not a number' was not in a correct format.)
Input was: NotANumber "not a number"
---
💣 ArgumentMustBePositiveOrZeroException(Name = userId, Value = -1)
Input was: NotPositive ("userId", -1)
---
💣 ArgumentNullException(ParamName = userId, Message = Value cannot be null. (Parameter 'userId'))
Input was: Null "userId"
---
✅ userId 3 is valid
Input was: Valid ("userId", 3)
---
```

</details>

{% hint style="warning" %}
### Pattern order

Pay attention to pattern order

* `:? ArgumentNullException` is placed before `:? ArgumentException` because of the type inheritance relationship. Still, we are covered by the compiler: in case the 2 patterns are placed in the wrong order, we get the following warning: `This rule will never be matched`
* `Failure` and `ArgumentMustBePositiveOrZeroException` patterns can be placed anywhere above the last pattern `exn` because they target specific types.
* The pattern `exn` must be placed at the end because it's the most general one.
{% endhint %}

{% hint style="success" %}
### Deconstruction patterns

`Failure` and `ArgumentMustBePositiveOrZeroException` patterns support deconstruction:\
→ We can access the exception property elegantly compared to `?: XxxException as exn -> exn.Xxx`.
{% endhint %}
