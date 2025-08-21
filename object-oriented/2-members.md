# Members

Additional elements in type definition _(class, record, union)_

* _(Event)_
* Method
* Property
* Indexed property
* Operator overload

## Static and instance members

Static member: `static member member-name ...`.

Instance member:

* Concrete member: `member self-identifier.member-name ...`
* Abstract member: `abstract member member-name: type-signature`
* Virtual member = requires 2 declarations
  1. Abstract member
  2. Default implementation: `default self-identifier.member-name ...`
* Override virtual member: `override self-identifier.member-name ...`

☝ `member-name` in PascalCase _(.NET convention)_

☝ No `protected` member!

## _Self-identifier_

* In C♯, Java, TypeScript : `this`
* In VB : `Me`
* In F♯ : we can choose → `this`, `self`, `me`, any valid _identifier_...

**Declaration:**

1. For the primary constructor❗: with `as` → `type MyClass() as self = ...`
   * ⚠️ Can be costly
2. For a member: `member me.Introduce() = printfn $"Hi, I'm {me.Name}"`
3. For a member not using it: with `_` _(since F♯ 6)_ → `member _.Hi() = printfn "Hi!"`

## Call a member

Calling a static member\
→ Prefix with the type name: `type-name.static-member-name`

Calling an instance member inside the type\
→ Prefix with _self-identifier_: `self-identifier.instance-member-name`

Calling an instance member from outside the type\
→ Prefix with instance-name: `instance-name.instance-member-name`

## Method

≃ Function attached directly to a type

2 forms of parameter declaration:

1. Curried parameters = FP style
2. Parameters in tuple = OOP style
   * Better interop with C♯
   * Only mode allowed for constructors
   * Support named, optional, arrayed parameters
   * Support overloads

```fsharp
// (1) Tuple form (the most classic)
type Product = { SKU: string; Price: float } with
    member this.TupleTotal(qty, discount) =
        (this.Price * float qty) - discount  // (A)

// (2) Curried form
type Product' =
    { SKU: string; Price: float }
    member me.CurriedTotal qty discount =
        (me.Price * float qty) - discount  // (B)
```

☝ `with` required in ① but not in ② because of indentation\
&#x20;   → `end` can end the block started with `with` _(not recommended)_

☝ `this.Price` Ⓐ and `me.Price` Ⓑ\
&#x20;   → Access to instance via _self-identifier_ defined by member

## Named arguments

Calls a tuplified method by specifying parameter names:

```fsharp
type SpeedingTicket() =
    member _.SpeedExcess(speed: int, limit: int) =
        speed - limit

    member x.CalculateFine() =
        if x.SpeedExcess(limit = 55, speed = 70) < 20 then 50.0 else 100.0
```

Useful to:

* Clarify a usage for the reader or compiler (in case of overloads)
* Choose the order of arguments
* specify only certain arguments, the others being optional

☝ Arguments _after a named argument_ must be named too.

## Optional parameters

Allows you to call a tuplified method (including constructors) without specifying all the parameters.

Optional parameter:

* Declared with `?` in front of its name → `?arg1: int`
* In the body of the method, wrapped in an `Option` → `arg1: int option`
  * You can use `defaultArg` to specify the **default value**
  * But the default value does not appear in the signature!

When the method is called, the argument can be specified either:

* Directly in its type → `Method(arg1 = 1)`
* Wrapped in an `Option` if named with prefix `?` → `Method(?arg1 = Some 1)`

☝ Other syntax for interop .NET: `[<Optional; DefaultParameterValue(...)>] arg`

**Example:**

```fsharp
type DuplexType = Full | Half

type Connection(?rate: int, ?duplex: DuplexType, ?parity: bool) =
    let duplex = defaultArg duplex Full
    let parity = defaultArg parity false
    let defaultRate = match duplex with Full -> 9600 | Half -> 4800
    let rate = defaultArg rate defaultRate
    do printfn "Baud Rate: %d - Duplex: %A - Parity: %b" rate duplex parity

let conn1 = Connection(duplex = Full)
let conn2 = Connection(?duplex = Some Half)
let conn3 = Connection(300, Half, true)
```

☝ Notice the _shadowing_ of parameters by variables of the same name\
`let parity (* bool *) = defaultArg parity (* bool option *) Full`

### .NET optional parameters

There is another possibility to declare optional parameters, based on attributes. It's less handy but required for .NET interop or for using other attributes on parameters.

`[<Optional; DefaultParameterValue(...)>] arg`

The `Optional` and `DefaultParameterValue` attributes are available in `open System.Runtime.InteropServices`.

Example: tracing the call to a function by retrieving its name from the `CallerMemberName` attribute in `System.Runtime.CompilerServices` (\*)

```fsharp
open System.Runtime.CompilerServices
open System.Runtime.InteropServices

type Tracer() =
    static member trace(message: string,
                [<CallerMemberName; Optional; DefaultParameterValue("")>] memberName: string,
                [<CallerFilePath; Optional; DefaultParameterValue("")>] path: string,
                [<CallerLineNumber; Optional; DefaultParameterValue(0)>] line: int) =
        printfn $"Message: {message}"
        printfn $"Member name: {memberName}"
        printfn $"Source file path: {path}"
        printfn $"Source line number: {line}"

open type Tracer

let main() =
    trace "foo"

main();;
// Message: foo
// Member name: main
// Source file path: C:\Users\xxx\stdin
// Source line number: 18
```

(\*) Documentation :link: : [Caller information - F# | Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/caller-information)

### Using `|>` pipe operator

You can use `|>` with a method with:

* 1 parameter
* 2 parameters, the last of which is .NET optional

```fsharp
open System.Runtime.InteropServices

type LogLevel =
    | Trace = 1
    | Error = 2

type Logger() =
    member _.Log(message: string, [<Optional; DefaultParameterValue(LogLevel.Trace)>] level: LogLevel) =
        printfn $"[{level}] {message}"

let logger = Logger()

"My message" |> logger.Log
// > [Trace] My message
```

💡 If we want a 3rd parameter, we have to write a sub-lambda, like currying the function ourselves:

```fsharp
// ...
type Logger() =
    // ...
    member this.LogFrom(origin: string, [<Optional; DefaultParameterValue(LogLevel.Trace)>] level: LogLevel) =
        fun message -> this.Log($"Origin: {origin} | {message}", level)

let logger = Logger()

"Mon message" |> logger.LogFrom "Root"
// > [Trace] Origin: Root | Mon message
```

## Parameter array

Allows specifying a variable number of parameters of the same type\
→ Via `System.ParamArray` attribute on the **last** method argument

```fsharp
open System

type MathHelper() =
    static member Max([<ParamArray>] items) =
        items |> Array.max

let x = MathHelper.Max(1, 2, 4, 5)  // 5
```

💡 Equivalent of C♯ `public static T Max<T>(params T[] items)`

## Call C♯ method _TryXxx()_

❓ How to call in F♯ a C♯ method `bool TryXxx(args, out T outputArg)`?\
&#xNAN;_(Example: `int.TryParse`, `IDictionnary::TryGetValue`)_

* 👎 Use F♯ equivalent of `out outputArg` but use mutation 😵
* ✅ Do not specify `outputArg` argument
  * Change return type to tuple `bool * T`
  * `outputArg` becomes the 2nd element of this tuple

```fsharp
  match System.Int32.TryParse text with
  | true, i  -> printf $"It's the number {i}."
  | false, _ -> printf $"{text} is not a number."
```

## Call method _Xxx(tuple)_

❓ How do you call a method whose 1st parameter is itself a tuple?!

Let's try:

```fsharp
let friendsLocation = Map.ofList [ (0,0), "Peter" ; (1,0), "Jane" ]
// Map<(int * int), string>
let peter = friendsLocation.TryGetValue (0,0)
// 💥 Error FS0001: expression supposed to have type `int * int`, not `int`.
```

💡 **Explanation:** `TryGetValue(0,0)` = method call in tuplified mode\
→ Specifies 2 parameters, `0` and `0`.\
→ `0` is an `int` whereas we expect an `int * int` tuple!

### Solutions

1. 😕 _Backward pipe_, but also confusing
   * `friendsLocation.TryGetValue <| (0,0)`
2. 👌 Double parentheses, but confusing syntax
   * `friendsLocation.TryGetValue((0,0))`
3. ✅ Use a function rather than a method
   * `friendsLocation |> Map.tryFind (0,0)`

## Method _vs_ Function

| Feature             | Function | Curried method | Tuplified method |
| ------------------- | -------- | -------------- | ---------------- |
| Partial application | ✅ yes    | ✅ yes          | ❌ no             |
| Named arguments     | ❌ no     | ❌ no           | ✅ yes            |
| Optional parameters | ❌ no     | ❌ no           | ✅ yes            |
| Params array        | ❌ no     | ❌ no           | ✅ yes            |
| Overload            | ❌ no     | ❌ no           | ✅ yes 1️⃣        |

**Notes**

1️⃣ If possible, prefer optional parameters to overloads.

2️⃣ Declaration order:

* Methods generally don't need to follow the top-down compilation rule.
* But it's required in the case of generic members\
  → See [https://stackoverflow.com/q/66358718/8634147](https://stackoverflow.com/q/66358718/8634147)
* We recommend declaring members from top to bottom, to ensure consistency with the rest of the code.

## Method _vs_ Function (2)

| Feature                   | Function      | Static method   | Instance method    |
| ------------------------- | ------------- | --------------- | ------------------ |
| Naming                    | camelCase     | PascalCase      | PascalCase         |
| Support for `inline`      | ✅ yes         | ✅ yes           | ✅ yes              |
| Recursive                 | ✅ if `rec`    | ✅ yes           | ✅ yes              |
| Inference of `x` in       | `f x` → ✅ yes | `K.M x` → ✅ yes | `x.M()` → ❌ no     |
| Can be passed as argument | ✅ yes : `g f` | ✅ yes : `g T.M` | ❌ no : `g x.M` 1️⃣ |

1️⃣ Alternatives:\
&#x20; → F♯ 8: shorthand members → `g _.M()`\
&#x20; → Wrap in lambda → `g (fun x -> x.M())`

### Static methods _vs_ companion module

Companion module is more idiomatic → default choice.

Static methods are interesting in some use cases:

* Usage easier due to optional parameters 1️⃣
* Usage more readable due to named arguments 3️⃣
* Usage terser to instanciate record with several fields:
  * Depending on your use of Fantomas and its configuration, multi-line record expression can be verbose
  * A factory method call is usually formatted in a single line, hence terser. 2️⃣
  * When field labels are necessary for code clarity, we can use named arguments.
* Record expressions can be ambiguous: we are not sure of which type it is.
  * A factory method can help resolve ambiguity: we force to use it qualified, hence the type is explicit.

```fsharp
type PersonName =
    { FirstName  : string
      MiddleName : string option
      LastName   : string }
    
    static member Create(firstName, lastName, ?middleName) =
      { FirstName  = firstName
        MiddleName = middleName
        LastName   = lastName }

// 1️⃣ no middleName
let johnDoe = PersonName.Create("John", "Doe")

// 2️⃣ multi-line → single-line
let jfk' =
    { FirstName  = "John"
      MiddleName = Some "Fitzgerald"
      LastName   = "Kennedy" }
// vs
let jfk = PersonName.Create("John", "Fitzgerald", "Kennedy")

// 3️⃣ Named arguments
let pierreLaurent = PersonName.Create(firstName = "Pierre", lastName = "Laurent")
```

## Properties

≃ Syntactic sugar hiding a _getter_ and/or a _setter_\
→ Allows the property to be used as if it were a field

There are 2 base ways to declare a property:

### Getter

`member this.Property = expression using this`

☝ The expression is evaluated on each call.

Example:

```fsharp
type Person = { FirstName: string; LastName: string } with
    member it.FullName = $"{it.LastName.ToUpper()} {it.FirstName}"

let joe = { FirstName = "Joe"; LastName = "Dalton" }
let s = joe.FullName  // "DALTON Joe"
```

`member _.Property = expression involving side-effect`

☝ This kind of property is generally not recommended. Prefer a method.

```fsharp
type Generator() =
    let random = new System.Random()
    // ❌ Avoid
    member _.NextValue = random.Next()
    // ✅ Prefer
    member _.Next() = random.Next()
```

### Automatic property

Automatic because a _backing field_ is generated by the compiler.

| Use case   | Syntax                                      | Equivalent in C♯                     |
| ---------- | ------------------------------------------- | ------------------------------------ |
| Read-only  | `member val Property = value`               | `public Type Property { get; }`      |
| Read/write | `member val Property = value with get, set` | `public Type Property { get; set; }` |

☝ The property returns the same value on each call, mutation with the _setter_ aside.

Example:

```fsharp
[<Struct>]
type PersonName(first: string, last: string) =
    member val First = first
    member val Last = last
    member val Full = $"{last.ToUpper()} {first}"

let joe = PersonName(first = "Joe", last = "Dalton")
let s = joe.Full  // "DALTON Joe"
```

☝️ `PersonName` is immutable and, as a _struc&#x74;_&#xD83D;�, has structural equality. It's the OO alternative to _records._

### Other cases

In other cases, the syntax is verbose: ([_details_](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/members/properties)).\
👉 When possible, prefer methods as they are more explicit.

### Properties and pattern matching

⚠️ Properties cannot be deconstructed\
→ Can only participate in pattern matching in `when` part

```fsharp
type Person = { First: string; Last: string } with
    member this.FullName = // Getter
        $"{this.Last.ToUpper()} {this.First}"

let joe = { First = "Joe"; Last = "Dalton" }
let { First = first } = joe  // val first : string = "Joe"
let { FullName = x } = joe
// 💥 ~~~~~~~~ Error FS0039: undefined record label 'FullName'

let salut =
    match joe with
    | _ when joe.FullName = "DALTON Joe" -> "Salut, Joe !"
    | _ -> "Hello!"
// val salut : string = "Salut, Joe !"
```

## Indexed properties

Allows access by index, as if the class were an array: `instance[index]`\
→ Useful for an ordered collection, to hide the implementation

Set up by declaring the member `Item`

```fsharp
member self-identifier.Item
    with get(index) =
        get-member-body
    and set index value =
        set-member-body
```

💡 Property _read-only_ (_write-only_) → declare only the _getter_ (_setter_)

☝ Notice the _setter_ parameters are curried

**Example :**

```fsharp
type Lang = En | Fr

type DigitLabel() =
    let labels = // Map<Lang, string[]>
        [| (En, [| "zero"; "one"; "two"; "three" |])
           (Fr, [| "zéro"; "un"; "deux"; "trois" |]) |] |> Map.ofArray

    member val Lang = En with get, set

    member me.Item with get i = labels[me.Lang][i]

let digitLabel = DigitLabel()
let v1 = digitLabel[1]     // "one"
digitLabel.Lang <- Fr
let v2 = digitLabel[2]     // "deux"
```

## Slice

> Same as indexed property, but with multiple indexes

**Declaration:** `GetSlice(?start, ?end)` method _(regular or extension)_

**Usage:** `..` operator

```fsharp
type Range = { Min: int; Max: int } with
    /// Defines a sub-range - newMin and newMax are optional and ignored if out-of-bounds
    member this.GetSlice(newMin, newMax) =
        { Min = max (defaultArg newMin this.Min) this.Min
        ; Max = min (defaultArg newMax this.Max) this.Max }

let range = { Min = 1; Max = 5 }
let slice1 = range[0..3] // { Min = 1; Max = 3 }
let slice2 = range[2..]  // { Min = 2; Max = 5 }
```

## Operator overload

Operator overloaded possible at 2 levels:

1. In a module, as a function\
   `let [inline] (operator-symbols) parameter-list = ...`
   * 👉 See session on functions
   * ☝ Limited: only 1 definition possible
2. In a type, as a member\
   `static member (operator-symbols) (parameter-list) =`
   * Same rules as for function form
   * 👍 Multiple overloads possible (N types × P _overloads_)

**Example:**

```fsharp
type Vector(x: float, y: float) =
    member _.X = x
    member _.Y = y

    override me.ToString() =
        let format n = (sprintf "%+.1f" n)
        $"Vector (X: {format me.X}, Y: {format me.Y})"

    static member (*)(a, v: Vector) = Vector(a * v.X, a * v.Y)
    static member (*)(v: Vector, a) = a * v
    static member (+) (v: Vector, w: Vector) = Vector(v.X + w.X, v.Y + w.Y)
    static member (~-)(v: Vector) = -1.0 * v // 👈 Unary '-' operator

let v1 = Vector(1.0, 2.0)   // Vector (X: +1.0, Y: +2.0)
let v2 = v1 * 2.0           // Vector (X: +2.0, Y: +4.0)
let v3 = 0.75 * v2          // Vector (X: +1.5, Y: +3.0)
let v4 = -v3                // Vector (X: -1.5, Y: -3.0)
let v5 = v1 + v4            // Vector (X: -0.5, Y: -1.0)
```
