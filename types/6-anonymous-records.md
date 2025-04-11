# Records anonymes

## Introduction

* Depuis F♯ 4.6 _(mars 2019)_
* Syntaxe : idem _Record_ avec accolades "larges" `{| fields |}`
  * `{| Age: int |}` → signature
  * `{| Age = 15 |}` → instance
* Typage _inline_ : pas besoin de pré-définir un `type` nommé
  * Alternative aux _Tuples_
* Autorisé en entrée/sortie de fonction
  * ≠ Type anonyme C♯

## Bénéfices ✅

Réduire _boilerplate_ • Améliorer interop avec systèmes externes (JavaScript, SQL...)

Exemples :

* Projection LINQ
* Personnalisation d'un record existant
* Sérialisation JSON
* Signature _inline_
* Alias par module

### ✅ Projection LINQ

💡 Sélectionner un sous-ensemble de propriétés

```fsharp
let names =
    query {
        for p in persons do
        select {| Name = p.FirstName |}
    }
```

En C♯, on utiliserait un type anonyme :

```csharp
var names =
    from p in persons
    select new { Name = p.FirstName };
```

_🔗_ [https://queil.net/2019/10/fsharp-vs-csharp-anonymous-records/](https://queil.net/2019/10/fsharp-vs-csharp-anonymous-records/)

### ✅ Personnalisation d'un record existant

💡 Un record anonyme peut être instancié à partir d'une instance de record

```fsharp
type Person = { Age: int; Name: string }
let william = { Age = 12; Name = "William" }

// Ajout d'un champ (Gender)
let william' = {| william with Gender = "Male" |}
            // {| Age = 12; Name = "William"; Gender = "Male" |}

// Modification de champs (Name, Age: int => float)
let jack = {| william' with Name = "Jack"; Age = 16.5 |}
        // {| Age = 16.5; Name = "Jack"; Gender = "Male" |}
```

### ✅ Sérialisation JSON

😕 Unions sérialisées dans un format pas pratique

```fsharp
#r "nuget: Newtonsoft.Json"
let serialize obj = Newtonsoft.Json.JsonConvert.SerializeObject obj

type CustomerId = CustomerId of int
type Customer = { Id: CustomerId; Age: int; Name: string; Title: string option }

serialize { Id = CustomerId 1; Age = 23; Name = "Abc"; Title = Some "Mr" }
```

```json
{
  "Id": { "Case": "CustomerId", "Fields": [ 1 ] }, // 👀
  "Age": 23,
  "Name": "Abc",
  "Title": { "Case": "Some", "Fields": [ "Mr" ] }  // 👀
}
```

💡 Définir un record anonyme pour sérialiser un _customer_

```fsharp
let serialisable customer =
    let (CustomerId customerId) = customer.Id
    {| customer with
         Id = customerId
         Title = customer.Title |> Option.toObj |}

serialize (serialisable { Id = CustomerId 1; Age = 23; Name = "Abc"; Title = Some "Mr" })
```

```json
{
  "Id": 1, // ✅
  "Age": 23,
  "Name": "Abc",
  "Title": "Mr"  // ✅
}
```

### ✅ Signature _inline_

💡 Utiliser un record anonyme _inline_ pour réduire charge cognitive

```fsharp
type Title = Mr | Mrs
type Customer =
    { Age  : int
      Name : {| First: string; Middle: string option; Last: string |} // 👈
      Title: Title option }
```

### ✅ Alias par module

```fsharp
module Api =
    type Customer = // ☝ Customer est un alias
        {| Id   : System.Guid
           Name : string
           Age  : int |}

module Dto =
    type Customer =
        {| Id   : System.Guid
           Name : string
           Age  : int |}

let (customerApi: Api.Customer) = {| Id = Guid.Empty; Name = "Name"; Age = 34 |}
let (customerDto: Dto.Customer) = customerApi // 🎉 Pas besoin de convertir
```

💡 Instant t : même type dans 2 modules&#x20;

💡 Plus tard : facilite personnalisation des types par module

## Limites 🛑

```fsharp
// Inférence limitée
let nameKo x = x.Name  // 💥 Error FS0072: Lookup on object of indeterminate type...
let nameOk (x: {| Name:string |}) = x.Name

// Pas de déconstruction
let x = {| Age = 42 |}
let {  Age = age  } = x  // 💥 Error FS0039: The record label 'Age' is not defined
let {| Age = age |} = x  // 💥 Error FS0010: Unexpected symbol '{|' in let binding

// Pas de fusion
let banana = {| Fruit = "Banana" |}
let yellow = {| Color = "Yellow" |}
let banYelKo = {| banana with yellow |} // 💥 Error FS0609...
let banYelOk = {| banana with Color = "Yellow" |}

// Pas d'omission
let ko = {| banYelOk without Color |}  // 💥 Mot clé 'without' n'existe pas

// Pas du typage structurel => tous les champs sont requis
let capitaliseFruit (x: {| Fruit: string |}) = x.Fruit.ToUpper()
capitaliseFruit {| Fruit = "Banana" |}                      // 👌 "BANANA"
capitaliseFruit {| Fruit = "Banana"; Origin = "Réunion" |}  // 💥 Too much fields... [Origin]
```
