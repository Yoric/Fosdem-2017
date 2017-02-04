class: middle, center

# Redox OS
## A safety-first µKernel in Rust

David Teller, Mozillian

---

# What is Redox?

- A safety-first µKernel in Rust.
- Un*x-y.
- Running on existing hardware.
- Young and free.

---

# Which runs

.fullimage[![](screenshot.png)]

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

# Why your smart device *can't* upgrade

- Your device needs a custom kernel.
- Your device needs custom drivers.
- ... game over

In IoT, *you* can't afford monolithic kernels.

---

# IoT + µKernel FTW!

- Your customizations live in userland.
- Upgrade the kernel, the drivers, your code independently.
- No friction.
- Also, easier to implement livepatching.
- Also, security.

---

# Security + µKernels FTW!

- The worst that can happen to you is kernel compromise.
- One process per file system, driver, security, ...
- Isolation hampers compromise escalation.
- Also, easier to track compromised processes.
- Also, upgrades.

---

class: middle, center

# About Rust

---

# Rust is...

- A modern systems programming language.
- Very safe, very fast, very high-level.
- Zero-cost abstractions in most cases.
- Zero-cost safety in most cases.
- Aims to replace C++ for systems programming.
- Scriptable compiler.

---

# Zero-cost abstractions

```rust
// Compiled with inlining.
fn main() {                                    // asm: 7 instr
    let foo_1 = (1..100).filter(|x| x%2 == 0); // asm: 0 instr
    let foo_2 = foo_1.map(|x| x / 2);          // asm: 0 instr

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
```
Ok
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

# File systems

- Pluggable file system processes.
- The kernel implements file *descriptors*, registration and dispatching.
- Process implements files, paths, ACLs...
- Open `file:foo/bar`, `pipe:`, `rand:`, ... to access different file systems.

```rust
// Extract of `rand:` filesystem.
fn read(&mut self, _: usize, buf: &mut [u8]) -> Result<usize> {
    let mut i = 0;
    for chunk in buf.chunks_mut(8) {
        let mut rand = self.prng.next_u64();
        for b in chunk.iter_mut() {
            *b = rand as u8;
            rand = rand >> 8;
            i += 1;
        }
    }
    Ok(i)
}
```


---
# Userspace drivers

- Redox supports pluggable driver processes.
- The kernel implements address mapping.
- Guess how IRQs are implemented?
- Guess how drivers are exposed?

---

class: middle, center
# Towards capabilities
Very much a WIP

---

# Capabilities-based security

- A *security capability* is an unforgeable right to do *something*.
- If you have a capability, you can use it, share it, drop it, weaken it.
- In safe programming languages, encapsulation + methods/closures.

---

# What's the point?

- Anything can serve as a sandbox for anything.
- Avoids messy "global namespace" side-effects.
- Plays nice with static/dynamic analysis.
- Dynamic analysis can help you pinpoint compromised processes.

---

# Capabilities in Unix

- Typical Unices don't have (these) capabilities.
- User/group, SELinux, GRSecurity, "application firewalls", containers...
  are much less flexible.

- (Actual capabilities in Unix: Capsicum)

---

# Capabilities in (future) Redox (1)

- A file descriptor *is* a capabilities.
- Some file descriptors give you access to other files.

```rust
// Assume that we already have access to `policy`.
let policy = File::open("file:/home/yoric/*.txt:r", ...);

// We can use/weaken it to open a specific file.
let readme = File::open("file:/home/yoric/readme.txt", policy);
```

---

# Capabilities in (future) Redox (2)

- Use files to send file descriptors.

```rust
let cable = Cable::open("cable:").unwrap(); // Empty path.
let pid = Process::fork().unwrap();
if pid == 0 {
  // ... Child process
} else {
  // Parent sandbox.
  // For this example, it behaves as a sandbox.

  // Open a file on behalf of the other process.
  let file = File::open("file:user/potus/mail/privkey.txt", ...).unwrap();

  // Now send the file to the other process.
  cable.write(CableMessage {
    files: [fd],
    key: b"private key",
    data: []
  }).unwrap();
}
```

---

class: middle, center

# Time to wrap up

---

# Redox

Redox is:
- designed for safety & security;
- cool & promising;
- incomplete & unfunded;
- waiting for you on https://redox-os.org/ !


