---
title: "Error Handling in Mine"
date: 2026-05-24
draft: false
description: "Error Handling in Mine"
---

I don't like exceptions. They're invisible (you call a function, and somewhere deep inside it throws something you didn't expect, and now you're debugging a stack trace at 2am).

Mine doesn't have exceptions. Errors are values, they're explicit, and the compiler makes sure you handle them.

---

## Defining errors

Use `err` to define a named set of error variants:

```mine
err MathErrors = [DivisionByZero, Overflow, Underflow]
err FileErrors = [NotFound, PermissionDenied, Corrupted]
```

No special `error` keyword, no global error registry like Zig's `anyerror`.

---

## Functions that can fail

Add `ErrorSet!ReturnType` to the signature. Use a named set or define errors inline:

```mine
// named error set
fn divide(a: f32, b: f32) MathErrors!f32 {
    if b == 0.0 { fail DivisionByZero }
    return a / b
}

// inline error set (no need to define separately)
fn divide(a: f32, b: f32) [DivisionByZero, Overflow]!f32 {
    if b == 0.0 { fail DivisionByZero }
    return a / b
}
```

`fail` returns an error variant without the dot (you're saying what you mean: this function *failed*, here's why).

A function that can fail but returns nothing:

```mine
fn save(data: []u8) [PermissionDenied, DiskFull]!void {
    if !hasPermission() { fail PermissionDenied }
    // ...
}
```

---

## Handling errors

**`try`** (propagate the error up to the caller):

```mine
fn process() MathErrors!f32 {
    let result = try divide(10.0, 0.0) // fails here, propagates up
    return result * 2.0
}
```

**`catch`** (handle inline):

```mine
let result = divide(10.0, 0.0) catch e { 0.0 }
```

**`match`** (handle each variant explicitly, variants use dot syntax in match):

```mine
match divide(a, b) {
    ok(v)            => { v * 2.0 }
    .DivisionByZero  => { 0.0 }
    .Overflow        => { maxof f32 }
}
```

`ok(v)` captures the success value. `match` is exhaustive (use `else` as a fallback):

```mine
match divide(a, b) {
    ok(v) => { v }
    else  => { 0.0 as f32 }
}
```

---

## Combining error sets

Named sets can be merged into a new set or inline in a signature:

```mine
err MathErrors = [DivisionByZero, Overflow]
err FileErrors = [NotFound, PermissionDenied]

// merge into a new named set
err AppErrors = [MathErrors, FileErrors, Unknown]

// merge inline in a signature
fn process() [MathErrors, FileErrors, Unknown]!void {..}

// or use the named merged set
fn process() AppErrors!void {..}
```

---

## Why I designed it this way?

Zig's error handling is good (explicit, no exceptions, compiler-enforced). But it has rough edges: `anyerror` is a global catch-all that weakens the guarantees, `return error.X` is verbose, and `catch |e| switch (e)` is awkward to read.

Mine keeps what's good and cleans up the rest:

|                 | Zig                        | Mine                                        |
| --------------- | -------------------------- | ------------------------------------------- |
| Define errors   | `const E = error{A, B}`    | `err E = [A, B]`                            |
| Inline errors   | `{A, B}!T`                 | `[A, B]!T`                                  |
| Return error    | `return error.A`           | `fail A`                                    |
| Propagate       | `try x`                    | `try x`                                     |
| Handle          | `catch \|e\| {}`           | `catch e {}` or `match`                     |
| Match success   | not supported              | `ok(v)`                                     |
| Merge sets      | `const M = Set1 \|\| Set2` | `err M = [Set1, Set2]`                      |
| Inline merge    | ✗ not supported            | `[Set1, Set2, Error]!T`                     |
| Catch-all       | `anyerror`                 | ✗ not allowed                               |

No surprises, no invisible control flow, no 2am stack traces :)

---

→ [Control Flow in Mine](./control-flow-in-mine)

→ [Exit Statements in Mine](./exit-stmts-in-mine)

→ [Deferred Statements in Mine](./deferred-stmts-in-mine)