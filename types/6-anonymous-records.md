# Anonymous records

## Introduction

* Since F♯ 4.6 _(March 2019)_
* Syntax: same as _Record_ with "fat" braces `{| fields |}`
  * `{| Age: int |}` → signature
  * `{| Age = 15 |}` → instance
* Inline typing: no need to pre-define a named `type`
  * Alternative to _Tuples_
* Allowed in function input/output
  * ≠ Anonymous type C♯

## Benefits ✅

* Reduce _boilerplate_
* Improve interop with external systems (JavaScript, SQL...)
* Compatible with C♯ anonymous types: \
  → When a C♯ API requires an anonymous type, we can give it an anonymous record instance.

Examples _(more on this later)_ :

* LINQ projection
* Customization of an existing record
* JSON serialization
* Inline signature
* Alias by module

### LINQ Projection

💡 Select a subset of properties

```fsharp
let names =
    query {
        for p in persons do
        select {| Name = p.FirstName |}
    }
```

In C♯, we would use an anonymous type:

```cs
var names =
    from p in persons
    select new { Name = p.FirstName };
```

🔗 [F# vs C#: Anonymous Records](https://queil.net/2019/10/fsharp-vs-csharp-anonymous-records/) by Krzysztof Kraszewski

### Customize an existing record

💡 An anonymous record can be instantiated from a record instance

```fsharp
type Person = { Age: int; Name: string }
let william = { Age = 12; Name = "William" }

// Add a field (Gender)
let william' = {| william with Gender = "Male" |}
            // {| Age = 12; Name = "William"; Gender = "Male" |}

// Modify fields (Name, Age: int => float)
let jack = {| william' with Name = "Jack"; Age = 16.5 |}
        // {| Age = 16.5; Name = "Jack"; Gender = "Male" |}
```

### JSON serialization

💡 An anonymous record can be used as an intermediary type to serialize a union in JSON.

**Example:**

```fsharp
#r "nuget: Newtonsoft.Json"
let serialize obj = Newtonsoft.Json.JsonConvert.SerializeObject obj

type CustomerId = CustomerId of int
type Customer = { Id: CustomerId; Age: int; Name: string; Title: string option }

serialize { Id = CustomerId 1; Age = 23; Name = "Abc"; Title = Some "Mr" }
```

Resulting JSON:

```json
{
  "Id": { "Case": "CustomerId", "Fields": [ 1 ] }, // 👀
  "Age": 23,
  "Name": "Abc",
  "Title": { "Case": "Some", "Fields": [ "Mr" ] }  // 👀
}
```

→ Format verbose and not practical.

💡 **Solution:** Define an anonymous record as "DTO" to serialize a _customer_.

```fsharp
let serialisable customer =
    let (CustomerId customerId) = customer.Id
    {| customer with
         Id = customerId
         Title = customer.Title |> Option.toObj |}

serialize (serialisable { Id = CustomerId 1; Age = 23; Name = "Abc"; Title = Some "Mr" })
```

Resulting JSON:

```json
{
  "Id": 1, // ✅
  "Age": 23,
  "Name": "Abc",
  "Title": "Mr"  // ✅
}
```

### Signature _inline_

💡 Use an anonymous record declared inside a bigger type to reduce cognitive load:

```fsharp
type Title = Mr | Mrs
type Customer =
    { Age  : int
      Name : {| First: string; Middle: string option; Last: string |} // 👈
      Title: Title option }
```

## Limits 🛑

```fsharp
// No inference from field usage
let nameKo x = x.Name  // 💥 Error FS0072: Lookup on object of indeterminate type...
let nameOk (x: {| Name:string |}) = x.Name

// No deconstruction
let x = {| Age = 42 |}
let {  Age = age  } = x  // 💥 Error FS0039: The record label 'Age' is not defined
let {| Age = age |} = x  // 💥 Error FS0010: Unexpected symbol '{|' in let binding

// No full objects merge
let banana = {| Fruit = "Banana" |}
let yellow = {| Color = "Yellow" |}
let banYelKo = {| banana with yellow |} // 💥 Error FS0609...
let banYelOk = {| banana with Color = "Yellow" |}

// No omissions
let ko = {| banYelOk without Color |}  // 💥 No 'without' keyword
```
