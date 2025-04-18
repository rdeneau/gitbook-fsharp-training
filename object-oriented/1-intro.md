---
description: Orienté-objet, polymorphisme
---

# Introduction

In F♯, object-oriented is sometimes more practical than functional style.

Object-oriented bricks in F♯:

1. Members
   - Methods, properties, operators
   - Attach functionalities directly to the type
   - Encapsulate the object's state (particularly if mutable)
   - Used with object dotting `my-object.my-member`
2. Interfaces and classes
   - Support abstraction through inheritance

## The four pillars of object-oriented programming

- Abstraction
- Encapsulation
- Inheritance
- Polymorphism

With the exception of inheritance, the other 3 concepts also apply to functional programming, especially in F♯, but to a lesser extent.

### Abstraction

In OO, especially in C♯, abstraction is implemented using:

- Types: classes or enums
- Abstract classes and interfaces

In FP, abstraction is implemented using:

- Types: algebraic data types are more powerful to model domain concepts
- Functions: although equivalent to an interface with a single method, a function has a lower level of abstraction than an interface

### Encapsulation

Encapsulation means 2 things:

- Hide elements:
  - This feature is easy in F♯: `private` keyword, expression nesting, partial application, `.fsi` files...
- Prevent a state from being invalid = protect application invariants
  - If the state is mutable, it's definitively object-oriented.
  - If the state is immutable, it should be encapsulated only when it's possible to create an instance or copy and update an instance up to an invalid state.

### Inheritance

Inheritance is an object-oriented technique. Still, F♯ supports it, with just some limitations compared to C♯.

## Polymorphism

In fact, there are several polymorphisms. The main ones are:

1. By sub-typing: the one classically evoked with object-orientation
   → Basic type defining abstract or virtual members
   → Subtypes inheriting and implementing these members
2. Ad hoc/overloading → overloading of members with the same name
3. Parametric → generic in C♯, Java, TypeScript
4. Structural/duck-typing → SRTP in F♯, structural typing in TypeScript
5. Higher-kinded → type classes in Haskell

F♯ supports the 4 first polymorphism.
