---
title: "Mutability in Mine"
date: 2026-05-20
draft: false
description: "Mutability in Mine Language"
---

When I started designing Mine, one of the first decisions I made was: **everything is immutable by default.**

Not because it's trendy. Because it's honest.

---

When you read a binding in code, you shouldn't have to wonder "will this change later?" (you should *know*). Immutability by default means the answer is always "no" unless someone explicitly said otherwise.

## The basics

In Mine, you declare bindings with `let` (immutable) or `mut` (mutable):

```mine
let x = 42 // immutable (cannot be reassigned)
mut y = 42 // mutable (can be reassigned)

x = 10     // CompilerError (`x` is immutable)
y = 10     // OK
```

Simple. If you didn't write `mut`, it won't change. The compiler enforces it, not convention.

---

## Pointers follow the same rule

Pointers are also immutable by default. A `*T` pointer lets you read through it (but not write). If you want to mutate the value it points to, you need `*mut T`:

```mine
mut val = 42 as i32

let p: *i32     = &val  // immutable pointer (read only)
let p: *mut i32 = &val  // mutable pointer (can write)

p.* = 10                // CompilerError (*i32 is immutable)

let q: *mut i32 = &val
q.* = 10                // OK
```

And you can't borrow an immutable binding as `*mut` (that would be lying to the compiler):

```mine
let val = 42 as i32
let p: *mut i32 = &val  // CompilerError (cannot borrow immutable as *mut)
```

---

## Functions and parameters

Parameters are immutable by default too. If a function wants to modify a local copy, it uses `mut`. If it wants to modify the *caller's* value, it takes a `*mut` pointer:

```mine
// mutates a local copy (caller's value unchanged)
fn double(mut n: i32) {
    n = n * 2       // OK (local copy)
}

// mutates the caller's value
fn increment(n: *mut i32) {
    n.* = n.* + 1   // OK
}
```

---

## In classes

Fields follow the same pattern (`let` for immutable, `mut` for mutable). Immutable fields can only be set in the constructor:

```mine
class Counter {
    let id: i32         // immutable (set once in constructor)
    pub mut count: i32  // mutable (can change anytime)

    Counter(pub id: i32) {
        this.count = 0 as i32
    }

    pub fn increment() {
        this.count = this.count + 1  // OK
        this.id    = 99 as i32       // CompilerError (`id` is immutable)
    }
}
```

---

## Why I think this matters

Most languages make mutability the default and ask you to opt into immutability (`const`, `final`, `readonly`...). Mine flips that.

It's a small change in syntax, but a big change in how you read code. When you see `let`, you *know* it won't change. When you see `mut`, you *know* it might. No guessing, no conventions, no hoping the previous developer was careful.

That's the kind of explicitness I wanted Mine to have from day one :)

---

→ [Visibility in Mine](./visibility-in-mine)

→ [Evaluation Phases in Mine](./evaluation-phases-in-mine)

→ [Types in Mine](./types-in-mine)