# Object expression

## Introduction

Expression used to implement an abstract type on the fly

```fsharp
let makeResource (resourceName: string) =
    printfn $"create {resourceName}"
    { new System.IDisposable with
        member _.Dispose() =
            printfn $"dispose {resourceName}" }
```

☝ Notice the signature of `makeResource`: `string -> System.IDisposable`. \
→ The backing class is hidden/transparent at the F♯ level.\
→ Advantage: _Upcasting_ is not required, compared to interface implementation in a type.

## Interface singleton

We can implement a _Singleton_ using an object expression:

```fsharp
[<Interface>]
type IConsole =
    abstract ReadLine : unit -> string
    abstract WriteLine : string -> unit

let console =
    { new IConsole with
        member this.ReadLine () = Console.ReadLine ()
        member this.WriteLine line = printfn "%s" line }
```

## Implementing several interfaces

Implementing several interfaces in an object expression is possible, but only the first interface is visible. To use the members of the other interfaces, we need to perform a downcast, which is unsafe by nature❗

```fsharp
let makeDelimiter (delim1: string, delim2: string, value: string) =
    { new System.IFormattable with
        member _.ToString(format: string, _: System.IFormatProvider) =
            if format = "D" then
                delim1 + value + delim2
            else
                value
      interface System.IComparable with
        member _.CompareTo(_) = -1 }

let o = makeDelimiter("<", ">", "abc")
// val o : System.IFormattable
let s = o.ToString("D", System.Globalization.CultureInfo.CurrentCulture)
// val s : string = "<abc>"
let i = (d :?> System.IComparable).CompareTo("cde")  // ❗ Unsafe
// val i : int = -1
```

👉 Prefer to split the object in 2.
