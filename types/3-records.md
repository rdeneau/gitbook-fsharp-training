# Records

## Records: key points

> Product type with named elements called _fields._

Alternative to tuples when they are imprecise, for instance `float * float`:

* Point? Coordinates? Vector?
* Real and imaginary parts of a complex number?

Eleviate the doubt by naming both the type and its elements:

```fsharp
type Point = { X: float; Y: float }
type Coordinate = { Latitude: float; Longitude: float }
type ComplexNumber = { Real: float; Imaginary: float }
```

## Declaration

Base syntax:

```fsharp
type RecordName =
    { Label1: type1
      Label2: type2
      ... }
```

☝️ Field labels in PascalCase, not ~~camelCase~~ → see [MS style guide](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#use-pascalcase-for-type-declarations-members-and-labels)

Complete syntax:

```fsharp
[ attributes ]                                // [<Struct>]
type [accessibility-modifier] RecordName =    // private, internal
    { [ mutable ] Label1: type1
      [ mutable ] Label2: type2
      ... }
    [ member-list ]                           // Properties, methods...
```

## Formatting styles

* Single-line: properties separated by `;`
* Multi-line: properties separated by line breaks
  * 3 variations: _Cramped, Aligned, Stroustrup_

```fsharp
// Single line
type PostalAddress = { Address: string; City: string; Zip: string }

// Cramped: historical ┆  // Aligned: C#-like      ┆  // Stroustrup: C++-like
type PostalAddress =   ┆  type PostalAddress =     ┆  type PostalAddress = {
    { Address: string  ┆      {                    ┆      Address: string
      City: string     ┆          Address: string  ┆      City: string
      Zip: string }    ┆          City: string     ┆      Zip: string
                       ┆          Zip: string      ┆  }
                       ┆      }
```

### Styles comparison

| Criterion                                                                | Best styles 🏆       |
| ------------------------------------------------------------------------ | -------------------- |
| Compactness                                                              | Single-line, Cramped |
| <p>Refacto Easiness<br><em>(re)indentation, fields (re)ordering</em></p> | Aligned, Stroustrup  |

☝️ **Recommendation:** _Strive for Consistency_\
→ Apply consistently the same multi-line style across a repository\
→ In addition, use the single-line style when relevant: line with < 80 chars

### Styles configuration

Fantomas configuration in the `.editorconfig` file:

```ini
max_line_length = 180
fsharp_multiline_bracket_style = cramped | aligned | stroustrup

fsharp_record_multiline_formatter = number_of_items
fsharp_max_record_number_of_items = 3
# or
fsharp_record_multiline_formatter = character_width
fsharp_max_record_width = 120
```

🔗 [https://fsprojects.github.io/fantomas/docs/end-users/Configuration.html#fsharp\_record\_multiline\_formatter](https://fsprojects.github.io/fantomas/docs/end-users/Configuration.html#fsharp_record_multiline_formatter)

## Members

👉 Members are declared after the fields

### Single-line style

```fsharp
// `with` keyword required
type PostalAddress = { Address: string; City: string; Zip: string } with
    member x.ZipAndCity = $"{x.Zip} {x.City}"

// Or use line breaks (recommended when >= 2 members)
type PostalAddress =
    { Address: string; City: string; Zip: string }

    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

### Multi-line _Cramped_ and _Aligned_ styles

☝️ 2 line breaks

```fsharp
type PostalAddress =
    { Address: string
      City: string
      Zip: string }

    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

### Multi-line _Stroustrup_ style

☝️ `with` keyword + 1 line break + indentation

```fsharp
type PostalAddress = {
    Address: string
    City: string
    Zip: string
} with
    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

## Conostruction: _record expression_

* Same syntax as an anonymous C♯ object without the `new` keyword
* All fields must be populated, but in any order (but can be confusing)
* Same possible styles: single/multi-lines

```fsharp
type Point = { X: float; Y: float }
let point1 = { X = 1.0; Y = 2.0 }
let pointKo = { Y = 2.0 }           // 💥 Error FS0764
//            ~~~~~~~~~~~ FS0764: No assignment given for field 'X' of type 'Point'
```

⚠️ **Trap:** differences declaration / instanciation\
&#x20;    → `:` for field type in record declaration\
&#x20;    → `=` for field value in record expression

## Deconstruction

* Fields are accessible by "dotting" into the object
* Alternative: deconstruction
  * Same syntax for deconstructing a _Record_ as for creating it 👍
  * Unused fields can be ignored 💡

```fsharp
let { X = x1 } = point1
let { X = x2; Y = y2 } = point1
```

⚠️ Additional members _(properties)_ cannot be deconstructed!

```fsharp
type PostalAddress =
    {
        Address: string
        City: string
        Zip: string
    }
    member x.CityLine = $"{x.Zip} {x.City}"

let address = { Address = ""; City = "Paris"; Zip = "75001" }

let { CityLine = cityLine } = address   // 💥 Error FS0039
//    ~~~~~~~~ The record label 'CityLine' is not defined
let cityLine = address.CityLine         // 👌 OK
```

## Inference

* A record type can be inferred from the fields used 👍 but not with members ❗
* As soon as the type is inferred, IntelliSense will work

```fsharp
type PostalAddress =
    { Address: string
      City: string
      Zip: string }

let department address =
    address.Zip.Substring(0, 2) |> int
    //     ^^^^ 💡 Infer that address is of type `PostalAddress`.

let departmentKo zip =
    zip.Substring(0, 2) |> int
//  ~~~~~~~~~~~~~ Error FS0072: Lookup on object of indeterminate type
```

## Pattern matching

Let's use an example: `inhabitantOf` is a function giving the inhabitants name _(in French)_ at a given address _(in France)_

```fsharp
type Address = { Street: string; City: string; Zip: string }

let department { Zip = zip } = int zip[0..1] // Address -> int

let private IleDeFrance = Set [ 75; 77; 78; 91; 92; 93; 94; 95 ]
let inIleDeFrance departmentNum = IleDeFrance.Contains(departmentNum) // int -> bool

let inhabitantOf address = // Address -> string
    match address with
    | { Street = "Pôle"; City = "Nord" } -> "Père Noël"
    | { City = "Paris" } -> "Parisien"
    | _ when department address = 78 -> "Yvelinois"
    | _ when department address |> inIleDeFrance -> "Francilien"
    | _ -> "Français"
```

## Name conflict

In F♯, typing is nominal, not structural as in TypeScript\
→ Use qualification to resolve ambiguity\
→ Even better: write ≠ types or put them in ≠ modules

```fsharp
type Person1 = { First: string; Last: string }
type Person2 = { First: string; Last: string }
let alice = { First = "Alice"; Last = "Jones"}  // val alice: Person2... (by proximity)

// ⚠️ Deconstruction
let { First = firstName } = alice   // Warning FS0667 (in F# 6)
//  ~~~~~~~~~~~~~~~~~~~~~  The labels of this record do not uniquely
//                         determine a corresponding record type

let { Person2.Last = lastName } = alice     // 👌 OK with qualification
let { Person1.Last = lastName } = alice     // 💥 Error FS0001
//                                ~~~~~ Type 'Person1' expected, 'Person2' given
```

## Modification: _copy and update_

Record is immutable, but easy to get a modified copy\
→ **copy and update** expression of a _Record_\
→ use multi-line formatting for long expressions

```fsharp
// Single-line
let address2 = { address with Street = "Rue Vivienne" }

// Multi-line
let address3 =
    { address with
        City = "Lyon"
        Zip  = "69001" }
```

### _Copy and update_: C♯ _vs_ F♯ _vs_ JS

```csharp
// Record C♯ 9.0
address with { Street = "Rue Vivienne" }
```

```fsharp
// F♯ copy and update
{ address with Street = "Rue Vivienne" }
```

```typescript
// Object destructuring with spread operator
{ ...address, street: "Rue Vivienne" }
```

### _Copy and update_ limits (< F♯ 8)

Reduced readability with several nested levels

```fsharp
type Street = { Num: string; Label: string }
type Address = { Street: Street }
type Person = { Address: Address }

let person = { Address = { Street = { Num = "15"; Label = "rue Neuf" } } }

let person' =
    { person with
        Address =
          { person.Address with
              Street =
                { person.Address.Street with
                    Num = person.Address.Street.Num + " bis" } } }
```

### _Copy and update_: F♯ 8 improvements

```fsharp
type Street = { Num: string; Label: string }
type Address = { Street: Street }
type Person = { Address: Address }

let person = { Address = { Street = { Num = "15"; Label = "rue Neuf" } } }

let person' =
    { person with
        Person.Address.Street.Num = person.Address.Street.Num + " bis" }
```

{% hint style="warning" %}
#### Qualification

Usually, we have to qualify the field to avoid a compilation error: see `Person.Address`.
{% endhint %}
