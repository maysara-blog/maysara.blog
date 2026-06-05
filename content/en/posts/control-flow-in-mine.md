---
title: "Control Flow in Mine"
date: 2026-05-25
draft: false
description: "Control Flow in Mine Language"
---

Control flow in Mine is about one thing: deciding what runs next.

In Mine, control flow is expressions. `if`, `match`, `try`, `catch` all produce a value when used in a context that expects one, or work as statement expressions when you don't need the value. This is different from exit statements (`return`, `fail`, `break`, `continue`, `unreachable`) and deferred statements (`defer`, `errdefer`), which are covered separately.

---

## if / else

```mine
if cond {
    // ...
} else if other {
    // ...
} else {
    // ...
}
```

Used as an expression, all branches must return the same type:

```mine
let label = if score > 100 { "high" } else { "low" }
```

---

## Conditional expression

Mine also supports the short ternary form:

```mine
let x = cond ? 1 as i32 : 2 as i32
```

Same semantics as `if/else` as an expression. Use whichever reads better in context.

---

## match

`match` checks a value against patterns. It is exhaustive (the compiler tells you if you missed a case):

```mine
match direction {
    .North => { "up"    }
    .South => { "down"  }
    .East  => { "right" }
    .West  => { "left"  }
}
```

Use `else` as a fallback to cover remaining cases:

```mine
match direction {
    .North => { "up" }
    else   => { "other" }
}
```

`match` works with any type:

```mine
match x {
    0    => { "zero" }
    1    => { "one"  }
    else => { "many" }
}
```

With ranges (`..` is exclusive, `..=` is inclusive):

```mine
match score {
    0..50    => { "fail"      } // 0 to 49
    50..75   => { "pass"      } // 50 to 74
    75..90   => { "good"      } // 75 to 89
    90..=100 => { "excellent" } // 90 to 100
    else     => { "invalid"   }
}
```

With tagged unions, capture the payload with `(binding)`:

```mine
match shape {
    .Circle(c) => { c.radius }
    .Rect(r)   => { r.width * r.height }
}
```

With error-returning functions, `ok(v)` captures the success value:

```mine
match divide(a, b) {
    ok(v)           => { v }
    .DivisionByZero => { 0.0 as f32 }
    else            => { 0.0 as f32 }
}
```

---

## try / catch

`try` propagates an error up to the caller. `catch` handles it inline. Both are expressions:

```mine
let result = try divide(a, b)                    // propagate
let result = divide(a, b) catch e { 0.0 as f32 } // handle inline
```

For more control, combine with `match`:

```mine
let result = match divide(a, b) {
    ok(v) => { v }
    else  => { 0.0 as f32 }
}
```

---

## orelse (`??`)

Unwrap an optional or fall back to a default value. Use `??` or the `orelse` keyword, they're identical:

```mine
let x: ?i32 = null
let y = x ?? 0 as i32     // 0 (x is null)
let y = x orelse 0 as i32 // same thing

let x: ?i32 = 42 as i32
let y = x ?? 0 as i32     // 42 (x has value)
```

Chain multiple optionals:

```mine
let y = a ?? b ?? 0 as i32         // ?? style
let y = a orelse b orelse 0 as i32 // orelse style
```

---

## comptime if / comptime match

Add `comptime` to resolve a branch at compile time. The other branches are discarded from the binary entirely:

```mine
comptime if PLATFORM == .windows {
    // only compiled on windows
} else {
    // only compiled elsewhere
}

comptime match ARCH {
    .x86_64 => {..}
    .arm64  => {..}
}
```

---

→ [Exit Statements in Mine](./exit-stmts-in-mine)

→ [Deferred Statements in Mine](./deferred-stmts-in-mine)
