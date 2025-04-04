# Signature

## From void to unit

### Issues with `void` in C♯

`void` needs to be handle separately = 2 times more work 😠

* 2 types of delegates: `Action` vs `Func<T>`
* 2 types od task: `Task` vs `Task<T>`

### Exemple : `ITelemetry`

```csharp
interface ITelemetry
{
  void Run(Action action);
  T Run<T>(Func<T> func);

  Task RunAsync(Func<Task> asyncAction);
  Task<T> RunAsync<T>(Func<Task<T>> asyncFunc);
}
```

🔗 Mads Torgersen is the C#'s Lead Designer. In [his interview by Nick Chapsas](https://youtu.be/T9UqIkuGnuo?si=9qFyQ0J1EsUjCkNP\&t=5071), at 1:21:25, he indicates the 3 features he would completely remove from C#: `event`, `delegate` and `void`!

### Type Void

☝ The issue with the`void` is that it's neither a type nor a value.

💡 What about a `Void` type, a singleton with no data in it:

```csharp
public class Void
{
    public static readonly Void Instance = new Void();

    private Void() {}
}
```

Let's play with it...

### `ITelemetry` simplification

First, let's define the following helpers to convert to `Void` :

```csharp
public static class VoidExtensions
{
    // Action -> Func<Void>
    public static Func<Void> AsFunc(this Action action)
    {
        action();
        return Void.Instance;
    }

    // Func<Task> -> Func<Task<Void>>
    public async static Func<Task<Void>> AsAsyncFunc(this Func<Task> asyncAction)
    {
        await asyncAction();
        return Void.Instance;
    }
}
```

Then, we can write a default implementation (C♯ 8) for 2 of 4 methods:

```csharp
interface ITelemetry
{
    void Run(Action action) =>
        Run(action.AsFunc());

    T Run<T>(Func<T> func);

    Task RunAsync(Func<Task> asyncAction) =>
        RunAsync(asyncAction.AsAsyncFunc());

    Task<T> RunAsync<T>(Func<Task<T>> asyncFunc);
}
```

### Type `unit`

In F♯  the`Void` type exists! It's called `unit` because it has only one instance, written`()` and that we use like any other literal.

### Impact on the function signature

* Rather than `void` functions, we have functions returning the `unit` type. (1)
* Rather than functions with 0 parameter, we have functions taking a unit parameter that can only be `()`. (2)

```fsharp
let doNothing () = () // unit -> unit
```

**Notes:**

* (1) This kind of functions either does nothing or produces side-effects.
* (2) The choice of `()` to write the instance of the `unit` type is smart: it allows functions without parameters to resemble their equivalent in C#.

## `ignore` function&#x20;

In F♯, "everything is an expression" \
→ No value is ignored, except `()`/`unit`  \
→ At the beginning of an expression or between several `let` bindings, we can insert `unit` expressions worth ()/unit, for example `printf "mon message"`

**Issue:** we call a function which triggers a side-effect but also returns a value we are not interested in.

**Example:** \
&#x20;`save` is a function that saves to the database and returns `true` or `false`&#x20;

<pre class="language-fsharp"><code class="lang-fsharp">let save entity = true // Fake implementation

let problem =
<strong>    save "hello"
</strong>//  ~~~~~~~~~~~~ ⚠️
// Warning FS0020: The result of this expression has type 'bool' and is implicitly ignored.
// Consider using 'ignore' to discard this value explicitly, e.g. 'expr |> ignore',
// or 'let' to bind the result to a name, e.g. 'let result = expr'.

    "ok" // Random final value just for demo purpose
</code></pre>

**Solution 1 :** discard the returned value

```fsharp
let solution1 =
    let _ = save "bonjour" // 👌
    "ok"
```

**Solution 2 :** use the built-in `ignore` function, that has this signature :`'a -> unit`\
→ Whatever the value supplied as parameter, it ignores it and returns `()`.

```fsharp
let solution2 =
    save "bonjour" |> ignore // 👍
    "ok"
```

**Other examples:**

```fsharp
// Side-effects / file system
System.IO.Directory.CreateDirectory folder |> ignore
// Ignore the returned DirectoryInfo

// Fluent builder:
let configureAppConfigurationForEnvironment (env: Environment) (basePath: string) (builder: IConfigurationBuilder) =
    builder
        .SetBasePath(basePath)
        .AddJsonFile("appsettings.json", optional = false, reloadOnChange = true)
        .AddJsonFile($"appsettings.{env.name}.json", optional = false, reloadOnChange = true)
    |> ignore

// Event handler:
textbox.onValueChanged(ignore)
```

:warning: **Trap:** ignoring a value that we should use in our program.

```fsharp
[ 1..5 ]
|> Seq.map save
|> ignore // 💣

// Expected
[ 1..5 ]
|> Seq.iter (save >> ignore)
```

## Notation fléchée

* Fonction à 0 paramètre : `unit -> TResult`
* Fonction à 1 paramètre : `T -> TResult`
* Fonction à 2 paramètres : `T1 -> T2 -> TResult`
* Fonction à 3 paramètres : `T1 -> T2 -> T3 -> TResult`

❓ **Quiz** : Pourquoi des `->` entre les paramètres ? Quel est le concept sous-jacent ?

## Curryfication

> ☝ _Currying_ en anglais

### Définition

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
// Template à 2 paramètres
let insideTag (tagName: string) (content: string) =
    $"<{tagName}>{content}</{tagName}>"

// Helpers à 1! paramètre `content`, `tagName` étant fixé par application partielle
let emphasize = insideTag "em"     // `tagName` fixé à la valeur "em"
let strong    = insideTag "strong" // `tagName` fixé à la valeur "strong"

// Equivalent - élégant mais + explicite
let em content = insideTag "em" content
```

:warning: **Attention :** l'application partielle impacte la signature :&#x20;

<pre><code><strong>val insideTag: tagName: string -> content: string -> string
</strong>val emphasize: (string -> string)  👈 1 👈 2
val em: content: string -> string
</code></pre>

1. Perte du nom des paramètres restant dans la signature (`content`)
2. Signature d'une valeur de type fonction d'où les parenthèses (`(string -> string)`)

### Syntaxe des fonctions F♯

Paramètres séparés par des espaces \
→ Indique que les fonctions sont curryfiées \
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

Dans VsCode avec Ionide, l'IntelliSense fournit un descriptif plus lisible des fonctions, en mettant chaque argument dans une nouvelle ligne :&#x20;

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

* De base, elle est compilée en méthode avec paramètres tuplifiés \
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
* En F♯, l'objet principal est _plutôt_ le **dernier paramètre** : \
  &#xNAN;_(ce qui s'appelle le style data-last)_
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
