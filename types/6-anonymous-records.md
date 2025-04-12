# Records anonymes

## Introduction

- Since F♯ 4.6 *(March 2019)*
- Syntax: same as *Record* with "fat" braces `{| fields |}`
  - `{| Age: int |}` → signature
  - `{| Age = 15 |}` → instance
- Inline typing: no need to pre-define a named `type`
  - Alternative to *Tuples*
- Allowed in function input/output
  - ≠ Anonymous type C♯

## Benefits ✅

- Reduce *boilerplate*
- Improve interop with external systems (JavaScript, SQL...)

Examples *(more on this later)* :

- LINQ projection
- Customization of an existing record
- JSON serialization
- Inline signature
- Alias by module

### ✅ LINQ Projection

💡 Select a subset of properties

```fs
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

### ✅ Customize an existing record

💡 An anonymous record can be instantiated from a record instance

```fs
type Person = { Age: int; Name: string }
let william = { Age = 12; Name = "William" }

// Add a field (Gender)
let william' = {| william with Gender = "Male" |}
            // {| Age = 12; Name = "William"; Gender = "Male" |}

// Modify fields (Name, Age: int => float)
let jack = {| william' with Name = "Jack"; Age = 16.5 |}
        // {| Age = 16.5; Name = "Jack"; Gender = "Male" |}
```

### ✅ JSON serialization

💡 An anonymous record can be used as an intermediary type to serialize a union in JSON.

**Example:**

```fs
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

💡 **Solution:** Define an anonymous record as "DTO" to serialize a *customer*.

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

### ✅ Signature *inline*

💡 Use an anonymous record declared inside a bigger type to reduce cognitive load:

```fs
type Title = Mr | Mrs
type Customer =
    { Age  : int
      Name : {| First: string; Middle: string option; Last: string |} // 👈
      Title: Title option }
```

## Limits 🛑

```fs
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
