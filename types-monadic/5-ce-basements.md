---
description: CE = Computation Expression
---

# 🚀 CE - Fondements théoriques

## CE : le couteau suisse ✨

Les _computation expressions_ servent à différentes choses :

* C♯ `yield return` → F♯ `seq {}`
* C♯ `async/await` → F♯ `async {}`
* C♯ expressions LINQ `from... select` → F♯ `query {}`
* ...

Fondements théoriques sous-jacents :

* Monoïde
* Monade
* Applicative

## Monoïde

≃ Type `T` définissant un ensemble comportant :

1. Opération `(+) : T -> T -> T`
   * Pour combiner des ensembles et garder le même "type"
   * Associative : `a + (b + c)` ≡ `(a + b) + c`
2. Élément neutre _(aka identity)_ ≃ ensemble vide
   * Combinable à tout ensemble sans effet
   * `a + e` ≡ `e + a` ≡ `a`

### CE monoïdale

Le builder d'une CE monoïdale _(telle que `seq`)_ dispose _a minima_ de :

* `Yield` pour construire l'ensemble élément par élément
* `Combine` ≡ `(+)` (`Seq.append`)
* `Zero` ≡ élément neutre (`Seq.empty`)

S'y ajoute généralement (entre autres) :

* `For` pour supporter `for x in xs do ...`
* `YieldFrom` pour supporter `yield!`

## Monade

≃ Type générique `M<'T>` comportant :

1. Fonction `return` de construction
   * Signature : `(value: 'T) -> M<'T>`
   * ≃ Wrap une valeur
2. Fonction `bind` de "liaison" _(aka opérateur `>>=`)_
   * Signature : `(f: 'T -> M<'U>) -> M<'T> -> M<'U>`
   * Utilise la valeur wrappée, la "map" avec la fonction `f`        vers une valeur d'un autre type et "re-wrap" le résultat

### Lois

`return` ≡ élément neutre pour `bind`

* À gauche : `return x |> bind f` ≡ `f x`
* À droite : `m |> bind return` ≡ `m`

`bind` est associatif

* `m |> bind f |> bind g` ≡ `m |> bind (fun x -> f x |> bind g)`

### Langages

**Haskell**

* Monades beaucoup utilisées. Les + communes : `IO`, `Maybe`, `State`, `Reader`.
* `Monad` est une _classe de type_ pour créer facilement ses propres monades.

**F♯**

* Certaines CE permettent des opérations monadiques.
* Plus rarement utilisées directement _(sauf par des Haskellers)_

**C♯**

* Monade implicite dans LINQ
* Librairie [LanguageExt](https://github.com/louthy/language-ext) de programmation fonctionnelle

### CE monadique

Le builder d'une CE monadique dispose des méthodes `Return` et `Bind`.

Les types `Option` et `Result` sont monadiques. \
→ On peut leur créer leur propre CE :

```fsharp
type OptionBuilder() =
    member _.Bind(x, f) = x |> Option.bind f
    member _.Return(x) = Some x

type ResultBuilder() =
    member _.Bind(x, f) = x |> Result.bind f
    member _.Return(x) = Ok x
```

### CE monadique et générique

[FSharpPlus](http://fsprojects.github.io/FSharpPlus/computation-expressions.html) propose une CE `monad` \
→ Marche pour tous les types monadiques : `Option`, `Result`, ... et même `Lazy` !

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let lazyValue = monad {
    let! a = lazy (printfn "I'm lazy"; 2)
    let! b = lazy (printfn "I'm lazy too"; 10)
    return a + b
} // System.Lazy<int>

let result = lazyValue.Value
// I'm lazy
// I'm lazy too
// val result : int = 12
```

### Exemple avec le type `Option`

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let addOptions x' y' = monad {
    let! x = x'
    let! y = y'
    return x + y
}

let v1 = addOptions (Some 1) (Some 2) // Some 3
let v2 = addOptions (Some 1) None     // None
```

:warning: **Limite :** on ne peut pas mélanger plusieurs types monadiques !

```fsharp
#r "nuget: FSharpPlus"
open FSharpPlus

let v1 = monad {
    let! a = Ok 2
    let! b = Some 10
    return a + b
} // 💥 Error FS0043...

let v2 = monad {
    let! a = Ok 2
    let! b = Some 10 |> Option.toResult
    return a + b
} // val v2 : Result<int,unit> = Ok 12
```

### CE monadiques spécifiques

Librairie [FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling/) propose : \
• CE `option {}` spécifique au type `Option<'T>` _(exemple ci-dessous)_ \
• CE `result {}` spécifique au type `Result<'Ok, 'Err>`

☝ Recommandé car + explicite que CE `monad`

```fsharp
#r "nuget: FSToolkit.ErrorHandling"
open FsToolkit.ErrorHandling

let addOptions x' y' = option {
    let! x = x'
    let! y = y'
    return x + y
}

let v1 = addOptions (Some 1) (Some 2) // Some 3
let v2 = addOptions (Some 1) None     // None
```

## Applicative

_A.k.a Applicative Functor_

≃ Type générique `M<'T>`

3 "styles" :

**Style A :** Applicatives avec `apply`/`<*>` et `pure`/`return` \
• ❌ Pas facile à comprendre \
• 💡 Présenté par Jérémie Chassaing dans le talk ❝[Applicatives in real life](https://vimeopro.com/newcrafts/newcrafts/video/338449781)❞\
• ☝ Déconseillé par Don Syme dans cette [note de nov. 2020](https://github.com/dsyme/fsharp-presentations/blob/master/design-notes/rethinking-applicatives.md)

**Style B :** Applicatives avec `mapN` \
• `map2`, `map3`... `map5` combine 2 à 5 valeurs wrappées

**Style C :** Applicatives avec `let! ... and! ...` dans une CE \
• Même principe : combiner plusieurs valeurs wrappées \
• Disponible à partir de F♯ 5 _(_[_annonce de nov. 2020_](https://devblogs.microsoft.com/dotnet/announcing-f-5/#applicative-computation-expressions)_)_

☝ **Conseil :** Styles B et C sont autant recommandés l'un que l'autre.

### CE applicative

Librairie [FsToolkit.ErrorHandling](https://github.com/demystifyfp/FsToolkit.ErrorHandling/) propose : \
• Type `Validation<'Ok, 'Err>` ≡ `Result<'Ok, 'Err list>` \
• CE `validation {}` supportant syntaxe `let!...and!...`

Permet d'accumuler les erreurs \
→ Usages : \
• Parsing d'inputs externes \
• _Smart constructor_ _(Exemple de code slide suivante...)_

```fsharp
#r "nuget: FSToolkit.ErrorHandling"
open FsToolkit.ErrorHandling

type [<Measure>] cm
type Customer = { Name: string; Height: int<cm> }

let validateHeight height =
    if height <= 0<cm>
    then Error "Height must me positive"
    else Ok height

let validateName name =
    if System.String.IsNullOrWhiteSpace name
    then Error "Name can't be empty"
    else Ok name

module Customer =
    let tryCreate name height : Result<Customer, string list> =
        validation {
            let! validName = validateName name
            and! validHeight = validateHeight height
            return { Name = validName; Height = validHeight }
        }

let c1 = Customer.tryCreate "Bob" 180<cm>  // Ok { Name = "Bob"; Height = 180 }
let c2 = Customer.tryCreate "Bob" 0<cm> // Error ["Height must me positive"]
let c3 = Customer.tryCreate "" 0<cm>    // Error ["Name can't be empty"; "Height must me positive"]
```

## Applicative _vs_ Monade

> Soit N opérations `tryXxx` renvoyant un `Option` ou `Result`

**Style monadique :**

* Avec `bind` ou CE `let! ... let! ...`
* **Chaîne** les opérations, exécutée 1 à 1, la N dépendant de la N-1
* S'arrête à 1ère opération KO → juste 1ère erreur dans `Result` ①
* [_Railway-oriented programming_](https://fsharpforfunandprofit.com/rop/) de Scott Wlaschin

```fsharp
module Result =
    // f : 'T -> Result<'U, 'Err>
    // x': Result<'T, 'Err>
    //  -> Result<'U, 'Err>
    let bind f x' =
        match x' with
        | Error e  -> Error e // 👈 (1)
        | Ok value -> f value
```

**Style applicatif :**

* Avec `mapN` ou CE `let! ... and! ...`
* **Combine** 2..N opérations indépendantes → parallélisables 👍
* Permet de combiner les cas `Error` contenant une `List` ②

```fsharp
module Validation =
    // f : 'T -> 'U -> Result<'V, 'Err list>
    // x': Result<'T, 'Err list>
    // y': Result<'U, 'Err list>
    //  -> Result<'V, 'Err list>
    let map2 f x' y' =
        match x', y' with
        | Ok x, Ok y -> f x y
        | Ok _, Error errors | Error errors, Ok _ -> Error errors
        | Error errors1, Error errors2 -> Error (errors1 @ errors2) // 👈 ②
```

## Autres CE

On a vu 2 librairies qui étendent F♯ et proposent leurs CE :

* FSharpPlus → `monad`
* FsToolkit.ErrorHandling → `option`, `result`, `validation`

Beaucoup de librairies ont leur propre DSL _(Domain Specific Language.)_ Certaines s'appuient alors sur des CE :

* Expecto
* Farmer
* Saturn

### Expecto

❝ Librairie de testing : assertions + runner ❞ 🔗 https://github.com/haf/expecto

```fsharp
open Expecto

let tests =
  test "A simple test" {
    let subject = "Hello World"
    Expect.equal subject "Hello World" "The strings should equal"
  }

[<EntryPoint>]
let main args =
  runTestsWithCLIArgs [] args tests
```

### Farmer

❝ _Infrastructure-as-code_ pour Azure ❞

🔗 [github.com/compositionalit/farmer](https://github.com/compositionalit/farmer)

```fsharp
// Create a storage account with a container
let myStorageAccount = storageAccount {
    name "myTestStorage"
    add_public_container "myContainer"
}

// Create a web app with application insights that's connected to the storage account
let myWebApp = webApp {
    name "myTestWebApp"
    setting "storageKey" myStorageAccount.Key
}

// Create an ARM template (Azure Resource Manager)
let deployment = arm {
    location Location.NorthEurope
    add_resources [
        myStorageAccount
        myWebApp
    ]
}

// Deploy it to Azure!
deployment
|> Writer.quickDeploy "myResourceGroup" Deploy.NoParameters
```

### Saturn

❝ Framework Web au-dessus de ASP.NET Core, pattern MVC ❞

🔗 [saturnframework.org](https://saturnframework.org/)

```fsharp
open Saturn
open Giraffe

let app = application {
    use_router (text "Hello World from Saturn")
}

run app
```

## Aller + loin

📹 [Extending F# through Computation Expressions](https://youtu.be/bYor0oBgvws) - 📜 [Slides](https://panesofglass.github.io/computation-expressions/#/)

🔗 [Computation Expressions Workshop](https://github.com/panesofglass/computation-expressions-workshop)
