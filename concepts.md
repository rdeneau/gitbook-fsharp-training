# Concepts

## Currying

### Definition

Consiste à transformer :

* une fonction prenant N paramètres\
  `Func<T1, T2, ...Tn, TReturn>` en C♯
* en une chaîne de N fonctions prenant 1 paramètre\
  `Func<T1, Func<Tn, ...Func<Tn, TReturn>>>`

### Application partielle

Appel d'une fonction avec moins d'arguments que son nombre de paramètres

* Possible grâce à la curryfication
* Renvoie fonction prenant en paramètre le reste d'arguments à fournir

```fsharp
// Template function with 2 parameters
let insideTag (tagName: string) (content: string) =
    $"<{tagName}>{content}</{tagName}>"

// Helpers with a single parameter `content`
// `tagName` is fixed by partial application
let emphasize = insideTag "em"     // `tagName` fixed to "em"
let strong    = insideTag "strong" // `tagName` fixed to "strong"

// Equivalent - élégant mais + explicite
let em content = insideTag "em" content
```

:warning: **Attention :** l'application partielle impacte la signature :

```fsharp
val insideTag: tagName: string -> content: string -> string
val emphasize: (string -> string)  // 👈 (1)(2)
val em: content: string -> string
```

1. Perte du nom des paramètres restant dans la signature (`content`)
2. Signature d'une valeur de type fonction d'où les parenthèses (`(string -> string)`)

### Syntaxe des fonctions F♯

Paramètres séparés par des espaces\
→ Indique que les fonctions sont curryfiées\
→ D'où les `->` dans la signature entre les paramètres

```fsharp
let fn () = result         // unit -> TResult
let fn arg = ()            // T    -> unit
let fn arg = result        // T    -> TResult

let fn x y = (x, y)        // T1 -> T2 -> (T1 * T2)

// Equivalents, explicitement curryfiés :
let fn x = fun y -> (x, y) // 1. Avec une lambda
let fn x =                 // 2. Avec une sous-fonction nommée
    let fn' y = (x, y)     // N.B. `x` vient du scope
    fn'
```

### IntelliSense Ionide

Dans VsCode avec Ionide, l'IntelliSense fournit un descriptif plus lisible des fonctions, en mettant chaque argument dans une nouvelle ligne :

```fsharp
let pair x y = (x, y)
// val pair:
//    x: 'a ->
//    y: 'b
//    -> 'a * 'b

let triplet x y z = (x, y, z)
// val triplet:
//    x: 'a ->
//    y: 'b ->
//    z: 'c
//    -> 'a * 'b * 'c
```

### Compilation .NET d’une fonction curryfiée

☝ Une fonction curryfiée est compilée différemment selon comment elle est appelée !

* De base, elle est compilée en méthode avec paramètres tuplifiés\
  → Vue comme méthode normale quand consommée en C♯

Exemple : F♯ puis équivalent C♯ _(version simplifiée de_ [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjoA3esAx4iuAEy5EAPgZNcARnJA===)_)_ :

```fsharp
module A =
    let add x y = x + y
    let value = 2 |> add 1
```

```csharp
public static class A
{
    public static int add(int x, int y) => x + y;
    public static int value => 3;
}
```

* Lorsqu'elle est appliquée partiellement, elle est compilée sous forme de classe pseudo `Delegate` étendant `FSharpFunc<int, int>` avec une méthode `Invoke` qui encapsule les 1ers arguments fournis

Exemple : F♯ puis équivalent C♯ _(version simplifiée de_ [_SharpLab_](https://sharplab.io/#v2:DYLgZgzgNAJiDUAfAtgexgV2AUwAQEFcBeAWAChdLccAXXAQxhlwA9cBPY13eD8q6tjqMYAeQB2eIgya4AjEA===)_)_ :

```fsharp
module A =
    let add x y = x + y
    let addOne = add 1
```

```csharp
public static class A
{
    internal sealed class addOne@3 : FSharpFunc<int, int>
    {
        internal static readonly addOne@3 @_instance = new addOne@3();

        internal addOne@3() { }

        public override int Invoke(int y) => 1 + y;
    }

    public static FSharpFunc<int, int> addOne => addOne@3.@_instance;

    public static int add(int x, int y) => x + y;
}
```

## Conception unifiée des fonctions

Le type `unit` et la curryfication permettent de concevoir les fonctions simplement comme :

* **Prend un seul paramètre** de type quelconque
  * y compris `unit` pour une fonction "sans paramètre"
  * y compris une autre fonction _(callback)_
* **Renvoie une seule valeur** de type quelconque
  * y compris `unit` pour une fonction "ne renvoyant rien"
  * y compris une autre fonction

👉 **Signature universelle** d'une fonction en F♯ : `'T -> 'U`

## Ordre des paramètres

Entre C♯ et F♯, on ne place pas au même endroit le paramètre concernant l'objet principal (le `this` dans le cas d'une méthode) :

* Dans méthode extension C♯, l'objet `this` est le 1er paramètre
  * Ex : `items.Select(x => x)`
* En F♯, l'objet principal est _plutôt_ le **dernier paramètre** :\
  \&#xNAN;_(ce qui s'appelle le style data-last)_
  * Ex : `List.map (fun x -> x) items`

Style _data-last_ favorise :

* Pipeline : `items |> List.map square |> List.sum`
* Application partielle : `let sortDesc = List.sortBy (fun i -> -i)`
* Composition de fonctions appliquées partiellement jusqu'au param "_data_"
  * `(List.map square) >> List.sum`

:warning: Friction avec BCL .NET car plutôt _data-first_

☝ Solution : wrapper dans une fonction avec params dans ordre sympa en F♯

```fsharp
let startsWith (prefix: string) (value: string) =
    value.StartsWith(prefix)
```

💡 **Tips** : utiliser `Option.defaultValue` plutôt que `defaultArg` avec les options

* Fonctions font la même chose mais params `option` et `value` sont inversés
* `defaultArg option value` : param `option` en 1er 😕
* `Option.defaultValue value option` : param `option` en dernier 👍

De même, préférer mettre **en 1er** les paramètres les + statiques = Ceux susceptibles d'être prédéfinis par application partielle

Ex : "dépendances" qui seraient injectées dans un objet en C♯

👉 Application partielle = moyen de simuler l'injection de dépendances
