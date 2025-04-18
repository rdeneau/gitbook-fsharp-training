# Class & Structure

## Class

Class in F♯ ≡ class in C♯

- Object-oriented building block
- Constructor of objects containing data of defined type and methods
- Can be static too

**Keyword:** we use the `type` keyword to define a class, like any other type in F♯.

```fsharp
type MyClass =
    // (static) let and do bindings...
    // Constructors...
    // Members...
```

## Class constructors

### Primary constructor

Non-static classes are generally followed by their **primary constructor:**

```fsharp
type CustomerName(firstName: string, lastName: string) =
    // Primary builder's body
    // Members...
```

☝ **Note:** The primary constructor parameters, here `firstName` and `lastName`, are visible throughout the class body.

### Secondary constructor

Syntax for defining another constructor: \
`new(argument-list) = constructor-body`

☝ In `constructor-body`, there must be a call the primary constructor.

```fsharp
type Point(x: float, y: float) =
    new() = Point(0, 0)
    // Members...
```

### Constructor parameters

☝ Constructors accept only tuples. They can not be curried!

### let bindings

We can use `let` bindings in the class definition to define variables (in fact class fields) and inner functions.

**Syntax:**

```fsharp
// Field.
[static] let [ mutable ] binding1 [ and ... binding-n ]

// Function.
[static] let [ rec ] binding1 [ and ... binding-n ]
```

☝ **Notes**

- Declared before class members
- Initial value mandatory
- Private
- Direct access: no need to qualify them with the `self-identifier`

**Example:**

```fsharp
type Person(firstName: string, lastName: string) =
    let fullName = $"{firstName} {lastName}"
    member _.Hi() = printfn $"Hi, I'm {fullName}!"

let p = Person("John", "Doe")
p.Hi()  // Hi, I'm John Doe!
```

### do bindings

We can use `do` bindings in the class definition to trigger side-effects *(like logging)* during the construction of the object or of the type, before the members.

**Syntax:**

```fsharp
[static] do expression
```

### Static constructor

There are no explicit static constructor in F♯ but we can use `static let` and `static do` bindings to perform "type initialization". They will be executed the fist time the type is used.

**Example:**

```fsharp
type K() =
    static let mutable count = 0

    // do binding exécuté à chaque construction
    do
        count <- count + 1

    member val CreatedCount = count

let k1 = K()
let count1 = k1.CreatedCount  // 1
let k2 = K()
let count2 = k2.CreatedCount  // 2
```

## Singleton

You can define a *Singleton* class by making the primary constructor private:

```fsharp
type S private() =
    static member val Instance = S()

let s = S.Instance  // val s : S
```

{% hint style="warning" %}
#### Warning

It's not thread-safe!
{% endhint %}

💡 **Alternative:** *single-case union*

```fsharp
type S = S
let s = S
```

## Empty class

To define a class without a body, use the verbose syntax `class ... end`:

```fsharp
type K = class end
let k = K()
```

## Generic class

There is no automatic generalization on type \
→ Generic parameters need to be specified:

```fsharp
type Tuple2_KO(item1, item2) = // ⚠️ 'item1' and 'item2': 'obj' type !
    // ...

type Tuple2<'T1, 'T2>(item1: 'T1, item2: 'T2) =  // 👌
    // ...
```

## Instanciation

Call one of the constructors, with tuple arguments

☝️ Don't forget `()` if no arguments, otherwise you get a function!

In a `let` binding: `new` is optional and not recommended \
→ `let v = Vector(1.0, 2.0)` 👌 \
→ `let v = new Vector(1.0, 2.0)` ❌

In a `use` binding: `new` mandatory \
→ `use d = new Disposable()`

## Property initialization

Properties can be initialized with setter at instantiation 👍

- Specify them as **named arguments** in the call to the constructor
- Place them after any constructor arguments

```fsharp
type PersonName(first: string) =
    member val First = first with get, set
    member val Last = "" with get, set

let p1 = PersonName("John")
let p2 = PersonName("John", Last = "Paul")
let p3 = PersonName(first = "John", Last = "Paul")
```

💡 Equivalent in C♯: `new PersonName("John") { Last = "Paul" }`

## Abstract class

Annotated with `[<AbstractClass>]`

One of the members is **abstract**:

1. Declared with the `abstract` keyword
2. No default implementation (with `default` keyword)
   *(Otherwise member is virtual)*

Inheritance with `inherit` keyword \
→ Followed by a call to the base class constructor

**Example:**

```fsharp
[<AbstractClass>]
type Shape2D() =
    member val Center = (0.0, 0.0) with get, set
    member this.Move(?deltaX: float, ?deltaY: float) =
        let x, y = this.Center
        this.Center <- (x + defaultArg deltaX 0.0,
                        y + defaultArg deltaY 0.0)
    abstract GetArea : unit -> float
    abstract Perimeter : float  with get

type Square(side) =
    inherit Shape2D()
    member val Side = side
    override _.GetArea () = side * side
    override _.Perimeter = 4.0 * side

let o = Square(side = 2.0, Center = (1.0, 1.0))
printfn $"S={o.Side}, A={o.GetArea()}, P={o.Perimeter}"  // S=2, A=4, P=8
o.Move(deltaY = -2.0)
printfn $"Center {o.Center}"  // Center (1, -1)
```

## Fields

2 kind of field: implicit or explicit

### Implicit fields

As we've already seen, `let` bindings implicitly define fields in the class...

### Explicit fields

**Syntax:** `val [ mutable ] [ access-modify ] field-name : type-name`

**Notes:**

- Their declaration cannot contain an initial value
- They are `public` by default, but can be compiled using a backing field:
  - `val mutable a: int` → public field
  - `val a: int` → internal field `a@` + property `a => a@`

### Field *vs* property

```fsharp
// Explicit fields readonly
type C1 =
    val a: int
    val b: int
    val mutable c: int
    new(a, b) = { a = a; b = b; c = 0 } // 💡 Constructor 2ndary "compact"

// VS readonly properties
type C2(a: int, b: int) =
    member _.A = a
    member _.B = b
    member _.C = 0

// VS auto-implemented property
type C3(a: int, b: int) =
    member val A = a
    member val B = b with get
    member val C = 0 with get, set
```

## Explicit field *vs* implicit field *vs* property

Explicit fields are **not often used**:

- Only for classes and structures
- Useful with native function manipulating memory directly
    *(Because fields order is preserved - see [SharpLab](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfA9MgBAYQBYEMC2ADhGgKYAeBwAlgMZUAuJxATiTjAPYB2wAngLAAoerwIlMARjQBeIWnloAbjmBocINFS705C5aoBGGrTsEK0XEgHcAFDihoDAShloA3mtc4A3I9cHfAF80VDRAXg3AQp3Mbgh6ZgBXGkZ45jQAJi4YHCpWNAAiGg5CHCSSPKEhUIA1AGU0AmYOBqoAS/oWljZOHgFhUXEMNLtjbQcjTW0XWTMFPBI8AxJUgH0AOgBBL115OYWltDWAIX8hIA===))*
- Need a `[<ThreadStatic>]` variable
- Interaction with F♯ class of code generated without primary constructor

Implicit fields / `let` bindings are quite common, to define intermediate variable during construction.

Other use cases match auto-implemented properties:

- Expose a value → `member val`
- Expose a mutable "field" → `member val ... with get, set`

## Structures

Alternatives to classes, but more limited inheritance and recursion features

Same syntax as for classes, but with the addition of:

- `[<Struct>]` attribute
- Or `struct...end` block *(more frequent)*

```fsharp
type Point =
    struct
        val mutable X: float
        val mutable Y: float
        new(x, y) = { X = x; Y = y }
    end

let x = Point(1.0, 2.0)
```
