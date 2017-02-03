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

Also, why IoT deserves µKernels.

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

---

# Security + µKernels FTW!

---

# µKernels 

---

# Why you should consider a µKernel

- Safety.
- Survivability.
- Upgradability.

---

## Why you should consider Rust

- Zero-cost safety.

// FIXME: Example: `pid`.
// FIXME: An example of `Send` or `Sync`.

# Life in userspace

## Userspace file systems

// FIXME: Diagram of how it works?
// FIXME: A simple one. Maybe `/dev/null`.

## Userspace drivers

// FIXME: vesad

# Towards capabilities (WIP)

## About capabilities

## Capabilities in Unix

// FIXME: 1 capability == 1 file
// FIXME: weakening capabilities == `openat`
// FIXME: sharing capabilities ~= `pipe` (or sockets)