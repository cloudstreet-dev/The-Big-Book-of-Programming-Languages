# Chapter 12: Zig - C But Make It Modern

## The Pragmatic Systems Language

Zig was created in 2016 by Andrew Kelley, who was frustrated with C and C++'s complexity and footguns. He wanted a language that gave you C's control and performance but with better safety, clearer error handling, and no hidden control flow.

Zig is not trying to be Rust. It doesn't have a borrow checker or complex type system. Instead, it's trying to be "C, but done right"—a simple, low-level language that makes the right thing easy and the wrong thing obvious.

By 2025, Zig is still young but gaining traction. It's used in performance-critical applications, embedded systems, and as a C/C++ build tool. It's not mainstream yet, but its pragmatic approach is winning fans.

## The Philosophy

Zig's design principles:

**No hidden control flow**: If code executes, you can see it. No operator overloading, no exceptions, no implicit function calls.

**No hidden allocations**: Memory allocation is always explicit. You choose the allocator.

**Simple language**: Fewer features, more predictable behavior.

**Excellent C interop**: Call C code seamlessly, and C code can call Zig.

**Compile-time execution**: Run code at compile time for metaprogramming.

**Explicit over implicit**: Make it obvious what's happening.

**Incremental improvement**: Focus on practical problems, not theoretical purity.

The result is a language that feels like C but fixes many of its problems without adding Rust's complexity.

## What It Looks Like

```zig
const std = @import("std");

pub fn main() void {
    const stdout = std.io.getStdOut().writer();
    stdout.print("Hello, World!\n", .{}) catch {};
}

// Functions
fn add(a: i32, b: i32) i32 {
    return a + b;
}

// Structs
const Point = struct {
    x: f64,
    y: f64,

    pub fn init(x: f64, y: f64) Point {
        return Point{ .x = x, .y = y };
    }

    pub fn distance(self: Point, other: Point) f64 {
        const dx = self.x - other.x;
        const dy = self.y - other.y;
        return @sqrt(dx * dx + dy * dy);
    }
};

// Error handling
const FileError = error{
    NotFound,
    PermissionDenied,
};

fn openFile(path: []const u8) FileError!File {
    if (/* file not found */) {
        return FileError.NotFound;
    }
    return file;
}

// Using errors
const file = try openFile("data.txt");

// Arrays and slices
const array = [_]i32{ 1, 2, 3, 4, 5 };
const slice: []const i32 = array[0..3];

// Optionals (nullable)
var maybe_value: ?i32 = null;
maybe_value = 42;

if (maybe_value) |value| {
    std.debug.print("Value: {}\n", .{value});
}
```

Key features visible here:
- **Type inference with `const` and `var`**: `const` is immutable, `var` is mutable
- **Error unions**: `FileError!File` means "File or an error"
- **`try` keyword**: Propagates errors
- **`catch` keyword**: Handles errors
- **Optionals with `?`**: Explicit null handling
- **Compile-time known sizes**: `[_]` infers array size

## The Type System

Zig's type system is simple and explicit:

```zig
// Integers (explicit sizes)
const x: i32 = 42;      // 32-bit signed
const y: u64 = 100;     // 64-bit unsigned
const z: i8 = -128;     // 8-bit signed

// Floats
const pi: f64 = 3.14159;
const e: f32 = 2.71828;

// Booleans
const flag: bool = true;

// Pointers
const ptr: *i32 = &x;
const value = ptr.*;    // Dereference

// Arrays (size known at compile time)
const array: [5]i32 = [_]i32{ 1, 2, 3, 4, 5 };

// Slices (runtime size)
const slice: []const i32 = array[0..3];

// Strings are just slices of bytes
const name: []const u8 = "Alice";

// Optionals (nullable)
const maybe: ?i32 = null;

// Enums
const Color = enum {
    red,
    green,
    blue,
};

// Tagged unions
const Value = union(enum) {
    int: i32,
    float: f64,
    string: []const u8,
};
```

No implicit conversions. No surprises. Everything is explicit.

## Error Handling: Better Than Exceptions

Zig uses error unions instead of exceptions:

```zig
const MathError = error{
    DivisionByZero,
    Overflow,
};

fn divide(a: i32, b: i32) MathError!i32 {
    if (b == 0) {
        return MathError.DivisionByZero;
    }
    return @divTrunc(a, b);
}

// Using try to propagate errors
fn doMath() MathError!void {
    const result = try divide(10, 0);  // Returns error
    std.debug.print("{}\n", .{result});
}

// Using catch to handle errors
fn doMathSafely() void {
    const result = divide(10, 0) catch |err| {
        std.debug.print("Error: {}\n", .{err});
        return;
    };
    std.debug.print("Result: {}\n", .{result});
}

// Or provide a default
const result = divide(10, 0) catch 0;
```

Errors are values, not control flow. The compiler ensures you handle them.

## Memory Management: Explicit Allocators

Unlike C, Zig makes allocation explicit and customizable:

```zig
const std = @import("std");

pub fn main() !void {
    // Choose your allocator
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();

    const allocator = gpa.allocator();

    // Allocate memory
    const memory = try allocator.alloc(u8, 100);
    defer allocator.free(memory);

    // Use memory...
}

// Different allocators for different needs
const page_allocator = std.heap.page_allocator;  // OS pages
const c_allocator = std.heap.c_allocator;        // C's malloc/free
const arena = std.heap.ArenaAllocator.init(allocator);  // Arena (free all at once)
```

This explicit approach:
- Makes allocation visible
- Lets you choose the right allocator for your use case
- Enables testing with mock allocators
- Prevents hidden performance costs

## Compile-Time Execution: Metaprogramming

Zig runs code at compile time for powerful metaprogramming:

```zig
// comptime parameters
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}

// Compile-time computation
const factorial = comptime blk: {
    var result = 1;
    var i = 1;
    while (i <= 10) : (i += 1) {
        result *= i;
    }
    break :blk result;
};
// factorial is computed at compile time

// Generic data structures
fn ArrayList(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,

        pub fn init(allocator: Allocator) ArrayList(T) {
            return .{
                .items = &[_]T{},
                .len = 0,
            };
        }
    };
}

const int_list = ArrayList(i32).init(allocator);
```

Compile-time execution is Zig's answer to C++ templates and Rust generics, but simpler and more explicit.

## C Interoperability

Zig can call C code and be called by C:

```zig
// Import C header
const c = @cImport({
    @cInclude("stdio.h");
});

pub fn main() void {
    c.printf("Hello from C!\n");
}

// Export to C
export fn add(a: i32, b: i32) i32 {
    return a + b;
}
```

Zig can also:
- Compile C code (zig is a C compiler!)
- Cross-compile C code easily
- Mix Zig and C in the same project

This makes Zig excellent for incrementally replacing C code.

## What Zig Is Best For

**Systems programming**: Operating systems, device drivers, embedded systems.

**Replacing C**: Incremental replacement of C codebases.

**Performance-critical code**: Game engines, compilers, databases.

**Cross-compilation**: Zig makes cross-compiling trivial.

**Embedded systems**: Low-level control without C's pitfalls.

**Build tool**: Using `zig cc` as a C/C++ compiler and cross-compiler.

## What Zig Is Worst For

**High-level applications**: Use Python, Go, or Rust for rapid development.

**Web development**: Possible but not ideal. Use JavaScript, Python, or Go.

**Quick scripts**: Too low-level. Use Python or Bash.

**Large teams of beginners**: Manual memory management is challenging.

**Stable production systems**: Zig is still pre-1.0, language changes occur.

## The Ecosystem

Zig's ecosystem is young but growing:

**Build system**: Zig has its own build system (build.zig).

**Package manager**: Experimental package manager in progress.

**Standard library**: Growing. Covers basics but not as comprehensive as Rust or Go.

**Notable projects**:
- **Bun**: JavaScript runtime (parts written in Zig)
- **TigerBeetle**: Financial database
- Various game engines and embedded projects

The ecosystem will mature as the language stabilizes.

## Zig as a C/C++ Compiler

One of Zig's killer features: it's an excellent C/C++ compiler:

```bash
# Compile C code
zig cc -o program program.c

# Cross-compile easily
zig cc -target x86_64-linux -o program program.c
zig cc -target aarch64-macos -o program program.c

# Works with C++
zig c++ -o program program.cpp
```

Zig bundles clang and libc for many targets, making cross-compilation trivial. This alone makes it valuable even if you don't write Zig code.

## The Community

**The C Developers**: Tired of C's problems but not ready for Rust's complexity.

**The Performance Engineers**: Building databases, game engines, embedded systems.

**The Rust Skeptics**: Want memory safety but not Rust's learning curve.

**The Early Adopters**: Willing to deal with pre-1.0 instability for a better systems language.

**The Build Tool Users**: Using `zig cc` to compile C/C++ projects.

### Common Phrases
- "No hidden control flow"
- "Explicit is better"
- "It's like C but better"
- "Simpler than Rust, safer than C"
- "comptime is amazing"
- "Still waiting for 1.0"

## Zig vs Rust

The inevitable comparison:

**Rust**:
- Borrow checker prevents data races and memory errors
- Steeper learning curve
- More mature ecosystem
- Strongly typed with rich type system
- More popular and stable

**Zig**:
- Manual memory management (like C)
- Simpler, easier to learn
- Explicit over clever
- Younger, less stable
- Better C interop

**When to choose Zig**: You want C-level control without C's footguns, and you don't need Rust's strong safety guarantees.

**When to choose Rust**: You want the compiler to prevent entire classes of bugs, and you're willing to learn the borrow checker.

Both are excellent. Different tradeoffs.

## Defer: Cleanup Made Easy

Zig's `defer` ensures cleanup:

```zig
pub fn processFile() !void {
    const file = try std.fs.cwd().openFile("data.txt", .{});
    defer file.close();  // Runs when scope exits

    const data = try file.readToEndAlloc(allocator, 1024 * 1024);
    defer allocator.free(data);  // Cleanup allocation

    // Process data...
    // file.close() and allocator.free() run automatically
}
```

Multiple `defer` statements execute in reverse order (LIFO). This makes resource management much cleaner than C.

## Should You Learn Zig?

**Yes, if**:
- You do systems programming
- You're frustrated with C/C++ but find Rust too complex
- You want to understand low-level programming better
- You're interested in embedded systems or game development
- You want an excellent C/C++ cross-compilation toolchain

**No, if**:
- You need a stable, mature language for production
- You're building web apps or data science projects
- You want strong compile-time safety guarantees (use Rust)
- You prefer garbage-collected languages

**Maybe, if**:
- You're curious about systems languages
- You want to replace C code incrementally
- You're interested in compile-time metaprogramming

## The Verdict

Zig is what you'd get if you redesigned C with 50 years of hindsight. It keeps C's simplicity and control but fixes many of its problems: better error handling, explicit allocation, no undefined behavior guarantees (where possible), and powerful compile-time execution.

Is Zig ready for production? It's pre-1.0 (as of 2025), so expect changes. But it's being used successfully in real projects. The language is simple enough that updates are manageable.

Zig won't replace Rust for projects where safety is paramount. But it might replace C for projects where simplicity and control matter more than compiler-enforced safety. It's a pragmatic choice: safer than C, simpler than Rust.

The biggest appeal? Zig feels like C but without the constant dread of undefined behavior. You still manage memory manually, but the language helps you get it right. You still write low-level code, but it's clearer and more maintainable.

If C feels too dangerous and Rust feels too complex, Zig might be your sweet spot. Give it a try. Write some low-level code that doesn't make you nervous. Use `comptime` to blow your mind. Cross-compile to 20 platforms without thinking.

Zig is the systems language for people who miss C's simplicity but not its footguns.

---

**Next**: [Chapter 13: TypeScript - JavaScript Grows Up](13-typescript.md)
