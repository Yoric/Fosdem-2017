class: middle, center

# Redox OS
David Teller, Mozillian

---

# What is Redox?

- Un*x-y
- µKernel
- In Rust
- Running on existing hardware
- Young and free

---

![](screenshot.png)

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

# File systems

- Redox supports pluggable file system processes.
- The kernel implements file *descriptors*, registration and dispatching.
- Each file system implements files, paths, rights...

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
WIP

---

# About capabilities

- A *security capability* is an unforgeable right to do *something*.
- If you have a capability, you can use it, share it, drop it, weaken it.
- In safe programming languages, methods/closures.

- Typical Unices don't have capabilities.
- User/group, SELinux, GRSecurity, "application firewalls", containers...
  are much less flexible.
- Capsicum is a Linux/BSD + capabilities.

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
fn main() {
  let cable = Cable::open("cable:").unwrap(); // Empty path.
  let pid = Process::fork().unwrap();
  if pid == 0 {
    // Child process.
    // For this simplified example, we assume that this process is
    // running unprivileged and has somehow dropped rights to open
    // files.

    // Receive data from `cable`.
    if let Ok(message) = cable.read() {
      assert_eq!(message.files.len(), 1);
      assert_eq!(message.data.len(), 0);
      assert_eq!(message.key, b"private key"); // The key is typically used to demultiplex messages.
      // ...
    }
  } else {
    // Parent process.
    // For this example, it behaves as a sandbox.

    // Open a file on behalf of the other process.
    let file = File::open("file:user/potus/mail/privkey.txt").unwrap();

    // Now send the file to the other process.
    cable.write(CableMessage {
      files: [fd],
      key: b"private key",
      data: []
    }).unwrap();
  }
}
```

- Bonus: we can track compromised capabilities and processes!

---

class: middle, center

# Time to wrap up

---

# Redox

Redox is:
- cool;
- promising;
- incomplete;
- unfunded;
- waiting for you on https://redox-os.org/ !


