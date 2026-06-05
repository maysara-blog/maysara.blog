---
title: "Visibility in Mine"
date: 2026-05-26
draft: false
description: "Visibility in Mine Language"
---

Here's something I believe strongly: **code should be private until proven otherwise.**

When you write something, it belongs to its scope. If you want the outside world to see it (say so explicitly). That's the rule in Mine, and it applies everywhere.

---

## The basics

Everything in Mine is private by default. Use `pub` to expose it:

```mine
let secret = 42      // private (default)
pub let visible = 42 // public

fn helper() {..}     // private
pub fn api() {..}    // public
```

No `prv` keyword needed (silence means private). I like that. It means the default state of your code is *contained*, not *exposed*.

---

## `prv` (explicit private)

There's also `prv`, which does the same thing as the default but makes it *explicit*. I added it mainly for constructor shorthands, where you want to be clear about which parameters become private fields:

```mine
class User {
    User(pub username: []u8, prv password: []u8)
    //   ^^^                  ^^^
    //   public field          private field (explicit)
}

let u = User("maysara", "hunter2")
u.username  // OK
u.password  // CompilerError (private)
```

Without `pub` or `prv`, the parameter is just a local value (not promoted to a field at all).

---

## Static members

`static` members belong to the class itself, not to any instance. You call them directly on the class:

```mine
class Config {
    pub static let VERSION = "0.1.0"
    pub static fn default() Config {..}
}

Config.VERSION      // OK (no instance needed)
Config.default()    // OK (no instance needed)
```

One rule I'm firm about: **static fields must be immutable**. A mutable static field is shared global state, and that's exactly the kind of hidden mutation I designed Mine to avoid:

```mine
class Counter {
    pub static mut count = 0 as i32  // CompilerError (static fields must be immutable)
}
```

If you need shared mutable state, be explicit about it (don't hide it behind a static field).

---

## Why private by default?

Because the surface area of your code is a decision, not an accident.

When everything is public by default, you end up with APIs that were never meant to be APIs. Someone uses an internal helper, now you can't change it. Someone reads a field directly, now it's part of your contract.

Private by default means your public API is always *intentional*. Every `pub` is a decision you made consciously. I think that leads to better code, and honestly (it leads to less regret later :P)

---

→ [Mutability in Mine](./mutability-in-mine)

→ [Evaluation Phases in Mine](./evaluation-phases-in-mine)

→ [How Types Work in Mine](./how-types-work-in-mine)

→ [Error Handling in Mine](./error-handling-in-mine)

---

<br>