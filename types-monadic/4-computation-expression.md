# 🚀 Computation expression (CE)

## Computation expression

Sucre syntaxique cachant une « machinerie »

* Applique la _Separation of Concerns_
* Code + lisible à l'intérieur de la _computation expression_ (CE)

Syntaxe : `builder { expr }`

* `builder` instance d'un [#builder](computation-expression-ce.md#builder "mention")📍
* `expr` peut contenir `let`, `let!`, `do!`, `yield`, `yield!`, `return`, `return!`

💡 **Note :** `seq`, `async` et `task` sont des CE

## Builder

Une _computation expression_ s'appuie sur un objet appelé _Builder_. → Cet objet permet éventuellement de stocker un état en _background_.

Pour chaque mot-clé supporté (`let!`, `return`...), le _Builder_ implémente une ou plusieurs méthodes associées. Exemples :

* `builder { return expr }` \
  → `builder.Return(expr)`
* `builder { let! x = expr; cexpr }` \
  → `builder.Bind(expr, (fun x -> {| cexpr |}))`

Le _builder_ peut également wrappé le résultat dans un type qui lui est propre :

* `async { return x }` renvoie un type `Async<'X>`
* `seq { yield x }` renvoie un type `Seq<'X>`

## Builder desugaring

Le compilateur opère la traduction vers les méthodes du _builder_.

→ La CE masque la complexité de ces appels, souvent imbriqués :

```fsharp
seq {
    for n in list do
        yield n
        yield n * 10 }

// Traduit en :
seq.For(list, fun () ->
    seq.Combine(seq.Yield(n),
                seq.Delay(fun () -> seq.Yield(n * 10)) ) )
```

## Exemples

### `logger`

Besoin : logguer les valeurs intermédiaires d'un calcul

```fsharp
let log value = printfn $"{value}"

let loggedCalc =
    let x = 42
    log x  // ❶
    let y = 43
    log y  // ❶
    let z = x + y
    log z  // ❶
    z
```

**Problèmes** ⚠️

1. Verbeux : les `log x` gênent lecture
2. _Error prone_ : oublier un `log`, logguer mauvaise valeur...

💡 Rendre les logs implicites dans une CE lors du `let!` / `Bind` :

```fsharp
type LoggingBuilder() =
    let log value = printfn $"{value}"; value
    member _.Bind(x, f) = x |> log |> f
    member _.Return(x) = x

let logger = LoggingBuilder()

//---

let loggedCalc = logger {
    let! x = 42
    let! y = 43
    let! z = x + y
    return z
}
```

### `maybe`

Besoin : simplifier enchaînement de "trySomething" renvoyant une `Option`

```fsharp
let tryDivideBy bottom top = // (bottom: int) -> (top: int) -> int option
    if (bottom = 0) or (top % bottom <> 0)
    then None
    else Some (top / bottom)

// Sans CE
let division =
    36
    |> tryDivideBy 2                // Some 18
    |> Option.bind (tryDivideBy 3)  // Some 6
    |> Option.bind (tryDivideBy 2)  // Some 3

// Avec CE
type MaybeBuilder() =
    member _.Bind(x, f) = x |> Option.bind f
    member _.Return(x) = Some x

let maybe = MaybeBuilder()

let division' = maybe {
    let! v1 = 36 |> tryDivideBy 2
    let! v2 = v1 |> tryDivideBy 3
    let! v3 = v2 |> tryDivideBy 2
    return v3
}
```

**Bilan :** ✅ Symétrie, ❌ Valeurs intermédiaires

## Limites

### Imbrication de CE

✅ On peut imbriquer des CE différentes ❌ Mais code devient difficile à comprendre

Exemple : combiner `logger` et `maybe` ❓

Solution alternative :

```fsharp
let inline (>>=) x f = x |> Option.bind f

let logM value = printfn $"{value}"; Some value  // 'a -> 'a option

let division' =
    36 |> tryDivideBy 2 >>= logM
      >>= tryDivideBy 3 >>= logM
      >>= tryDivideBy 2 >>= logM
```

### Combinaison de CE

Combiner `Async` + `Option`/`Result` ? → Solution : CE `asyncResult` + helpers dans [FsToolkit](https://demystifyfp.gitbook.io/fstoolkit-errorhandling/#a-motivating-example)

```fsharp
type LoginError =
    | InvalidUser | InvalidPassword
    | Unauthorized of AuthError | TokenErr of TokenError

let login username password =
    asyncResult {
        // tryGetUser: string -> Async<User option>
        let! user = username |> tryGetUser |> AsyncResult.requireSome InvalidUser
        // isPasswordValid: string -> User -> bool
        do! user |> isPasswordValid password |> Result.requireTrue InvalidPassword
        // authorize: User -> Async<Result<unit, AuthError>>
        do! user |> authorize |> AsyncResult.mapError Unauthorized
        // createAuthToken: User -> Result<AuthToken, TokenError>
        return! user |> createAuthToken |> Result.mapError TokenErr
    } // Async<Result<AuthToken, LoginError>>
```

##
