# Asynchronous workflow

## Purpose

1. Do not block the current thread while waiting for a long calculation
2. Allow parallel calculations
3. Indicate that a calculation may take some time

## `Async<'T>` type

Represents an asynchronous calculation

📆 Similar to the `async/await` pattern, way before C♯ and JS

* 2007: `Async<'T>` F♯
* 2012: `Task<T>` .NET and pattern `async`/`await`
* 2017: `Promise` JavaScript and pattern `async`/`await`

## Methods returning an `Async` object

`Async.AwaitTask(task : Task or Task<'T>) : Async<'T>`\
→ Convert a `Task` (.NET) to `Async` (F♯)

`Async.Sleep(milliseconds or TimeSpan) : Async<unit>`\
≃ `await Task.Delay()` ≠ `Thread.Sleep` → does not block current thread

FSharp.Control `CommonExtensions` module: extends the `System.IO.Stream` type ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-control-commonextensions.html))\
→ `AsyncRead(buffer: byte[], ?offset: int, ?count: int) : Async<int>`\
→ `AsyncWrite(buffer: byte[], ?offset: int, ?count: int) : Async<unit>`

FSharp.Control `WebExtensions` module: extends type `System.Net.WebClient` ([doc](https://fsharp.github.io/fsharp-core-docs/reference/fsharp-control-webextensions.html))\
→ `AsyncDownloadData(address : Uri) : Async<byte[]>`\
→ `AsyncDownloadString(address : Uri) : Async<string>`

## Run an async calculation

`Async.RunSynchronously(calc: Async<'T>, ?timeoutMs: int, ?cancellationToken) : 'T`\
→ Waits for the calculation to end, blocking the calling thread! (≠ `await` C♯) ⚠️

`Async.Start(operation: Async<unit>, ?cancellationToken) : unit`\
→ Performs the operation in the background _(without blocking the calling thread)_\
⚠️ If an exception occurs, it is "swallowed"!

`Async.StartImmediate(calc: Async<'T>, ?cancellationToken) : unit`\
→ Performs the calculation in the calling thread!

`Async.StartWithContinuations(calc, continuations..., ?cancellationToken)`\
→ Like `Async.RunSynchronously` ⚠️ ... with 3 continuation _callbacks_:\
→ on success ✅, exception 💥 and cancellation 🛑

## `async { expression }` block

_A.k.a. Async workflow_

Syntax for sequentially writing an asynchronous calculation\
→ The result of the calculation is wrapped in an `Async` object

**Keywords**

* `return` → final value of calculation • `unit` if omitted
* `let!` → access to the result of an async sub-calculation _(≃ `await` in C♯)_
* `use!` → like `use` _(management of an `IDisposable`)_ + `let!`
* `do!` → like `let!` for async calculation without return (`Async<unit>`)

```fsharp
let repeat (computeAsync: int -> Async<string>) times = async {
    for i in [ 1..times ] do
        printf $"Start operation #{i}... "
        let! result = computeAsync i
        printfn $"Result: {result}"
}

let basicOp (num: int) = async {
    do! Async.Sleep 150
    return $"{num} * ({num} - 1) = {num * (num - 1)}"
}

repeat basicOp 5 |> Async.RunSynchronously

// Start operation #1... Result: 1 * (1 - 1) = 0
// Start operation #2... Result: 2 * (2 - 1) = 2
// Start operation #3... Result: 3 * (3 - 1) = 6
// Start operation #4... Result: 4 * (4 - 1) = 12
// Start operation #5... Result: 5 * (5 - 1) = 20
```

### F♯ async function _vs_ C♯ async method

Let's compare an F♯ async function...

```fsharp
// string -> Async<int>
let getLength url = async {
    let! html = fetchAsync url
    do! Async.Sleep 1000
    return html.Length
}
```

... with its equivalent C♯ async method:

```csharp
public async Task<int> GetLength(string url) {
    var html = await FetchAsync(url);
    await Task.Delay(1000);
    return html.Length;
}
```

We can see the following equivalence regarding keywords and types:

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

| F♯                  | C♯                         |
| ------------------- | -------------------------- |
| `Async<int>` type   | `Task<int>` type           |
| `async {...}` block | `async` method keyword     |
| `let!` keyword      | `var` and `await` keywords |
| `do!` keyword       | `await` keyword            |

## Inappropriate use of `Async.RunSynchronously`

`Async.RunSynchronously` runs the calculation and returns the result BUT blocks the calling thread! Use it only at the "end of the chain" and not to _unwrap_ intermediate asynchronous calculations! Use an `async` block instead.

```fsharp
// ❌ Avoid
let a = calcA |> Async.RunSynchronously
let b = calcB a |> Async.RunSynchronously
calcC b

// ✅ Favor
async {
    let! a = calcA
    let! b = calcB a
    return calcC b
}
|> Async.RunSynchronously
```

## Parallel calculations/operations

### Async.Parallel

`Async.Parallel(computations: Async<'T> seq, ?maxDegreeOfParallelism) : Async<'T array>`

≃ `Task.WhenAll` : [Fork-Join model](https://en.wikipedia.org/wiki/Fork%E2%80%93join_model)

* _Fork_: calculations run in parallel
  * Use the optional `maxDegreeOfParallelism` to throttle/limit the number of concurrently executing tasks ; it's like `WithDegreeOfParallelism(throttle)` in LINQ and `ParallelEnumerable`.
* Wait for all calculations to finish
* _Join_: aggregation of results
  * In the same order as calculations

⚠️ All calculations must return the same type!

```fsharp
let downloadSite (site: string) = async {
    do! Async.Sleep (100 * site.Length)
    printfn $"{site} ✅"
    return site.Length
}

[ "google"; "msn"; "yahoo" ]
|> List.map downloadSite  // string list
|> Async.Parallel         // Async<string[]>
|> Async.RunSynchronously // string[]
|> printfn "%A"

// msn ✅
// yahoo ✅
// google ✅
// [|6; 3; 5|]
```

### Async.StartChild

`Async.StartChild(calc: Async<'T>, ?timeoutMs: int) : Async<Async<'T>>`

Allows several operations to be run in parallel\
→ ... whose results are of different types _(≠ `Async.Parallel`)_

Used in `async` block with 2 `let!` per child calculation _(cf. `Async<Async<'T>>`)_

**Shared cancellation** 📍\
→ Child calculation shares cancellation token with its parent calculation

**Example:**

We will test the following script in the console FSI, using the `#time` FSI directive _(_[_doc_](https://docs.microsoft.com/en-us/dotnet/fsharp/tools/fsharp-interactive/#f-interactive-directive-reference)_)_.

Let's first define a function `delay`\
→ which returns the specified value `x`\
→ after `ms` milliseconds

```fsharp
let delay (ms: int) x = async {
    do! Async.Sleep ms
    return x
}

let inSeries = async {
    let! result1 = "a" |> delay 100
    let! result2 = 123 |> delay 200
    return (result1, result2)
}

#time "on"
inSeries |> Async.RunSynchronously    // Real: 00:00:00.317, ...
#time "off"

// --> Timing now on

// Real: 00:00:00.323, CPU: 00:00:00.015, GC gen0: 0, gen1: 0, gen2: 0
// val it: string * int = ("a", 123)


// --> Timing now off

//-------------------------

let inParallel = async {
    let! child1 = "a" |> delay 100 |> Async.StartChild
    let! child2 = 123 |> delay 200 |> Async.StartChild
    let! result1 = child1
    let! result2 = child2
    return (result1, result2)
}

#time "on"
inParallel |> Async.RunSynchronously  // Real: 00:00:00.205, ...
#time "off"

// --> Timing now on

// Real: 00:00:00.218, CPU: 00:00:00.031, GC gen0: 0, gen1: 0, gen2: 0
// val it: string * int = ("a", 123)


// --> Timing now off
```

**Timing results:**

| Operation    | Real           | CPU            |
| ------------ | -------------- | -------------- |
| `inSeries`   | `00:00:00.323` | `00:00:00.015` |
| `inParallel` | `00:00:00.218` | `00:00:00.031` |

→ `inParallel` is working, longing \~200ms vs \~300ms for `inSeries`.\
→ `inParallel` uses 2 times more CPU than `inSeries`.

## Cancelling a task

Based on a default or explicit `CancellationToken/Source`:

* `Async.RunSynchronously(computation, ?timeout, ?cancellationToken)`
* `Async.Start(computation, ?cancellationToken)`

Trigger cancellation

* Explicit token + `cancellationTokenSource.Cancel()`
* Explicit token with timeout `new CancellationTokenSource(timeout)`
* Default token: `Async.CancelDefaultToken()` → `OperationCanceledException` 💣

Check cancellation

* Implicit: at each keyword in async block: `let`, `let!`, `for`...
* Explicit local: `let! ct = Async.CancellationToken` then `ct.IsCancellationRequested`.
* Explicit global: `Async.OnCancel(callback)`

**Example:**

```fsharp
let sleepLoop = async {
    let stopwatch = System.Diagnostics.Stopwatch()
    stopwatch.Start()
    let log message = printfn $"""   [{stopwatch.Elapsed.ToString("s\.fff")}] {message}"""

    use! __ = Async.OnCancel (fun () ->
        log "  Cancelled ❌")

    for i in [ 1..5 ] do
        log $"Step #{i}..."
        do! Async.Sleep 500
        log $"  Completed ✅"
}

open System.Threading

printfn "1. RunSynchronously:"
Async.RunSynchronously(sleepLoop)

printfn "2. Start with CancellationTokenSource + Sleep + Cancel"
use manualCancellationSource = new CancellationTokenSource()
Async.Start(sleepLoop, manualCancellationSource.Token)
Thread.Sleep(1200)
manualCancellationSource.Cancel()

printfn "3. Start with CancellationTokenSource with timeout"
use cancellationByTimeoutSource = new CancellationTokenSource(1200)
Async.Start(sleepLoop, cancellationByTimeoutSource.Token)
```

<details>

<summary>Outputs</summary>

```txt
1. RunSynchronously:
   [0.009] Step #1...
   [0.532]   Completed ✅
   [0.535] Step #2...
   [1.037]   Completed ✅
   [1.039] Step #3...
   [1.543]   Completed ✅
   [1.545] Step #4...
   [2.063]   Completed ✅
   [2.064] Step #5...
   [2.570]   Completed ✅
2. Start with CancellationTokenSource + Sleep + Cancel
   [0.000] Step #1...
   [0.505]   Completed ✅
   [0.505] Step #2...
   [1.011]   Completed ✅
   [1.013] Step #3...
   [1.234]   Cancelled ❌
3. Start with CancellationTokenSource with timeout
... idem 2.
```

</details>

## Summary

Adapted from 🔗 [Cancellation Tokens in F#](https://dev.to/askpt/cancellation-tokens-in-f-28gh), _by André Silva_

CT = Cancellation Token\
CTS = Cancellation Token Source

<table><thead><tr><th width="240">Keyword/Function</th><th width="100" data-type="checkbox">Shared CT</th><th width="100" data-type="checkbox">CT param</th><th width="100" data-type="checkbox">Linked CTS</th><th width="118" data-type="checkbox">Current thread</th><th width="211">Use case</th></tr></thead><tbody><tr><td><code>do!</code>, <code>let!</code>... in <code>async {}</code></td><td>true</td><td>false</td><td>false</td><td>false</td><td>Any?</td></tr><tr><td><code>Async.StartChild</code></td><td>false</td><td>false</td><td>true</td><td>false</td><td>Parallel operations</td></tr><tr><td><code>Async.Start</code></td><td>false</td><td>true</td><td>false</td><td>false</td><td>Fire &#x26; forget: send a message to a bus...</td></tr><tr><td><code>Async.StartImmediately</code></td><td>false</td><td>true</td><td>false</td><td>true</td><td>?</td></tr><tr><td><code>Async.RunSynchronously</code></td><td>false</td><td>true</td><td>false</td><td>true</td><td>Program root, scripting...</td></tr></tbody></table>
