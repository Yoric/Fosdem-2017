class: middle, center

# Redox OS
David Teller, Mozillian

---

# What is Redox?

- Unix (not Posix)
- µKernel
- In Rust
- Running on existing hardware

---

class: middle, center

# About the µKernel

Also, why IoT *deserves* µKernels.

---

# µKernels

- Delegate as much as you can to userland.
- Including file system, drivers, security...
- µKernels should rule the world :)

---

# Android *can't* upgrade

- Your device needs a custom kernel.
- Your device needs custom drivers.
- => Your life is miserable.
  - Will upstream accept your patches?
  - Can you afford to follow upstream?
  - Can you afford to integrate your new drivers?
  - For how long?

In IoT, only megacorps can afford monolithic kernels.

---

# IoT + µKernel FTW!

- Your customizations live in userland.
- Upgrade the kernel, the drivers, your code independently.
- No friction.
- Also, easier to implement livepatching.
- Also, easier to track compromised processes.

---

# Security + µKernels FTW!

- The worst that can happen to you is kernel compromise.
- One process per file system, driver, security, ...
- => isolated from the kernel
- => isolated from each other
- also, easier to shutdown/restart/upgrade.

---

class: middle, center

# About Rust

---

# Rust is...

- A modern systems programming language.
- Very safe, very fast, very high-level.
- Zero-cost abstractions in most cases.
- Zero-cost safety in most cases.
- Candidate for replacing C++ for systems programming.
- Scriptable compiler.

---

# Zero-cost abstractions

```rust
// Compiled with inlining.
fn main() {                                    // asm: 7 loc
    let foo_1 = (1..100).filter(|x| x%2 == 0); // asm: 0 loc
    let foo_2 = foo_1.map(|x| x / 2);          // asm: 0 loc
    println!("foo_2: {:?}", foo_2);            // asm: 16 loc

    let foo_3 : Vec<u64> = foo_2.collect();    // asm: 67 loc
    println!("foo_3: {:?}", foo_3);            // asm: 19 loc
}                                              // asm: 40 loc
```

---

# Zero-cost safety

// FIXME: Example: `pid`.

---

## Also, concurrency

// FIXME: An example of `Send` or `Sync`.

---

class: middle, center
# Life in userspace

---
# Userspace file systems

// FIXME: Diagram of how it works?
// FIXME: A simple one. Maybe `/dev/null`.
---
# Userspace drivers

// FIXME: vesad

---

class: middle, center
# Towards capabilities (WIP)

---

# About capabilities

---

# Capabilities in Unix

// FIXME: 1 capability == 1 file
// FIXME: weakening capabilities == `openat`
// FIXME: sharing capabilities ~= `pipe` (or sockets)
// FIXME: Tracking compromised processes.