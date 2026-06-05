---
title: "Deferred Statements in Mine"
date: 2026-05-27
draft: false
description: "Deferred Statements in Mine Language"
---

Deferred statements schedule code to run at the end of the current scope, regardless of how that scope exits. They're how Mine handles cleanup without relying on a garbage collector or try/finally blocks.

---

## defer

Schedules a block to run when the current scope ends, no matter how it exits (normally, via `return`, via `fail`, anything):

```mine
fn process() FileErrors!void {
    let file = try open(path)
    defer file.close()          // guaranteed to run when the function exits

    try write(file, data)       // if this fails, defer still runs
    try flush(file)             // same here
}
```

Multiple `defer` statements run in reverse order (last in, first out):

```mine
fn example() void {
    defer @printn("third")
    defer @printn("second")
    defer @printn("first")
    // prints: first, second, third
}
```

---

## errdefer

Like `defer`, but only runs if the scope exits with an error. If the function returns normally, `errdefer` is skipped:

```mine
fn setup() AppErrors!Resource {
    let r = try allocate()
    errdefer r.free()           // only runs if something below fails

    try r.initialize()          // if this fails, r.free() runs
    try r.configure()           // if this fails, r.free() runs

    return r                    // success, errdefer is skipped
}
```

---

## defer vs errdefer

| Statement | Runs on success | Runs on error |
| --------- | --------------- | ------------- |
| `defer`   | yes             | yes           |
| `errdefer`| no              | yes           |

Use `defer` for cleanup that always needs to happen (closing files, releasing locks).
Use `errdefer` for cleanup that only makes sense on failure (freeing a resource that won't be used).

---

## Why deferred statements?

The alternative is try/finally blocks. They work, but they nest. Every resource you acquire needs another layer of try/finally around it. It gets deep fast.

`defer` stays flat. You acquire a resource, immediately write `defer resource.release()`, and move on. The cleanup is right next to the acquisition, where it's easy to see and hard to forget.

I borrowed this from Zig and Go, and I think it's one of the best ideas in modern language design :)

---

→ [Control Flow in Mine](./control-flow-in-mine)

→ [Exit Statements in Mine](./exit-stmts-in-mine)