# 📜 Recap

## Modules and namespaces

- Purpose: Group by functionality
- Scope: namespaces > files > modules

| Property                     | Namespace      | Module                    |
|------------------------------|----------------|---------------------------|
| .NET Compilation             | `namespace`    | `static class`            |
| Type                         | *Top-level*    | Local (ou *top-level*)    |
| Contains                     | Modules, Types | Val, Fun, Type, Modules   |
| `[<RequireQualifiedAccess>]` | ❌ No          | ✅ Yes *(vs shadowing)*   |
| `[<AutoOpen>]`               | ❌ No          | ✅ Yes but be careful❗  |

## 🔗 Additional ressources

[docs.microsoft.com/.../fsharp/style-guide/conventions#organizing-code](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/conventions#organizing-code)
