# Syntax: Bases

## Comments

```fsharp
(* This is block
   comment *)


// And this is line comment


/// XML doc summary


/// <summary>
/// Full XML doc
/// </summary>
```

## Variables

* Values for the purists
* Keyword:`let` to declare/name a value
* No need for `;` at the end of the declaration
* It creates a _Binding_ that is immutable by default
  * ≃ `const` in JS, `readonly` members in C♯
* Mutable binding with `let mutable`
  * ≃ `let` en JS, `var` en C♯
  * ⚠️ The assignment operator is `<-`, not `=` used for equility
  * Use it sparingly, on a limited scope

```fsharp
let x = 1
x <- 2
// 💥 error FS0027: This value is not mutable. Consider using the mutable keyword...

let mutable x = 1
x <- 2 // ✅ Autorisé
```

### Traps ⚠️

Don't confuse the mutability of a variable (the part on the left-side of the binding) with the mutability of an object (the part on the right-side of the binding).

* `let` prevents the binding from being modified, i.e. assigning a new value to a variable.
* `let` does not prevent an object from mutating...
* ...with one exception: **mutable structures** (e.g. [ArrayCollector](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-compilerservices-arraycollector-1.html)) working by value: when you call a method that modifies a structure (e.g. `Add` or `Close`), you get a new copy. It's `let mutable` that allows you to keep a reference to this new value.

```fsharp
open FSharp.Core.CompilerServices

let collectorKo = ArrayCollector<int>()
collectorKo.Add(1)
collectorKo.Add(2)
let result = collectorKo.Close()
// val result: int array = [||]

let mutable collector = ArrayCollector<int>()
collector.Add(1)
collector.Add(2)
let result = collector.Close()
// result it: int array = [|1; 2|]
```

## Names

* Same constraints on variable naming than in C♯
* ... except the apostrophe `'`  _(tick)_
  * allowed in name in the middle or at the end, but not at the beginning!
  * at the end of the name → indicates a variant _(code convention)_
* Between double _backticks_&#x20;
  * allow any character, in particular whitespaces, except line breaks

```fsharp
let x = 1
let x' = x + 1

// Works on keyword too! But avoid it because it's confusing!
let if' b t f = if b then t else f

let ``123 456`` = "123 456"
// 💡 Auto-complétion : no need to enter the ``, just the 123
```

## _Shadowing_

* Use to redefine a value with a name already used above\
  → The previous value is no longer accessible in the current scope
* Not allowed at `module` level but allowed in a sub-scope
* Convenient but can be misleading\
  → Not recommended, except in special cases

```fsharp
let a = 2

let a = "ko"  // 💥 Error FS0037: Duplicate definition of value 'a'

let b =
    let a = "ok" // 👌 No compilation error
    // `a` is bound to the "ok" string (not the previous value: 2)
    // in all the rest of the b expression
    let a = "ko" // 👌 Consecutive shadowings are possible!
    ...
```

## Type Annotation

* Optional thanks to inference
* The Type is declared after the name `name: type` _(like in TypeScript)_
* The value is mandatory, even with `mutable` which is a good constraint for the code 👍

```fsharp
let x = 1       // Type inferred (int)
let y: int = 2  // Type explicit

let z1: int
// 💥 Error FS0010: Incomplete structured construct at or before this point
// in binding. Expected '=' or other token. 

let mutable z2: int
// 💥 Same error
```

## Constant

* _What:_ Variable erased during compilation, every usage is replaced by the value
  * ≃ `const` C♯ - same idea than `const enum` in TypeScript
* _How:_ Value decorated with the `Literal` attribute\
  ⚠️ Attributes are between `[< >]` \
  → It's a frequent beginner error to use`[ ]` _(like in C♯)_
* Recommended naming convention : PascalCase

```fsharp
[<Literal>] // Line break required before the `let`
let AgeOfMajority = 18

let [<Literal>] Pi = 3.14 // Also possible but not recommended by MS/Fantomas formatter
```

## Nombre

```fsharp
let pi = 3.14             // val pi : float = 3.14       • System.Double
let age = 18              // val age : int = 18          • System.Int32
let price = 5.95m         // val price : decimal = 5.95M • System.Decimal
```

⚠️ No implicit conversion between number types

→ 💡 Use `int`, `float`, `decimal` helper functions to do this conversion

```fsharp
let i = 1
i * 1.2;;  // 💣 error FS0001: The type 'float' does not match the type 'int'
​
float 3;;             // val it : float = 3.0
decimal 3;;           // val it : decimal = 3M
int 3.6;;             // val it : int = 3
int "2";;             // val it : int = 2
```

☝️ Note that this rule has been relaxed in some cases in [F# 6](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/fsharp-6#additional-implicit-conversions).

## String

```fsharp
let name = "Bob"              // val name : string = "Bob"

// String formatting (available from the get go)
let name2 = sprintf "%s Marley" name  // val name2 : string = "Bob Marley"

// String interpolation (F♯ 5)
let name2' = $"{name} Marley"  // val name2' : string = "Bob Marley"

// Type safe string interpolation
let rank = 1
let name2'' = $"%s{name} Marley, number %i{rank}"
// val name2'': string = "Bob Marley, number 1"

// Access to a character by its index (>= 0) (F♯ 6)
let initial = name2[0]         // val initial :  char = 'B'
let initial = name2.[0]        // ☝️ Previous syntax, still supported

// String slicing (F♯ 6) (Alternative to x.Substring(index [, length]) method)
let firstName = name2[0..2]   // val firstName : string = "Bob"
let lastName  = name2[4..]    // val lastName: string = "Marley"
```

Additional syntax:

```fsharp
// Verbatim string: idem C♯
let verbatimXml = @"<book title=""Paradise Lost"">"

// Triple-quoted string : no need to esapce the double-quotes `"`
let tripleXml = """<book title="Paradise Lost">"""

// Regular strings accept line breaks but do not trim whitespaces
let poemIndented = "
    The lesser world was daubed
    By a colorist of modest skill
    A master limned you in the finest inks
    And with a fresh-cut quill."

// Solution : backslash strings :
// Whitespaces (space and line break) are ignored 
// between the \ terminating a line and the following non-whitespace character
// (hence the \n to add line breaks):
let poem = "\
    The lesser world was daubed\n\
    By a colorist of modest skill\n\
    A master limned you in the finest inks\n\
    And with a fresh-cut quill."

// We can also combine line breaks and backslash strings:
let poemWithoutBackslashN = "\
    The lesser world was daubed
\
    By a colorist of modest skill
\
    A master limned you in the finest inks
\
    And with a fresh-cut quill."
```

### String interpolation in F# 8

An interpolated string cannot contain braces (`$"{xxx}"`) unless they are doubled (`$"{{xxx}}"`). Since F# 8, the $ character is doubled (`$$$` $$) or tripled ($$) to indicate the number of braces from which interpolation starts, respectively `{{ }}` and `{{{ }}}`.

```fsharp
let classAttr = "bold"
let cssNew = $$""".{{classAttr}}:hover {background-color: #eee;}"""
```

**Conclusion**: there are many many ways to write a string in F#!

### Encoding

String literals are encoded in **Unicode**:

{% code fullWidth="false" %}
```fsharp
let unicodeString1 = "abc"  // val unicodeString1: string = "abc"
let unicodeString2 = "ab✅" // val unicodeString2: string = "ab✅"
```
{% endcode %}

We can work in **ASCII** using the `B` suffix, but in this case we get a `byte array`:

{% code fullWidth="false" %}
```fsharp
let asciiBytes = "abc"B
// val asciiBytes1: byte array = [|97uy; 98uy; 99uy|]

let asciiBytesKO = "ab🚫"B
// 💥 Error FS1140: This byte array literal contains characters
//    that do not encode as a single byte
```
{% endcode %}

💡 Works also for character: `'a'B`.

## Lists

The `List` in F♯ is immutable and is different from `System.Collection.Generic.List<T>` .

```fsharp
let abc = [ 'a'; 'b'; 'c' ] // val abc : char list = ['a'; 'b'; 'c']
let a =
  [ 2
    3 ]  // val a : int list = [2; 3]
```

* Creation with`[]`, items separated by `;` ou line breaks + indentation\
  :warning: Common trap : using `,` to separate items\
  E.g. `[ 1, 2 ]`  compiles but it is not a list of 2 items! \
  It's a list of 1 item, a tuple of 2 elements! 📍
* Type annotation is using the ML notation (legacy from OCaml):\
  → `int list` equivalent to `List<int>` \
  ☝ Idiomatic only for a limited number of FSharp.Core types:\
  &#x20;      → `array`, `list` and `option` 📍

### List operators

* `::`  _Cons_ operator (_Cons_ meaning "construction")\
  → Add an item to the top of the list.
* `@` _Append_ operator\
  → Append 2 lists
* `..` _Range_ operator\
  → `min..max`: range of numbers or characters between a min and max (included)\
  → `min..step..max`: the step defines the difference between 2 consecutive numbers/characters

```fsharp
let ints = [2..5]                 // val ints : int list = [2; 3; 4; 5]
let ints' = 1 :: ints             // val ints' : int list = [1; 2; 3; 4; 5]
let floats = [ 2. .. 5. ]         // val floats: float list = [2.0; 3.0; 4.0; 5.0]

let chars = [ 'a' .. 'd' ]        // val chars : char list = ['a'; 'b'; 'c'; 'd']
let chars' = chars @ [ 'e'; 'f' ] // val chars' : char list = ['a'; 'b'; 'c'; 'd'; 'e'; 'f']
let e = chars'[4]                 // val e: char = 'e'
```

⚠️ A space must be placed before `[]` to create a list, to distinguish it from the access by index.

### `List`  module

This module contains functions for manipulating one or more lists.

| F♯ List            | C♯ LINQ (IEnumerable)      | JS Array             |
| ------------------ | -------------------------- | -------------------- |
| `map`, `collect`   | `Select()`, `SelectMany()` | `map()`, `flatMap()` |
| `exists`, `forall` | `Any(predicate)`, `All()`  | `some()`, `every()`  |
| `filter`           | `Where()`                  | `filter()`           |
| `find`, `tryFind`  | ×                          | `find()`             |
| `fold`, `reduce`   | `Aggregate([seed]])`       | `reduce()`           |
| `average`, `sum`   | `Average()`, `Sum()`       | ×                    |

☝ **Other functions :** see the [documentation](syntax-bases.md#syntaxe-cle)
