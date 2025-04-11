# Types valeur

## Type composite _struct_

Un type composite peut être déclaré en tant que type valeur

* Instances stockées dans la **pile** _(stack)_ plutôt que dans le tas _(heap)_
* Permet parfois de gagner en performance
* Plutôt adapté aux types compacts : peu de champs, peu de comportements

Types de déclaration :

* Attribut `[<Struct>]`
* Mot clé `struct`
* Structure

## Attribut `[<Struct>]`

Pour _Record_ et _Union_

À placer avant ou après le mot cle `type`

```fsharp
type [<Struct>] Point = { X: float; Y: float }

[<Struct>]
type SingleCase = Case of string
```

## Mot clé `struct`

Pour littéral de Tuple et _Record_ anonyme

```fsharp
let t = struct (1, "a")
// struct (int * string)

let a = struct {| Id = 1; Value = "a" |}
// struct {| Id: int; Value: string |}
```

## Structures

Alternatives aux classes 📍 [#classe](../oriente-objet/classe-structure.md#classe "mention")\
mais + limités / héritage et récursivité

👉 Cf. session sur l'orienté-objet et les classes...
