# Introduction

## Key points

Microsoft language family on the **.NET** platform

* Designer: **Don Syme** @ Microsoft Research
* ≃ OCaml implementation for .NET
* ≃ Inspired by Haskell _(Version 1.0 in 1990)_
* `dotnet new -lang F#`
* Interoperability between C♯ and F♯ projects/assemblies

Multi-paradigm language **Functional-first** and very concise

Whereas C♯ is imperative/objective-oriented-first and rather verbose\
&#xNAN;_(even if it takes inspiration from F♯ to be + succinct)_

## History

| Date     | C♯      | F♯     | .NET                              | Visual Studio |
| -------- | ------- | ------ | --------------------------------- | ------------- |
| 2002     | C♯ 1.0  |        | .NET Framework 1.0                | VS .NET 2002  |
| **2005** |         | F♯ 1.x | .NET Framework 1.0                | VS 2005 ?     |
| 2010     | C♯ 4.0  | F♯ 2.0 | .NET Framework 4                  | VS 2010       |
| 2015     | C♯ 6.0  | F♯ 4.0 | .NET Framework 4.6, .NET Core 1.x | VS 2015       |
| 2018     | C♯ 7.3  | F♯ 4.5 | .NET Framework 4.8, .NET Core 2.x | VS 2017       |
| 2019     | C♯ 8.0  | F♯ 4.7 | .NET Core 3.x                     | VS 2019       |
| 2020     | C♯ 9.0  | F♯ 5.0 | .NET 5.0                          | VS 2019       |
| 2021     | C♯ 10.0 | F♯ 6.0 | .NET 6.0 (LTS)                    |               |
| ...      | ...     | ...    | ...                               |               |
| 2024     | C♯ 13.0 | F♯ 9.0 | .NET 9.0                          |               |

🔗 Versions details: [https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/](https://learn.microsoft.com/en-us/dotnet/fsharp/whats-new/)

## Editors / IDE

* VsCode + [Ionide](https://marketplace.visualstudio.com/items?itemName=Ionide.Ionide-fsharp) (\*)\
  → ☝ More a boosted text editor than a full IDE\
  → ☝ Permissive: does not always report all compilation errors\
  (\*) Additional extensions: \
  [https://www.compositional-it.com/news-blog/fantastic-f-and-azure-developer-extensions-for-vscode/](introduction.md#key-points)
* Visual Studio / Rider\
  → ☝ Less refactoring capabilities than for C♯
* [https://try.fsharp.org/](introduction.md#points-cles)\
  → Online [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) with some examples

## F♯ interactive _(FSI)_

* REPL available in VS, Rider, vscode + `dotnet fsi`
* Usage : instantly test a snippet
  * 💡 In the FSI console, enter `;;` at the end of an expression to evaluate it

Example from vscode with ionide:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>FSI: Send Selection</p></figcaption></figure>

💡 In the FSI console, enter `;;` at the end of an expression to evaluate it

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

☝️ **Notes:**

* _C♯ interactive_ is more recent (VS 2015 Update 1). The FSI was there from the get go.
* Alternative worth trying, also working for C#: [LINQPad](https://www.linqpad.net/)

## File types

4 file types: `.fs`, `.fsi`, `.fsx`, `.fsproj`

> ⚠️ Single language : for F♯ only

### Standalone file

* Script file `.fsx`
  * Executable _(hence the final **x**)_ using the FSI
  * Independent but can reference other files, DLLs, NuGet packages

### Project files

* In C♯ : `.sln` contains `.csproj` projects that contains `.cs` files
* In F♯ : `.sln` contains `.fsproj` projects that contains `.fs(i)` code files

`.fsi` are signature files _(**i** for interface)_

* Associated with a `.fs` file of the same name
* Optional and rather rare in codebases
* Usages
  * Reinforces encapsulation _(like `.h` en C)_
  * Separate long documentation _(xml-doc)_
* More info : [MSDN](https://docs.microsoft.com/fr-fr/dotnet/fsharp/language-reference/signature-files)

💡 **Interop C♯ - F♯** = It's easy to have both `.csproj` and `.fsproj` projects in the same `.sln`

### F♯ Project

Création in the IDE or using the CLI `dotnet`:

* `dotnet new -l` : list supported project types
* `dotnet new console --language F# -o MyFSharpApp`
  * Création of a console project named `MyFSharpApp`
  * `--language F#` is key to specify the language, by default in C#
* `dotnet build` : to build the project
* `dotnet run` : to build the project and run the underlying executable
