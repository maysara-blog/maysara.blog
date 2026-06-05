---
title: "Generics in Mine"
date: 2026-05-28
draft: false
description: "Generics in Mine Language"
---

I'll be honest: I could have made Mine's generic system more powerful. Trait bounds, higher-kinded types, associated types... I know what those are, and I chose not to add them.

Here's what I built instead, and why.

---

## The basics (`<T>` on classes and defs)

Add `<T>` after a class or `def` name:

```mine
class Box<T> {
    Box(pub value: T)
}

let b = Box(42 as i32)    // T = i32
let b = Box(true)         // T = bool
let b: Box<i32> = Box(42) // explicit
```

The compiler infers `T` from what you pass in. Multiple type parameters work the same way:

```mine
class Pair<A, B> {
    Pair(pub first: A, pub second: B)
}

let p = Pair(42 as i32, true)           // A = i32, B = bool
let p: Pair<i32, bool> = Pair(42, true) // explicit
```

`def` works the same way:

```mine
def Wrapper<T> { value: T }

let w: Wrapper<i32>  = { value: 42 }
let w: Wrapper<bool> = { value: true }
```

---

## Constraints (`T: SomeDef`)

When you need `T` to have certain fields, constrain it:

```mine
def Named { name: []u8 }

class Registry<T: Named> {
    Registry(pub item: T)

    pub fn label() []u8 {
        return this.item.name // OK, T is guaranteed to have `name`
    }
}

let r = Registry({ name: "Alice" })  // OK
let r = Registry({ age: 30 as i32 }) // CompilerError (missing `name: []u8`)
```

Multiple constraints with `&`:

```mine
def Aged { age: i32 }

class Profile<T: Named & Aged> {
    Profile(pub data: T)
}

let p = Profile({ name: "Alice", age: 30 as i32 }) // OK
let p = Profile({ name: "Alice" })                 // CompilerError (missing `age`)
```

---

## Generic functions

Generic functions use the same `<T>` syntax:

```mine
fn identity<T>(x: T) T {
    return x
}

let a = identity(42 as i32)   // T = i32
let b = identity(true)        // T = bool
```

Multiple type parameters:

```mine
fn swap<A, B>(a: A, b: B) .{B, A} {
    return .{b, a}
}

let t = swap(42 as i32, true)              // returns .{bool, i32}
let t: .{bool, i32} = swap(42, true)       // explicit
```

Generic functions with constraints:

```mine
def Comparable { value: i32 }

fn compare<T: Comparable>(a: T, b: T) bool {
    return a.value < b.value
}

compare({ value: 10 }, { value: 20 }) // OK
compare({ id: 10 }, { id: 20 })       // CompilerError (missing `value`)
```

---

## Monomorphization (zero runtime cost)

When you write `Box<i32>` and `Box<bool>`, Mine compiles two separate versions. No boxing, no runtime type erasure, no overhead. Same approach as Zig and C++.

Generic functions get the same treatment (each concrete type parameter generates its own compiled version).

---

## Why I kept it simple

I've seen generic systems so expressive they become their own language. You spend more time satisfying the type checker than writing actual programs.

Mine's generics cover what you need day to day: parameterizing containers and functions, constraining types to have certain fields, composing constraints. That's 95% of real use cases. The remaining 5% can usually be solved with a different design.

Simple isn't the same as weak. It means the tool does what you need without getting in the way :)

---

→ [Types in Mine](./types-in-mine)

→ [Classes in Mine](./classes-in-mine)
