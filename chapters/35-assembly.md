# Chapter 35: Assembly - As Close to Metal as It Gets

## The Hardware's Native Language

Assembly language sits at the lowest level of programming above raw machine code (binary). Each CPU architecture has its own assembly language—x86, x86-64, ARM, MIPS, RISC-V, etc.—and each instruction maps nearly one-to-one with machine code.

Assembly emerged in the 1940s-1950s with the first computers. Instead of programming in raw binary (punched cards, toggle switches), programmers used mnemonics: `MOV` instead of `10001011`, `ADD` instead of `00000011`. An assembler translates these mnemonics to machine code.

By 2025, almost nobody writes assembly for entire applications. Compilers generate better assembly than humans for most code. But assembly remains crucial for bootloaders, embedded systems, performance-critical sections, and understanding how computers actually work.

Learning assembly is like learning Latin—you won't speak it daily, but it illuminates everything built on top of it.

## The Philosophy

**Direct hardware control**: Every instruction explicitly tells the CPU what to do. No abstractions, no hidden operations.

**Maximum performance**: No compiler overhead, no runtime, no garbage collection. Just the CPU executing your instructions.

**Architecture-specific**: Different CPUs have different instruction sets. x86 assembly doesn't run on ARM. Code is not portable.

**Minimal abstraction**: Registers, memory addresses, jumps. That's it. You build everything else yourself.

**Educational value**: Understanding assembly reveals how high-level languages work. Loops, functions, objects—all become concrete.

The result: The closest you can get to the hardware while remaining somewhat human-readable.

## CPU Architectures

**x86 (Intel/AMD, 32-bit)**: Complex Instruction Set Computing (CISC). Dominant PC architecture for decades. Now mostly legacy.

**x86-64 (AMD64)**: 64-bit extension of x86. Modern PCs and servers. More registers, 64-bit addressing.

**ARM**: Reduced Instruction Set Computing (RISC). Dominant in mobile, tablets, increasingly in servers (Apple M-series, AWS Graviton).

**RISC-V**: Open-source RISC architecture. Growing in embedded systems and education.

**MIPS**: Academic RISC architecture. Used in education, some embedded systems.

We'll focus on x86-64 (AT&T syntax) since it's most common and educational.

## What It Looks Like (x86-64 AT&T Syntax)

### Hello World

```asm
; x86-64 Assembly (AT&T syntax on Linux)
.section .data
msg:
    .ascii "Hello, World!\n"
    msg_len = . - msg

.section .text
.global _start

_start:
    ; write(1, msg, msg_len) system call
    movq $1, %rax           # syscall number for write
    movq $1, %rdi           # file descriptor (stdout)
    movq $msg, %rsi         # buffer address
    movq $msg_len, %rdx     # buffer length
    syscall

    ; exit(0) system call
    movq $60, %rax          # syscall number for exit
    movq $0, %rdi           # exit code
    syscall
```

Assemble and link:
```bash
as -o hello.o hello.s
ld -o hello hello.o
./hello
```

### Registers

```asm
; x86-64 has 16 general-purpose 64-bit registers:
; rax, rbx, rcx, rdx, rsi, rdi, rbp, rsp, r8-r15

; Register naming (for rax):
; rax  - 64 bits (full register)
; eax  - lower 32 bits
; ax   - lower 16 bits
; al   - lower 8 bits
; ah   - bits 8-15

; Example: moving data
movq $42, %rax          # rax = 42
movq %rax, %rbx         # rbx = rax
movq (%rax), %rbx       # rbx = *rax (load from memory)
movq %rbx, (%rax)       # *rax = rbx (store to memory)
```

### Arithmetic

```asm
; Addition
movq $5, %rax
addq $3, %rax           # rax += 3 (rax = 8)

; Subtraction
movq $10, %rax
subq $4, %rax           # rax -= 4 (rax = 6)

; Multiplication
movq $7, %rax
imulq $3, %rax          # rax *= 3 (rax = 21)

; Division (complex!)
movq $20, %rax
movq $0, %rdx           # Clear rdx (upper 64 bits)
movq $4, %rbx
idivq %rbx              # rax = rdx:rax / rbx
                        # rax = quotient (5)
                        # rdx = remainder (0)

; Increment/Decrement
incq %rax               # rax++
decq %rax               # rax--
```

### Control Flow

```asm
; Conditional jumps
movq $5, %rax
cmpq $10, %rax          # Compare rax with 10
jl less_than            # Jump if less
jg greater_than         # Jump if greater
je equal                # Jump if equal
jne not_equal           # Jump if not equal

less_than:
    ; Code here
    jmp done

greater_than:
    ; Code here
    jmp done

equal:
    ; Code here

done:
    ; Continue

; Unconditional jump
jmp target_label

; Loop example
    movq $10, %rcx      # Loop counter
loop_start:
    ; Loop body
    decq %rcx
    jnz loop_start      # Jump if not zero
```

### Functions (Calling Convention)

```asm
; x86-64 System V calling convention (Linux/macOS):
; Arguments: rdi, rsi, rdx, rcx, r8, r9, then stack
; Return value: rax
; Caller-saved: rax, rcx, rdx, rsi, rdi, r8-r11
; Callee-saved: rbx, rbp, r12-r15

; Function definition
add_numbers:
    ; Arguments in rdi (a) and rsi (b)
    movq %rdi, %rax     # rax = a
    addq %rsi, %rax     # rax += b
    ret                 # Return (rax contains result)

; Calling the function
    movq $5, %rdi       # First argument
    movq $3, %rsi       # Second argument
    call add_numbers    # Call function
    ; Result in rax (8)

; Function with stack frame
factorial:
    pushq %rbp          # Save old base pointer
    movq %rsp, %rbp     # Set up new base pointer

    ; Function body here
    ; Local variables at negative offsets from rbp

    movq %rbp, %rsp     # Restore stack pointer
    popq %rbp           # Restore base pointer
    ret
```

### Arrays and Memory

```asm
; Define array in data section
.section .data
array:
    .quad 1, 2, 3, 4, 5     # 5 quad-words (64-bit each)

; Access array elements
    movq $array, %rax       # rax = address of array
    movq (%rax), %rbx       # rbx = array[0]
    movq 8(%rax), %rbx      # rbx = array[1] (8 bytes offset)
    movq 16(%rax), %rbx     # rbx = array[2] (16 bytes offset)

; Loop through array
    movq $array, %rax       # rax = array address
    movq $0, %rcx           # rcx = index
loop:
    movq (%rax,%rcx,8), %rbx  # rbx = array[rcx]
    ; Process rbx
    incq %rcx
    cmpq $5, %rcx
    jl loop
```

### Stack Operations

```asm
; Push and pop
pushq %rax              # Push rax onto stack (rsp -= 8)
popq %rbx               # Pop top of stack into rbx (rsp += 8)

; Stack grows downward in x86-64
; rsp points to top of stack
; rbp is base pointer (frame pointer)

; Example: saving registers
    pushq %rbx
    pushq %rcx
    ; Use rbx, rcx
    popq %rcx
    popq %rbx
```

## Real Example: Fibonacci

```asm
.section .data
result_msg: .ascii "Result: "
result_len = . - result_msg

.section .text
.global _start

_start:
    # Calculate fib(10)
    movq $10, %rdi
    call fibonacci
    # Result in rax

    # Exit
    movq $60, %rax
    movq $0, %rdi
    syscall

# Recursive fibonacci
# Input: rdi (n)
# Output: rax (fib(n))
fibonacci:
    # if (n <= 1) return n
    cmpq $1, %rdi
    jle base_case

    # Save n
    pushq %rdi

    # fib(n-1)
    decq %rdi
    call fibonacci
    pushq %rax              # Save fib(n-1)

    # fib(n-2)
    movq 8(%rsp), %rdi      # Restore original n
    subq $2, %rdi
    call fibonacci

    # Add results
    popq %rbx               # fib(n-1)
    addq %rbx, %rax         # rax = fib(n-1) + fib(n-2)

    # Restore stack
    addq $8, %rsp

    ret

base_case:
    movq %rdi, %rax
    ret
```

## Inline Assembly in C

```c
// GCC/Clang inline assembly
#include <stdio.h>

int main() {
    int a = 5, b = 3, result;

    // Inline x86-64 assembly
    __asm__ (
        "movl %1, %%eax\n\t"
        "addl %2, %%eax\n\t"
        "movl %%eax, %0"
        : "=r" (result)          // Output
        : "r" (a), "r" (b)       // Inputs
        : "%eax"                 // Clobbered registers
    );

    printf("Result: %d\n", result);  // 8
    return 0;
}
```

## The Good Parts

**Ultimate control**: You decide every single operation. No hidden allocations, no surprise overhead.

**Maximum performance**: When hand-tuned, assembly can be faster than any compiler. Critical for performance hotspots.

**Tiny binaries**: No runtime, no standard library. Bootloaders and embedded systems benefit.

**Understanding computers**: Learning assembly demystifies everything. You see how CPUs actually work.

**Security research**: Reverse engineering requires reading assembly. Understanding exploits requires writing it.

**Bare metal programming**: Bootloaders, operating system kernels, device drivers often use assembly for initialization.

## The Pain Points

**Architecture-specific**: Code for x86 won't run on ARM. No portability.

**No abstractions**: Want a string? Build it yourself. Want a function call? Manually manage the stack.

**Error-prone**: One wrong register, one bad offset, segmentation fault. No safety nets.

**Hard to read**: Assembly is write-once, read-never. Maintenance is nightmare fuel.

**Compilers are better**: Modern compilers generate better assembly than humans for 99% of code.

**Time-consuming**: What takes 5 lines in C might take 50 in assembly.

**Platform-specific tools**: Different assemblers (NASM, GAS), different syntax (AT&T vs Intel), different conventions.

## When You Actually Use Assembly

**Never for full applications**: Don't write applications in assembly. Just don't.

**Performance hotspots**: Profile your code, find the 1% that matters, optimize that in assembly if the compiler isn't good enough.

**Bootloaders**: The first code that runs when a computer boots. No operating system, no libraries. Pure assembly.

**Operating system kernels**: Context switching, interrupt handlers, low-level hardware interaction.

**Embedded systems**: Resource-constrained devices where every byte and cycle matters.

**Cryptography**: Performance-critical implementations of AES, SHA, etc. Hand-tuned assembly can be significantly faster.

**Reverse engineering**: Analyzing malware, understanding proprietary software, security research.

**Learning and education**: Understanding computer architecture, compilers, and how high-level languages work.

## Tools

**Assemblers**:
- GNU AS (GAS): Part of GNU binutils, AT&T syntax
- NASM: Netwide Assembler, Intel syntax, popular for x86
- MASM: Microsoft Macro Assembler (Windows)
- YASM: Rewrite of NASM

**Debuggers**:
- GDB: GNU Debugger, essential for assembly debugging
- LLDB: LLVM debugger
- WinDbg: Windows kernel debugger

**Disassemblers**:
- objdump: Part of binutils, disassembles object files
- Ghidra: NSA's reverse engineering tool
- IDA Pro: Commercial disassembler, industry standard

## Common Phrases You'll Hear

**"Compilers are better"**: Modern compilers optimize better than humans. Only hand-optimize bottlenecks.

**"Read the disassembly"**: Want to know what your C code actually does? Look at the assembly output.

**"Register allocation is the hard part"**: Deciding which variables go in which registers is complex.

**"Mind the calling convention"**: Function calls must follow platform-specific rules. Screw it up, segfault.

**"It's all pointers"**: At the assembly level, everything is memory addresses.

## The Verdict

Assembly is the language you learn to understand how computers work, not to write applications. It's the foundation beneath every high-level language.

**Use assembly when**:
- Writing bootloaders or OS kernels
- Optimizing critical hotspots after profiling
- Programming embedded systems with tight constraints
- Learning computer architecture
- Reverse engineering software
- Understanding security vulnerabilities

**Avoid assembly when**:
- Writing literally any application
- You value portability
- You want maintainable code
- Compiler-generated code is fast enough (it almost always is)
- You're on a deadline

Assembly teaches you what computers actually do. Every `if` statement, every function call, every object—they all become concrete operations on registers and memory.

Learn assembly not to use it, but to understand it. Read your compiler's output. See what your code becomes. Appreciate the abstractions high-level languages provide.

You'll probably never write a full program in assembly. But knowing assembly makes you a better programmer in every language.

It's the Latin of programming: dead for daily use, essential for understanding everything built on top of it.

---

**Next**: [Chapter 36: WebAssembly - The Browser's New Universal Runtime](36-webassembly.md)
