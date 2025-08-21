# 📜 Recap

## Modules and namespaces

* Purpose: Group by functionality
* Scope: namespaces > files > modules

| Property                     | Namespace      | Module                  |
| ---------------------------- | -------------- | ----------------------- |
| .NET Compilation             | `namespace`    | `static class`          |
| Type                         | _Top-level_    | Local (or _top-level_)  |
| Contains                     | Modules, Types | Val, Fun, Type, Modules |
| `[<RequireQualifiedAccess>]` | ❌ No           | ✅ Yes _(vs shadowing)_  |
| `[<AutoOpen>]`               | ❌ No           | ✅ Yes but be careful❗   |

## 🔗 Additional resources

[docs.microsoft.com/.../fsharp/style-guide/conventions#organizing-code](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#organizing-code)
