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

# Why you can't upgrade your IoT devices/smartphones

- Your device needs a custom kernel.
- Your device needs custom drivers.
- => Your life is miserable.
  - Will upstream accept your patches?
  - Can you afford to follow upstream?
  - Can you afford to integrate your new drivers?
  - For how long?

---

# IoT + µKernel FTW!

- Your customizations live in userland.
- Upgrade the kernel, the drivers, your code independently.
- No friction.
- Also, easier to implement livepatching.

---

# Security + µKernels FTW!

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
- Candidate for replacing C++ for systems programming.
- Scriptable compiler.

---

# Zero-cost safety.

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