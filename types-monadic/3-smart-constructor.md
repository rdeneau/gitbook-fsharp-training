# Smart constructor

## Empêcher les états invalides

🔗 [Designing with types: Making illegal states unrepresentable](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/), F♯ for fun and profit, Jan 2013

* Avoir un design qui empêche d'avoir des états invalides
  * Encapsuler état _(∑ primitives)_ dans un objet
* _Smart constructor_ sert à garantir un état initial valide
  * Valide les données en entrée
  * Si Ko, renvoie "rien" (`Option`) ou l'erreur (`Result`)
  * Si Ok, renvoie l'objet créé wrappé dans l'`Option` / le `Result`

## Encapsuler état dans un type

👉 Ajouter du sens à une primitive

👉 Faire émerger un concept, le réifier

### Mot clé `private`

* Cache contenu de l'objet
* Champs et constructeur ne sont plus visibles de l'extérieur
* Smart constructeur défini dans module compagnon 👍 ou méthode statique

### Single-case union 👌

`Type X = private X of a: 'a...`

🔗 [Designing with types: Single case union types](https://fsharpforfunandprofit.com/posts/designing-with-types-single-case-dus/) sur F♯ for fun and profit, Jan 2013

### Record 👍

`Type X = private { a: 'a... }`

🔗 [SCU: really?](https://paul.blasuc.ci/posts/really-scu.html) de Paul Blasucci, Mai 2021

## Implémentations

### Exemple 1

Smart constructeur :

* Fonction `tryCreate` dans module compagnon
* Renvoie une `Option`

```fsharp
type Latitude = private { Latitude: float } // 👈 Un seul champ, nommé comme le type

[<RequireQualifiedAccess>]                  // 👈 Optionnel
module Latitude =
    let tryCreate (latitude: float) =
        if latitude >= -90. && latitude <= 90. then
            Some { Latitude = latitude }    // 👈 Constructeur accessible ici
        else
            None

let lat_ok = Latitude.tryCreate 45.  // Some { Latitude = 45.0 }
let lat_ko = Latitude.tryCreate 115. // None
```

### Exemple 2

Smart constructeur :

* Méthode statique `Of`
* Renvoie `Result` avec erreur de type `string`

```fsharp
type Tweet =
    private { Tweet: string }

    static member Of tweet =
        if System.String.IsNullOrEmpty tweet then
            Error "Tweet shouldn't be empty"
        elif tweet.Length > 280 then
            Error "Tweet shouldn't contain more than 280 characters"
        else Ok { Tweet = tweet }

let tweet1 = Tweet.Of "Hello world" // Ok { Tweet = "Hello world" }
```
