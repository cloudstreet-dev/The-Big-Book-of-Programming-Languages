# Chapter 35: Assembly - As Close to Metal as It Gets

## The Hardware's Native Language

Assembly language is as close as you can get to machine code while remaining (somewhat) human-readable. Each CPU architecture has its own assembly language—x86, ARM, MIPS, etc.—and each instruction maps directly to machine code.

Assembly is rarely written by hand in 2025, but understanding it helps you comprehend how computers actually work and what higher-level languages are doing behind the scenes.

## The Philosophy

**Direct hardware control**: Every instruction is explicit.

**Maximum performance**: No abstraction overhead.

**Architecture-specific**: Different assembly for each CPU.

**Educational**: Understand how computers actually execute code.

## What It Looks Like (x86-64)

```asm
; x86-64 Assembly (AT&T syntax)
.section .data
msg: .ascii "Hello, World!\n"
msg_len = . - msg

.section .text
.global _start

_start:
    ; Write system call
    mov $1, %rax        # syscall number (write)
    mov $1, %rdi        # file descriptor (stdout)
    mov $msg, %rsi      # buffer address
    mov $msg_len, %rdx  # buffer length
    syscall

    ; Exit system call
    mov $60, %rax       # syscall number (exit)
    mov $0, %rdi        # exit code
    syscall
```

## The Verdict

Assembly is the lowest-level programming language. It's used for:
- Bootloaders and embedded systems
- Performance-critical code sections
- Understanding how CPUs work
- Reverse engineering and security research

You don't write applications in assembly. You write small, critical sections or study it to understand computer architecture.

---

**Next**: [Chapter 36: WebAssembly - The Browser's New Universal Runtime](36-webassembly.md)
