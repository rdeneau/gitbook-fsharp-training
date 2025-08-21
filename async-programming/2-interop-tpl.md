---
description: 'TPL : Task Parallel Library'
---

# Interop with .NET TPL

## Interaction with .NET libraries

Asynchronous libraries in .NET and the `async`/`await` C♯ pattern:\
→ Based on **TPL** and the `Task` type

Gateways with asynchronous workflow F♯ :

* `Async.AwaitTask` and `Async.StartAsTask` functions
* `task {}` block

## Gateway functions

`Async.AwaitTask: Task<'T> -> Async<'T>`\
→ Consume an asynchronous .NET library in `async` block

`Async.StartAsTask: Async<'T> -> Task<'T>`\
→ Launch an async calculation as a `Task`

```fsharp
let getValueFromLibrary param = async {
    let! value = DotNetLibrary.GetValueAsync param |> Async.AwaitTask
    return value
}

let computationForCaller param =
    async {
        let! result = getAsyncResult param
        return result
    }
    |> Async.StartAsTask
```

## `task {}` block

> Allows consuming an asynchronous .NET library directly, using a single `Async.AwaitTask` rather than one for each async method called.

💡 Available since F♯ 6 _(before, we_ needed the [_Ply_](https://github.com/crowded/ply) _NuGet package)_

```fsharp
task {
    use client = new System.Net.Http.HttpClient()
    let! response = client.GetStringAsync("https://www.google.fr/")
    response.Substring(0, 300) |> printfn "%s"
}  // Task<unit>
|> Async.AwaitTask  // Async<unit>
|> Async.RunSynchronously
```

## `Async` _vs_ `Task`

#### 1. Calculation start mode

`Task` = _hot tasks_ → calculations started immediately❗

`Async` = _task generators_ = calculation specification, independent of startup\
→ Functional approach: no side-effects or mutations, composability\
→ Control of startup mode: when and how 👍

#### 2. Cancellation support

`Task`: by adding a `CancellationToken` parameter to async methods\
→ Forces manual testing if token is canceled = tedious + _error-prone_❗

`Async`: automatic support in calculations - token to be provided at startup 👍

## Recommendation for async functions in F♯

C♯ `async` applied at a method level\
≠ F♯ `async` defines an async block, not an async function

☝ **Recommendation:**\
» Put the entire body of the async function in an `async` block.

```fsharp
// ❌ Avoid
let workThenWait () =
    Thread.Sleep(1000)
    async { do! Async.Sleep(1000) } // Async only in this block 🧐

// ✅ Prefer
let workThenWait () = async {
    Thread.Sleep(1000)
    printfn "work"
    do! Async.Sleep(1000)
}
```

## Pitfalls of the `async`/`await` C♯ pattern

### Pitfall 1 - Really asynchronous?

In C♯: method `async` remains on the calling thread until the first `await`\
→ Misleading feeling of being asynchronous throughout the method

```csharp
async Task WorkThenWait() {
    Thread.Sleep(1000); // ⚠️ Blocks calling thread !
    await Task.Delay(1000); // Really async from here 🤔
}
```

### Pitfall 2 - Omit the `await`

```csharp
async Task PrintAfterOneSecond(string message) {
    await Task.Delay(1000);
    Console.WriteLine($"[{DateTime.Now:T}] {message}");
}

async Task Main() {
    PrintAfterOneSecond("Before"); // ⚠️ Missing `await`→ warning CS4014
    Console.WriteLine($"[{DateTime.Now:T}] After");
    await Task.CompletedTask;
}
```

Compiles but returns unexpected result: _After_ before _Before_❗

```txt
[11:45:27] After
[11:45:28] Before
```

This pitfall is also present in F♯:

```fsharp
let printAfterOneSecond message = async {
    do! Async.Sleep 1000
    printfn $"[{DateTime.Now:T}] {message}"
}

async {
    printAfterOneSecond "Before" // ⚠️ Missing `do!` → warning FS0020
    printfn $"[{DateTime.Now:T}] After"
} |> Async.RunSynchronously
```

Compiles but returns another unexpected result: no _Before_ at all ⁉️

```txt
[11:45:27] After
```

#### Compilation warnings

The previous examples compile but with big _warnings_!

C♯ [_warning CS4014_](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs4014) message:

> _Because this call is not awaited, execution of the current method continues before the call is completed._\
> &#xNAN;_&#x43;onsider applying the `await` operator..._

F♯ _warning FS0020_ message:

> _The result of this expression has type `Async<unit>` and is implicitly ignored._\
> &#xNAN;_&#x43;onsider using `ignore` to discard this value explicitly..._

☝ **Recommendation:** be sure to **always** handle this type of _warnings_! _This is even more crucial in F♯ where compilation can be tricky._

## Async.Parallel

## Thread-safety

Impure functions can be not thread-safe, for instance if they mutate a shared object like the `Console`.

It's possible to make them thread-safe using the `lock` function:

```fsharp
open System
open System.Threading

let printColoredMessage =
    let lockObject = obj ()

    fun (color: ConsoleColor) (message: string) ->
        lock lockObject (fun () ->
            Console.ForegroundColor <- color
            printfn $"%s{message} (thread ID: %i{Thread.CurrentThread.ManagedThreadId})"
            Console.ResetColor())

[ ConsoleColor.Red
  ConsoleColor.Green
  ConsoleColor.Blue ]
|> List.randomShuffle
|> List.indexed
|> List.map (fun (i, color) -> async { printColoredMessage color $"Message {i}" })
|> Async.Parallel
|> Async.RunSynchronously
```

Results in the console (example):

<div align="left"><figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure></div>

🔗 [Lock function documentation](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-core-operators.html#lock)
