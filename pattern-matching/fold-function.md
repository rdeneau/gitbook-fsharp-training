# 🚀 Fold function

Function associated with a union type and hiding the _matching_ logic\
Takes N+1 parameters for a union type with N _cases_

## **Example**

```fsharp
type [<Measure>] C
type [<Measure>] F

type Temperature =
    | Celsius     of float<C>
    | Fahrenheint of float<F>

module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius    : float<C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float<F> -> 'T

    let private [<Literal>] FactorC2F = 1.8<F/C>
    let private [<Literal>] DeltaC2F = 32.0<F>

    let celsiusToFahrenheint x = (x * FactorC2F) + DeltaC2F  // float<C> -> float<F>
    let fahrenheintToCelsius x = (x - DeltaC2F) / FactorC2F  // float<F> -> float<C>

    let toggleUnit temperature =
        temperature |> fold
            (celsiusToFahrenheint >> Fahrenheint)
            (fahrenheintToCelsius >> Celsius)

let t1 = Celsius 100.0<C>
let t2 = t1 |> Temperature.toggleUnit  // Fahrenheint 212.0
```

## Advantages

`fold` hides the implementation details of the type

For example, we could add a `Kelvin` _case_ and only impact `fold`, not the functions that call it, such as `toggleUnit` in the previous example

<pre class="language-fsharp"><code class="lang-fsharp">type [&#x3C;Measure>] C
type [&#x3C;Measure>] F
<strong>type [&#x3C;Measure>] K  // 🌟
</strong>
type Temperature =
    | Celsius     of float&#x3C;C>
    | Fahrenheint of float&#x3C;F>
<strong>    | Kelvin      of float&#x3C;K>  // 🌟
</strong>
module Temperature =
    let fold mapCelsius mapFahrenheint temperature : 'T =
        match temperature with
        | Celsius x     -> mapCelsius x      // mapCelsius: float&#x3C;C> -> 'T
        | Fahrenheint x -> mapFahrenheint x  // mapFahrenheint: float&#x3C;F> -> 'T
<strong>        | Kelvin x      -> mapCelsius (x * 1.0&#x3C;C/K> + 273.15&#x3C;C>)  // 🌟
</strong>
Kelvin 273.15&#x3C;K>
|> Temperature.toggleUnit
// Celsius 0.0&#x3C;C>
</code></pre>

🔗 ["Recursive types and folds" series > **Part 3: Introducing folds**](https://fsharpforfunandprofit.com/posts/recursive-types-and-folds-2/)**,** _fsharp for fun and profit_
