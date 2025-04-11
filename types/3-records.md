# Records

## Records: key points

> Product type with named elements called *fields.*

Alternative to tuples when they are imprecise, for instance `float * float`:

- Point? Coordinates? Vector?
- Real and imaginary parts of a complex number?

Eleviate the doubt by naming both the type and its elements:

```fs
type Point = { X: float; Y: float }
type Coordinate = { Latitude: float; Longitude: float }
type ComplexNumber = { Real: float; Imaginary: float }
```

## Declaration

Base syntax:

```fs
type RecordName =
    { Label1: type1
      Label2: type2
      ... }
```

☝️ Field labels in PascalCase, not ~~camelCase~~ → see [MS style guide](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#use-pascalcase-for-type-declarations-members-and-labels)

Complete syntax:

```fs
[ attributes ]                                // [<Struct>]
type [accessibility-modifier] RecordName =    // private, internal
    { [ mutable ] Label1: type1
      [ mutable ] Label2: type2
      ... }
    [ member-list ]                           // Properties, methods...
```

## Formatting styles

- Single-line: properties separated by `;`
- Multi-line: properties separated by line breaks
  - 3 variations: *Cramped, Aligned, Stroustrup*

```fs
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

| Criterion                                                     | Best styles 🏆       |
|---------------------------------------------------------------|----------------------|
| Compactness                                                   | Single-line, Cramped |
| Refacto Easiness <br/> *(re)indentation, fields (re)ordering* | Aligned, Stroustrup  |

☝️ **Recommendation:** *Strive for Consistency*
→ Apply consistently the same multi-line style across a repository
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

🔗 https://fsprojects.github.io/fantomas/docs/end-users/Configuration.html#fsharp_record_multiline_formatter

## Members

👉 Members are declared after the fields

### Single-line style

```fs
// `with` keyword required
type PostalAddress = { Address: string; City: string; Zip: string } with
    member x.ZipAndCity = $"{x.Zip} {x.City}"

// Or use line breaks (recommended when >= 2 members)
type PostalAddress =
    { Address: string; City: string; Zip: string }

    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

### Multi-line *Cramped* and *Aligned* styles

☝️ 2 line breaks

```fs
type PostalAddress =
    { Address: string
      City: string
      Zip: string }

    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

### Multi-line *Stroustrup* style

☝️ `with` keyword + 1 line break + indentation

```fs
type PostalAddress = {
    Address: string
    City: string
    Zip: string
} with
    member x.ZipAndCity = $"{x.Zip} {x.City}"
    member x.CityAndZip = $"%s{x.City}, %s{x.Zip}"
```

## Record expression for instanciation

- Same syntax as an anonymous C♯ object without the `new` keyword
- All fields must be populated, but in any order (but can be confusing)
- Same possible styles: single/multi-lines

```fs
type Point = { X: float; Y: float }
let point1 = { X = 1.0; Y = 2.0 }
let pointKo = { Y = 2.0 }           // 💥 Error FS0764
//            ~~~~~~~~~~~ FS0764: No assignment given for field 'X' of type 'Point'
```

⚠️ **Trap:** differences declaration / instanciation
     → `:` for field type in record declaration
     → `=` for field value in record expression

## Deconstruction

- Fields are accessible by "dotting" into the object
- Alternative: deconstruction
  - Same syntax for deconstructing a *Record* as for instantiating it 👍
  - Unused fields can be ignored 💡

```fs
let { X = x1 } = point1
let { X = x2; Y = y2 } = point1
```

⚠️ Additional members *(properties)* cannot be deconstructed!

```fs
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

- A record type can be inferred from the fields used 👍 but not with members ❗
- As soon as the type is inferred, IntelliSense will work

```fs
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

Let's use an example: `inhabitantOf` is a function giving the inhabitants name *(in French)* at a given address *(in France)*

```fs
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

In F♯, typing is nominal, not structural as in TypeScript
→ Use qualification to resolve ambiguity
→ Even better: write ≠ types or put them in ≠ modules

```fs
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

## Modification

Record is immutable, but easy to get a modified copy
→ **copy and update** expression of a *Record*
→ use multi-line formatting for long expressions

```fs
// Single-line
let address2 = { address with Street = "Rue Vivienne" }

// Multi-line
let address3 =
    { address with
        City = "Lyon"
        Zip  = "69001" }
```

### Record *copy-update*: C♯ / F♯ / JS

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

### *Copy-update* limits (< F# 8)

Reduced readability with several nested levels

```fs
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

### *Copy-update* : F# 8 improvements

```fs
type Street = { Num: string; Label: string }
type Address = { Street: Street }
type Person = { Address: Address }

let person = { Address = { Street = { Num = "15"; Label = "rue Neuf" } } }

let person' =
    { person with
        Person.Address.Street.Num = person.Address.Street.Num + " bis" }
```

☝️ Usually we have to qualify the field: see `Person.`
