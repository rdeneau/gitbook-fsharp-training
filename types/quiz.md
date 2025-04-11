# 🍔 Quiz

Relier types et concepts :

```fsharp
type Color1 = int * int * int
type Color2 = Red | Green | Blue
type Color3 = Red=1 | Green=2 | Blue=3
type Color4 = { Red: int; Green: int; Blue: int }
type Color5 = {| Red: int; Green: int; Blue: int |}
type Color6 = Color of Red: int * Green: int * Blue: int
type Color7 =
    | RGB of { Red: int; Green: int; Blue: int }
    | HSL of { Hue: int; Saturation: int; Lightness: int }

// A. Alias
// B. Enum
// C. Record
// D. Record anonyme
// E. Single-case union
// F. Union
// G. Union enum-like
// H. Tuple
```

> 💡 Plusieurs choix possibles

### Réponses

| Types                                       | Concepts                                |
| ------------------------------------------- | --------------------------------------- |
| `type Color1 = int * int * int`             | **H.** Tuple + **A.** Alias             |
| `type Color2 = Red ∣ Green ∣ Blue`          | **G.** Union enum-like                  |
| `type Color3 = Red=1 ∣ Green=2 ∣ Blue=3`    | **B.** Enum                             |
| `type Color4 = { Red: int; Green: int… }`   | **C.** Record                           |
| `type Color5 = {∣ Red: int; Green: int… ∣}` | **D.** Record anonyme + **A.** Alias    |
| `type Color6 = Color of Red: int * …`       | **E.** Single-case union + **H.** Tuple |
| `type Color7 = RGB of { Red: int… } ∣ HSL…` | **F.** Union + **C.** Record            |

## Composition de types

### Création de nouveaux types ?&#x20;

* ❌ Les types algébriques ne supportent pas l'héritage de classe.
  * Ils ne peuvent pas hériter d'une classe de base.
  * Ils sont `sealed` : on ne peut pas en hériter.
  * Mais ils peuvent implémenter des interfaces.
* ✅ Les types algébriques sont créés par composition, sous forme de :&#x20;
  * Agrégation dans un _product type_ (_Record_, _Tuple_)
  * "Choix" dans un _sum type_ (_Union_)

### ❓ Combiner 2 _Unions_ ?

Considérons les 2 types _Union_ ci-dessous :&#x20;

```fsharp
type Noir = Pique | Trefle
type Rouge = Coeur | Carreau
```

Comment les combiner pour avoir un objet qui soit au choix `Pique`, `Trefle`, `Coeur` ou `Carreau` ?

* Par aplatissement un peu comme en TypeScript ?

```fsharp
type CouleurKo = Noir | Rouge
```

❌ `Noir` (comprendre `CouleurKo.Noir`) est un case de l'union `CouleurKo` totalement décorrélé du type `Noir`. F# permet de les nommer pareil mais c'est malencontreux ici.

* Par composition ?

```fsharp
type Couleur = Noir of Noir | Rouge of Rouge
let pique = Noir Pique
```

✅ Un peu verbeux mais cela marche

* Par création d'une toute nouvelle Union ?

```fsharp
type Couleur = Pique | Trefle | Coeur | Carreau
let pique = Pique
```

:heavy\_check\_mark:Intéressant pour changer de modélisation

:warning: Attention au conflit de nom sur les _Cases_ des _Unions_ : \
→ `Pique` = `Couleur.Pique` ou `Noir.Pique` ?

☝ On peut passer par des fonctions pour convertir explicitement chaque cas entre eux :&#x20;

```fsharp
type Noir = Pique | Trefle
type Rouge = Coeur | Carreau

type Couleur =
    | Pique | Trefle | Coeur | Carreau

    static member ofNoir noir =
        match noir with
        | Noir.Pique -> Couleur.Pique
        | Noir.Trefle -> Couleur.Trefle

    static member ofRouge rouge =
        match rouge with
        | Rouge.Coeur -> Couleur.Coeur
        | Rouge.Carreau -> Couleur.Carreau

let coeur = Couleur.ofRouge Rouge.Coeur
```

### 💡 Extension d'un _Record_

Il existe une autre manière de créer un type à partir d'un autre type en procédant par extension. \
→ Il est possible de créer à la volée un _Record_ _anonyme_ qui étend un _Record_ existant, en partant d'une instance de ce _Record_ et en lui ajoutant des champs :&#x20;

```fsharp
type Person = { Name: string; City: string }
let joe = { Name = "Joe"; City = "Paris" }
let joeWithZip = {| joe with Zip = 75001 |}
(*
val joeWithZip: {| City: string; Name: string; Zip: int |} = { City = "Paris"
                                                               Name = "Joe"
                                                               Zip = 75001 }
*)
```

❌ Par contre, on ne peut pas faire l'inverse, à savoir enlever de champs pour obtenir un type "réduit". Ce n'est pas encore supporté par F#.

## Conclusion

Beaucoup de façons de modéliser !

De quoi s'adapter :

* À tous les goûts ?
* En fait surtout au domaine métier !
