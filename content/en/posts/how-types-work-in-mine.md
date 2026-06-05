---
title: "How Types Work in Mine"
date: 2026-05-23
draft: false
description: "How Types Work in Mine Language"
---

Mine's type system has one goal: **never lie to you.**

No hidden sizes, no silent coercions, no "well it's *technically* a string." Here's how it works.

---

## Numbers are free until you pin them

This is the part I'm most proud of. Plain number literals in Mine have no concrete type (they're `comptime_int` or `comptime_float`, free-precision compile-time values):

```mine
let x = 42   // comptime_int (no size, no limit)
let y = 3.14 // comptime_float (full f128 precision)
```

You pin them when you're ready:

```mine
let x = 42 as i32   // i32
let x = 42 as u256  // or u256 (any width)
let x: i64 = 42     // or via annotation

let y = 3.14 as f32 // f32
```

Characters follow the same rule:

```mine
let c = 'A'  // u8 (ASCII)
let h = '❤️' // u21 (Unicode codepoint)
```

No default integer size forced on you. No silent truncation. The compiler holds the value until you decide what it should be.

---

## Integer types (arbitrary width)

Mine supports `iN` and `uN` for any N (not just the "standard" sizes):

```mine
let a: u8   = 255
let b: i32  = -1000
let c: i1   = 0 // 1-bit integer
let d: u256 = 0 // 256-bit unsigned
```

Common sizes like `i8`, `i16`, `i32`, `i64`, `u8`, `u16`, `u32`, `u64` all work (but so does anything in between).

Floats are `f16`, `f32`, `f64`, and `f128`. Straightforward.

---

## Other primitives

| Type    | Meaning                                              |
| ------- | ---------------------------------------------------- |
| `bool`  | `true` or `false`                                    |
| `void`  | no value (for functions that return nothing)         |
| `never` | function never returns (infinite loop, always fails) |
| `any`   | any type (opts out of type checking)                 |
| `type`  | a type itself (used in comptime contexts)            |

```mine
fn crash() never { loop {..} } // never returns
fn log() void { @printn("x") } // returns nothing
fn id(x: any) any { return x } // any type
comptime let T: type = i32     // T holds a type
```

---

## Optionals and undefined

`?T` is Mine's optional type (a value that might not exist):

```mine
let x: ?i32 = null      // no value
let x: ?i32 = 42        // has value
let y = x ?? 0 as i32   // unwrap or fallback
```

`undefined` is uninitialized memory (unsafe, but explicit):

```mine
let x: i32  = undefined // uninitialized (reading before assignment is UB)
let x: ?i32 = null      // safe absence
```

The distinction matters. `null` is a value you *chose*. `undefined` is memory you haven't touched yet. Mine makes you pick which one you mean.

---

## def (shape contracts and type aliases)

`def` serves two purposes. As a **type alias**:

```mine
def UserId = i32
def Matrix = [][]f32

let id: UserId = 1 // same as i32 underneath
```

As a **shape contract** (a named set of field types):

```mine
def Point { x: f32, y: f32 }

let p: Point = { x: 1.0, y: 2.0 }         // OK
let p: Point = { x: 1.0, y: 2.0, z: 3.0 } // CompilerError (extra field)
```

Only anonymous objects satisfy a `def`. Classes have their own identity (a `Vec2` is never a `Point` even if it has the same fields).

---

## Strings and slices

In Mine, a string is just a `[]u8` (a slice of bytes. No special `string` type, no hidden magic).

```mine
let s: []u8 = "hello"   // immutable slice (default)
s.len                   // 5
s.ptr                   // *u8 (pointer to first element)
s[0]                    // 'h' as u8
```

Slices are immutable by default, same as everything else. Use `[]mut u8` for a mutable buffer:

```mine
let buf: []mut u8 = [0, 0, 0, 0]
buf[0] = 'h'
buf.ptr                 // *mut u8 (inherits mutability from the slice)
buf.ptr.* = 'H'         // OK
```

The `.ptr` type follows the slice's mutability (you never get a writable pointer from an immutable slice):

| Slice      | `.ptr` type          |
| ---------- | -------------------- |
| `[]u8`     | `*u8` (read only)    |
| `[]mut u8` | `*mut u8` (writable) |

**Subslicing** uses the range syntax you already know:

```mine
let s: []u8 = "hello"
let sub = s[1..3]       // "el" ([]u8)
let sub = s[1..=3]      // "ell" ((inclusive))
```

**Fixed-size arrays** vs slices:

```mine
let arr: [5]u8 = [1, 2, 3, 4, 5]   // fixed, size known at compile time
let s: []u8 = arr[1..3]            // slice of the array (runtime length)
```

**Typed slices** work for any type, not just `u8`:

```mine
let nums: []i32  = [1, 2, 3]
let flags: []bool = [true, false, true]
let matrix: [][]f32 = [[1.0, 2.0], [3.0, 4.0]]
```

Compared to Zig: no `[]const u8` vs `[]u8` confusion (Mine's immutable by default means `[]u8` is already the safe, read-only case). No `[*]u8` many-item pointer weirdness (`.ptr` gives you exactly what the slice's mutability allows).

---

## Enums, unions, and tagged unions (all via `def`)

Mine uses `def` for all three. No separate `enum`, `union`, or `union(enum)` keywords like Zig.

**Enum** (a set of named variants, no data):

```mine
def Direction = .North | .South | .East | .West

// enums can be merged
def Cardinals  = .North     | .South     | .East      | .West
def Diagonals  = .NorthEast | .NorthWest | .SouthEast | .SouthWest
def AllDirs    = Cardinals  | Diagonals

let d: AllDirs = .NorthEast

match d {
    .North => { "up"   }
    .South => { "down" }
    else   => { "side" }
}
```

**Union** (a value that can be one of several types):

```mine
def Value = i32 | bool | []u8

let v: Value = 42 as i32

match v {
    i32(n)  => { n * 2 }
    bool(b) => { if b { 1 } else { 0 } }
    else    => { 0 }
}
```

**Tagged union** (variants with their own data):

```mine
def Shape =
    | .Circle { radius: f32 }
    | .Rect   { width: f32, height: f32 }

let s: Shape = .Circle { radius: 5.0 }

match s {
    .Circle(c) => { c.radius * c.radius } // capture the data
    .Rect(r)   => { r.width * r.height }
}
```

> `match` is exhaustive (if you don't cover all variants, it's a compiler error). Use `else` as a fallback.

```mine
match s {
    .Circle(c) => { c.radius }
    else       => { 0.0 } // covers .Rect and anything added later
}
```

Compared to Zig: no `union(enum)` awkwardness, no `struct` for variant data, cleaner capture syntax (`(binding)` vs `|binding|`), and everything lives under one keyword (`def`).

---

## Compound types

```mine
// Array (fixed size, same type)
let arr: []u8 = [1, 2, 3]
arr[0]  // 1
arr.len // 3

// Tuple (fixed size, mixed types)
let t: .{i32, bool} = .{42, true}
t[0]    // 42
t.len   // 2

// Anonymous object (named fields)
let p = { x: 1.0, y: 2.0 }
```

---

## Why it works this way

Every type in Mine means exactly what it says. When something doesn't have a concrete type yet (like a `comptime_int` the compiler tells you that too, instead of silently picking one for you).

Explicit types feel like work at first. Then you stop hitting weird bugs at 2am and it starts feeling like a gift :P

---

→ [Mutability in Mine](./mutability-in-mine)

→ [Visibility in Mine](./visibility-in-mine)

→ [Evaluation Phases in Mine](./evaluation-phases-in-mine)

→ [Error Handling in Mine](./error-handling-in-mine)

→ [Control Flow in Mine](./control-flow-in-mine)