---
title: "Error Handling in Mine"
date: 2026-05-29
draft: false
description: "Error Handling in Mine"
---

I don't like exceptions. They're invisible (you call a function, and somewhere deep inside it throws something you didn't expect, and now you're debugging a stack trace at 2am).

Mine doesn't have exceptions. Errors are values, they're explicit, and the compiler makes sure you handle them.

---

## Defining errors

Use `err` to define a named set of error variants:

```mine
err MathErrors = .DivisionByZero | .Overflow | .Underflow
err FileErrors = .NotFound | .PermissionDenied | .Corrupted
```

Consistent with `def` (same `|` syntax, same `.Variant` style). No special `error` keyword, no global error registry like Zig's `anyerror`.

---

## Functions that can fail

Add `ErrorSet!ReturnType` to the signature:

```mine
fn divide(a: f32, b: f32) MathErrors!f32 {
    if b == 0.0 { fail .DivisionByZero }
    return a / b
}
```

`fail` returns an error variant (cleaner than Zig's `return error.DivisionByZero`). You're saying what you mean: this function *failed*, here's why.

A function that can fail but returns nothing:

```mine
fn save(data: []u8) FileErrors!void {
    if !hasPermission() { fail .PermissionDenied }
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

**`match`** (handle each variant explicitly):

```mine
match divide(a, b) {
    ok(v)           => { v * 2.0 }
    .DivisionByZero => { 0.0 }
    .Overflow       => { maxof f32 }
}
```

`ok(v)` captures the success value. Error variants use the same `.Variant` syntax as enums (flat, no nesting, no `Ok`/`Err` wrapper types like Rust).

`match` is exhaustive (if you don't cover all variants the compiler tells you. Use `else` as a fallback):

```mine
match divide(a, b) {
    ok(v) => { v }
    else  => { 0.0 as f32 }
}
```

---

## Combining error sets

Error sets can be merged with `|` (same syntax as defining them):

```mine
err MathErrors = .DivisionByZero | .Overflow
err FileErrors = .NotFound       | .PermissionDenied

// merge into a new set
err AppErrors = MathErrors      | FileErrors | .Unknown
//            = .DivisionByZero | .Overflow  | .NotFound | .PermissionDenied | .Unknown
```

Use the merged set in function signatures:

```mine
fn process() AppErrors!void {..}

// or inline without defining a name
fn process() MathErrors|FileErrors!void {..}
```

---

## Why I designed it this way

Zig's error handling is good (explicit, no exceptions, compiler-enforced). But it has rough edges: `anyerror` is a global catch-all that weakens the guarantees, `return error.X` is verbose, and `catch |e| switch (e)` is awkward to read.

Mine keeps what's good (explicit errors, `try` propagation, exhaustive handling) and cleans up the rest:

|                  | Zig                        | Mine                     |
| ---------------- | -------------------------- | ------------------------ |
| Define errors    | `const E = error { A, B }` | `err E = .A \| .B`       |
| Return error     | `return error.A`           | `fail .A`                |
| Propagate        | `try x`                    | `try x`                  |
| Handle           | `catch \|e\| switch`       | `catch e { }` or `match` |
| Success in match | --                         | `ok(v)`                  |
| Global catch-all | `anyerror`                 | ✗ not allowed            |

No surprises, no invisible control flow, no 2am stack traces :)

---

→ [Mutability in Mine](./mutability-in-mine)

→ [Visibility in Mine](./visibility-in-mine)

→ [Evaluation Phases in Mine](./evaluation-phases-in-mine)

→ [How Types Work in Mine](./how-types-work-in-mine)

---

<br>