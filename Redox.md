class: middle, center

# Redox OS
David Teller, Mozillian

---

# What is Redox?

- Un*x-y
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
fn main() {                                    // asm: 7 instr
    let foo_1 = (1..100).filter(|x| x%2 == 0); // asm: 0 instr
    let foo_2 = foo_1.map(|x| x / 2);          // asm: 0 instr
    println!("foo_2: {:?}", foo_2);            // asm: 16 instr

    for i in foo_2 {
      print!("=> {}", i);                      // asm: 27 instr
    }
}                                              // asm: 7 instr
// => 4 function calls
// => 0 allocation
```

---

# Zero-cost safety

```rust
#[derive(PartialEq, Eq, PartialOrd, Ord, Debug)]
pub struct Pid(u64); // Backed by a `u64`, different type.

impl Pid {
  pub fn increment(self) -> Pid {
    Pid(self.0 + 1)               // compiled as `+= 1`
  }
}
```

---

# Zero-cost safety

```rust
fn foo() {
  let ptr = Box::new(42); // Allocate a pointer.
  if true {
    drop(ptr)
  }
  println!("ptr: {}", *ptr)
}
```
```
Error: Value used after move!
  println!("ptr: {}", *ptr)
                      ^^^^
```

---

## Zero-cost *thread* safety (1)

```rust
fn foo() {
  let mut ptr = Box::new(42); // Allocate a pointer.
  thread::spawn(|| {          // Spawn a thread.
    *ptr = 0;                 // Modify the pointer.
  });
}
```
```
Error: closure may outlive the current function,
 but it borrows `ptr`, which is owned by the current
 function
```

---

## Zero-cost *thread* safety (2)

```rust
fn foo() {
  let mut ptr = Box::new(42); // Allocate a pointer.
  thread::spawn(move || {     // Spawn a thread, steal `ptr`.
    *ptr = 0;                 // Modify the pointer.
  });
}
```

---

## Zero-cost *thread* safety (3)

```rust
fn foo() {
  let mut ptr = Box::new(42); // Allocate a pointer.
  thread::spawn(move || {     // Spawn a thread, steal `ptr`.
    *ptr = 0;                 // Modify the pointer.
  });
  println!("ptr: {}", *ptr);
}
```
```
Error: Value used after move!
  println!("ptr: {}", *ptr)
                      ^^^^
```

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