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

In F♯ the`Void` type exists! It's called `unit` because it has only one instance, written`()` and that we use like any other literal.

### Impact on the function signature

* [Rather than `void` functions, we have functions returning the `unit` type.](#user-content-fn-1)[^1]
* [Rather than functions with 0 parameter, we have functions taking a unit parameter that can only be `()`.](#user-content-fn-2)[^2]

```fsharp
let doNothing () = () // unit -> unit
let now () = System.DateTime.Now // unit -> DateTime
```

## `ignore` function

Remember : in F♯, everything is an expression.\
→ No value is ignored, except `()`/`unit` designed for this purpose\
→ At the beginning of an expression or between several `let` bindings, we can insert `unit` expressions worth ()/unit, for example `printf "mon message"`

**Issue:** we call a function which triggers a side-effect but also returns a value we are not interested in.

**Example:**\
`save` is a function that saves to the database and returns `true` or `false`

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
    let _ = save "hello" // 👌
    "ok"
```

**Solution 2 :** use the built-in `ignore` function, that has this signature :`'a -> unit`\
→ Whatever the value supplied as parameter, it ignores it and returns `()`.

```fsharp
let solution2 =
    save "hello" |> ignore // 👍
    "ok"
```

**Other examples:**

<pre class="language-fsharp"><code class="lang-fsharp">// Side-effects / file system
System.IO.Directory.CreateDirectory folder |> ignore
// Ignore the returned DirectoryInfo

// Fluent builder:
let configureAppConfigurationForEnvironment (env: Environment) (basePath: string) (builder: IConfigurationBuilder) =
    builder
        .SetBasePath(basePath)
        .AddJsonFile("appsettings.json", optional = false, reloadOnChange = true)
        .AddJsonFile($"appsettings.{env.name}.json", optional = false, reloadOnChange = true)
<strong>    |> ignore
</strong>
// Event handler:
textbox.onValueChanged(ignore)
</code></pre>

:warning: **Trap:** ignoring a value that we should use in our program.

```fsharp
[ 1..5 ]
|> Seq.map save
|> ignore // 💣

// Expected
[ 1..5 ]
|> Seq.iter (save >> ignore)
```

## Arrow notation

* 0-parameters function: `unit -> TResult`.
* 1-parameter  function: `T -> TResult`.
* 2-parameters function: `T1 -> T2 -> TResult` \* 3-parameter function: \`T1 -> T2 -> T3 -> TResult
* 3-parameters function: `T1 -> T2 -> T3 -> TResult`

### ❓ **Quiz**

> Do you know why we have `->` between the parameters? \
> What is the underlying concept?

Answer in the next page...

[^1]: This kind of functions either does nothing or produces side-effects.

[^2]: The choice of `()` to write the instance of the `unit` type is smart: it allows functions without parameters to resemble their equivalent in C#.
