---
title: "Exit Statements in Mine"
date: 2026-05-26
draft: false
description: "Exit Statements in Mine Language"
---

Exit statements end the current execution path. They are not expressions and produce no value. Each one says clearly where execution goes next.

---

## return

Exits the current function and optionally returns a value to the caller:

```mine
fn add(a: i32, b: i32) i32 {
    return a + b
}

fn log(msg: []u8) void {
    @printn(msg)
    return // explicit, or just let the function end
}
```

---

## fail

Exits the current function by returning an error. Used in functions with an error-returning signature:

```mine
fn divide(a: f32, b: f32) MathErrors!f32 {
    if b == 0.0 { fail DivisionByZero }
    return a / b
}
```

`fail` is Mine's equivalent of `return error.X` in Zig, just cleaner. You're saying what you mean: this function failed, here's why.

---

## break

Exits the current loop:

```mine
for i in 0..10 {
    if i == 5 { break }
    @printn(i) // 0, 1, 2, 3, 4
}
```

With labeled loops, `break` can exit an outer loop directly:

```mine
outer: for i in 0..10 {
    for j in 0..10 {
        if j == 5 { break outer } // exits the outer loop
    }
}
```

With labeled blocks, `break` exits the block and optionally returns a value:

```mine
let x = result: {
    if cond { break result 42 as i32 }
    break result 0 as i32
}
```

Labels use `name:` syntax, same as in Rust and Kotlin.

---

## continue

Skips the rest of the current iteration and moves to the next one. If the loop has a continue expression (`: expr`), it runs before the next iteration:

```mine
for i in 0..10 {
    if i == 5 { continue }
    @printn(i) // 0, 1, 2, 3, 4, 6, 7, 8, 9
}

mut i = 0 as i32
while i < 10 : (i += 1) {
    if i == 5 { continue } // i += 1 still runs
    @printn(i)
}
```

With labeled loops, `continue` skips to the next iteration of the target loop:

```mine
outer: for i in 0..10 {
    for j in 0..10 {
        if j == 5 { continue outer } // skips to next iteration of outer loop
    }
}
```

---

## unreachable

Tells the compiler that this point in the code should never be reached. In debug builds, hitting it is a panic. In release builds, it's undefined behavior:

```mine
fn direction(d: Direction) []u8 {
    match d {
        .North => { return "up"    }
        .South => { return "down"  }
        .East  => { return "right" }
        .West  => { return "left"  }
    }
    unreachable // match is exhaustive, but compiler may need this
}
```

Use `unreachable` when you know a path is impossible but the compiler can't prove it.

---

→ [Control Flow in Mine](./control-flow-in-mine)

→ [Deferred Statements in Mine](./deferred-stmts-in-mine)