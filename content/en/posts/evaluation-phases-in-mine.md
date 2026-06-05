---
title: "Evaluation Phases in Mine"
date: 2026-05-22
draft: false
description: "Evaluation Phases in Mine Language"
---

Mine has two phases: **runtime** and **compile time**. By default, everything runs at runtime. If you want something evaluated at compile time, you say so with `comptime`.

That's the whole idea. But there's a detail I find really elegant (and it's about numbers).

---

## Runtime by default

Simple rule: unless you write `comptime`, it happens at runtime.

```mine
let x = heavyFn()           // runs at runtime (default)
comptime let x = heavyFn()  // runs once at compile time
```

This applies to bindings, functions, and expressions:

```mine
fn compute(n: i32) i32 {..}          // runtime function
comptime fn compute(n: i32) i32 {..} // compile-time function

comptime if(cond) { // branch resolved at compile time
    // only this branch gets compiled
}
```

## `comptime` parameters

A `comptime` parameter tells the compiler: "this value must be known at compile time at the call site."

```mine
fn allocate(comptime size: i32) {..}

comptime let n = 64
allocate(n) // OK (n is comptime-known)

let m = readInput()
allocate(m) // CompilerError (m is not comptime-known)
```

## But... numbers are already comptime :P

Here's the part I love. Plain integer and float literals in Mine have no concrete type (they're `comptime_int` and `comptime_float` until you pin them):

```mine
let x = 42          // comptime_int (no size, totally free)
let x = 42 as i32   // now it's i32
let x = 42 as u256  // or u256 (your call)
let x: i64 = 42     // or i64 via annotation
```

Same for floats:

```mine
let y = 3.14        // comptime_float (equivalent to f128, full precision)
let y = 3.14 as f32 // pinned to f32
```

And characters follow the same rule:

```mine
let c = 'A'         // comptime_int (ASCII fits in u8)
let c = 'A' as u8   // u8

let h = '❤️'        // comptime_int (Unicode, needs u21)
let h = '❤️' as u21 // u21
```

No default integer size forced on you. No silent truncation. You decide when you're ready.

I borrowed this idea from Zig, and I think it's one of the cleanest approaches to numeric types I've seen. When you write `42`, you're writing a *mathematical value*, not a machine word. The machine word comes later, when it matters.

## In classes

`comptime` works on fields and methods too:

```mine
class Config {
    comptime let MAX_RETRIES = 3 as i32  // compile-time constant
    comptime fn defaultName() []u8 {..}  // evaluated at compile time
}
```

Comptime fields must be immutable (a mutable compile-time field doesn't really make sense). Compile-time means "decided once, forever."

## Why I care about this

Performance-critical code often has two kinds of values: things you know at compile time, and things you only know at runtime. Most languages blur the line (you `const` things and hope the optimizer figures it out).

Mine makes the line explicit. `comptime` is a *contract*: this value exists at compile time, the compiler guarantees it, and you can depend on it. Runtime is everything else.

That's the kind of explicitness I want everywhere in Mine :)

---

→ [Mutability in Mine](./mutability-in-mine)

→ [Visibility in Mine](./visibility-in-mine)

→ [How Types Work in Mine](./how-types-work-in-mine)