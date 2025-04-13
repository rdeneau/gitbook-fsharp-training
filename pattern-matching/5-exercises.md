# 🕹️ Exercises

The following exercises on [exercism - F# track](https://exercism.org/tracks/fsharp) can be solved with active patterns:

## Collatz Conjecture *(easy)*

- [Instructions](https://exercism.org/tracks/fsharp/exercises/collatz-conjecture)
- [Wikipedia](https://en.wikipedia.org/wiki/Collatz_conjecture)

<details>

<summary>My solution</summary>

```fsharp
let private (|NegativeOrZero|One|PositiveEven|PositiveOdd|) number =
    if number <= 0 then NegativeOrZero
    elif number = 1 then One
    elif number % 2 = 0 then PositiveEven
    else PositiveOdd
let steps (number: int): int option =
    let rec loop n count =
        match n with
        | NegativeOrZero -> None
        | One            -> Some count
        | PositiveEven   -> loop (n / 2) (count + 1)
        | PositiveOdd    -> loop (3 * n + 1) (count + 1)
    loop number 0
```

</details>

## 🎯 Darts *(easy)*

- [Instructions](https://exercism.org/tracks/fsharp/exercises/darts)
- [Wikipedia](https://en.wikipedia.org/wiki/Darts)

<details>

<summary>My solution</summary>

```fsharp
let private radiusOfCircle = {| Outer = 10.0 ; Middle = 5.0 ; Inner = 1.0 |}

let private distanceToOrigin point =
    point
    |> List.sumBy (fun x -> x ** 2.0)
    |> sqrt

let (|Inner|Middle|Outer|Outside|) point =
    match distanceToOrigin point with
    | x when x > radiusOfCircle.Outer  -> Outside
    | x when x > radiusOfCircle.Middle -> Outer
    | x when x > radiusOfCircle.Inner  -> Middle
    | _                                -> Inner

let score (x: double) (y: double): int =
    match [ x; y ] with
    | Outside -> 0
    | Outer   -> 1
    | Middle  -> 5
    | Inner   -> 10
```

</details>

## 👑 Queen Attack *(medium)*

- [Instructions](https://exercism.org/tracks/fsharp/exercises/queen-attack)
- [Wikipedia](https://en.wikipedia.org/wiki/Queen_(chess))

<details>

<summary>My solution</summary>

```fsharp
type Position = int * int

let (|SameRow|SameColumn|SameDiagonal|Other|) (queen1: Position, queen2: Position) =
  match queen1, queen2 with
  | (x1, _), (x2, _) when x1 = x2 -> SameRow
  | (_, y1), (_, y2) when y1 = y2 -> SameColumn
  | (x1, y1), (x2, y2) when abs (y1 - y2) = abs (x1 - x2) -> SameDiagonal
  | _ -> Other

let canAttack (queen1: Position) (queen2: Position) =
  match queen1, queen2 with
  | SameRow | SameColumn | SameDiagonal -> true
  | Other -> false
```

</details>

## 🤖 Robot Name *(medium)*

[Instructions](https://exercism.org/tracks/fsharp/exercises/robot-name)

When a robot comes off the factory floor, it has no name.

The first time you turn on a robot, a random name is generated in the format of two uppercase letters followed by three digits, such as RX837 or BC811.

Every once in a while we need to reset a robot to its factory settings, which means that its name gets wiped. The next time you ask, that robot will respond with a new random name.

The names must be random: they should not follow a predictable sequence. Using random names means a risk of collisions. Your solution must ensure that every existing robot has a unique name.

<details>

<summary>My solution</summary>

```fsharp
open System

type Robot = Robot of string

let private random = Random(Guid.NewGuid().GetHashCode());

let private letters = [| 'A'..'Z' |]

let private letter () = letters[random.Next(0, 25)]
let private digit () = random.Next(0, 9)

let private randomRobotName () =
    $"{letter()}{letter()}{digit()}{digit()}{digit()}"

let mutable private names = Set.empty

let private (|Available|Taken|) name =
    if names |> Set.contains name
    then Taken
    else Available name

let rec private uniqueRobotName () =
    match randomRobotName () with
    | Available name -> name
    | Taken -> uniqueRobotName()

let private makeRobotWith adjustNames =
    let name = uniqueRobotName()
    names <-
        names
        |> adjustNames
        |> Set.add name
    Robot name

let makeRobot () = makeRobotWith id

let reset (Robot previousName) = makeRobotWith (Set.remove previousName)
```

</details>
