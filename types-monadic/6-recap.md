# 📜 Récap’



## Types `Option` et `Result`

* Types union avec 2 cases respectifs
  * `Option<'T>` : `Some<'T>` et `None`
  * `Result<'T, 'E>` : `Ok<'T>` et `Error<'E>`
* A quoi ils servent :
  * Modéliser absence de valeur et erreurs métier
  * Opérations partielles rendues totales `tryXxx`
    * _Smart constructor_ `tryCreate`
* Comment on s'en sert :
  * Chaînage : `map`, `bind`, `filter` → _ROP_
  * Pattern matching
* Leurs bénéfices :
  * `null` free, `Exception` free → pas de guard polluant code
  * Rend logique métier et _happy path_ + lisible

## _Computation expression (CE)_

* Sucre syntaxique : syntaxe intérieure standard ou "bangée" (`let!`)
* _Separation of Concerns_ : logique métier _vs_ « machinerie »
* Compilateur fait lien avec _builder_
  * Objet stockant un état
  * Build une valeur en sortie, d'un type spécifique
* Imbricables mais pas faciles à combiner !
* Concepts théoriques sous-jacents
  * Monoïde → `seq` _(d'éléments composables et avec un "zéro"_)
  * Monade → `async`, `option`, `result`
  * Applicative → `validation`/`Result<'T, 'Err list>`
* Librairies NuGet : FSharpPlus, FsToolkit, Expecto, Farmer, Saturn

## Ressources complémentaires 🔗

Compositional IT _(Isaac Abraham)_

* [Writing more succinct C# – in F#! (Part 2)](https://www.compositional-it.com/news-blog/writing-more-succinct-c-in-f-part-2/), Jul 2020

F# for Fun and Profit _(Scott Wlaschin)_

* [The Option type](https://fsharpforfunandprofit.com/posts/the-option-type/), Jun 2012
* [Making illegal states unrepresentable](https://fsharpforfunandprofit.com/posts/designing-with-types-making-illegal-states-unrepresentable/), Jan 2013
* [Série de 11 articles sur les CE](https://fsharpforfunandprofit.com/series/computation-expressions/), Jan 2013
* [Série de 7 articles sur monades 'n co](https://fsharpforfunandprofit.com/series/map-and-bind-and-apply-oh-my/), Aug 2015
