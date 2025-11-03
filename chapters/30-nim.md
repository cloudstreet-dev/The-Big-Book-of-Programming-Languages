# Chapter 30: Nim - Python's Syntax, C's Performance

## The Underdog's Promise

Nim (originally named Nimrod until biblical associations proved unfortunate) was created in 2008 by Andreas Rumpf, a German developer who asked a simple question: "What if Python were fast?"

Not "fast for a scripting language." Not "fast enough." Actually fast. C-level fast. Compile-to-native-code fast. Zero-overhead-abstraction fast.

The answer was Nim: a language with Python-like syntax that compiles to C, producing executables as fast as anything written in C or C++. It's statically typed with aggressive type inference, offers manual memory management with optional garbage collection, and provides a macro system powerful enough to make Lisp developers jealous.

By 2025, Nim remains niche. It's not backing startups, not taught in bootcamps, not demanded in job postings. But it has a devoted community who understand what it offers: a practical systems programming language that doesn't make you suffer.

## The Philosophy

**Efficiency first**: Nim compiles to C, C++, or JavaScript. The C backend produces code competitive with hand-written C. No virtual machine, no interpreter overhead, no garbage collection pauses (unless you want them).

**Expressiveness second**: Python-like syntax with significant indentation. Clean, readable code without brace noise. If you can read Python, you can read Nim.

**Elegance throughout**: Minimal punctuation. Strong type inference. Uniform function call syntax. The language gets out of your way.

**Metaprogramming power**: Compile-time execution, templates, and macros that operate on the abstract syntax tree. Build your own control flow, DSLs, and abstractions.

**Pragmatic choices**: Use garbage collection or manual memory management. Choose your trade-offs. The language provides mechanisms, not mandates.

The result is a language that feels high-level but performs low-level. Python on the outside, C on the inside.

## What It Looks Like

```nim
# Classic Hello World
echo "Hello, World!"

# Variables: var (mutable), let (immutable), const (compile-time)
var x = 42                    # Type inferred as int
let y = "hello"               # Immutable string
const Pi = 3.14159            # Compile-time constant

# Functions use 'proc' keyword
proc greet(name: string): void =
  echo "Hello, ", name, "!"

proc add(x, y: int): int =
  result = x + y              # 'result' is implicit return variable

# Or explicit return
proc multiply(x, y: int): int =
  return x * y

# Type inference works for functions too
proc divide(x, y: float) = x / y  # Return type inferred

# Collections
var numbers = @[1, 2, 3, 4, 5]      # Seq (dynamic array)
var squares = numbers.map(x => x * x)  # Lambda syntax

# Arrays (fixed size)
var grid: array[10, int]
grid[0] = 42

# Strings are mutable by default
var message = "Hello"
message.add(" World")
echo message

# Control flow - Python-like indentation
if x > 0:
  echo "Positive"
elif x < 0:
  echo "Negative"
else:
  echo "Zero"

# Case statement (more powerful than Python's match)
case x
of 0:
  echo "Zero"
of 1..10:
  echo "Small"
of 100, 200, 300:
  echo "Big round numbers"
else:
  echo "Something else"

# Loops
for i in 1..10:            # Range
  echo i

for item in numbers:       # Iterator
  echo item

while x > 0:
  dec x                    # Decrement

# Iterators are first-class
iterator countup(a, b: int): int =
  var current = a
  while current <= b:
    yield current
    inc current

for i in countup(5, 10):
  echo i
```

## Type System

Nim is statically typed with aggressive type inference:

```nim
# Type inference
var x = 42                    # int
var y = 3.14                  # float
var name = "Alice"            # string
var items = @[1, 2, 3]        # seq[int]

# Explicit types when needed
var temperature: float = 98.6
var count: int32 = 1000

# Type aliases
type
  UserId = int
  Email = string

var user: UserId = 12345

# Object types
type
  Person = object
    name: string
    age: int
    email: Email

  Employee = object of Person  # Inheritance
    employeeId: string
    salary: float

# Creating instances
var alice = Person(name: "Alice", age: 30, email: "alice@example.com")

# Tuple types
type Point = tuple[x: int, y: int]
var origin: Point = (0, 0)

# Or anonymous tuples
var coords = (x: 10, y: 20)
echo coords.x

# Enums
type
  Color = enum
    Red, Green, Blue

  Direction = enum
    North = "N"
    South = "S"
    East = "E"
    West = "W"

# Generics
proc identity[T](x: T): T =
  return x

proc swap[T](a, b: var T) =
  let temp = a
  a = b
  b = temp

# Generic types
type
  Stack[T] = object
    data: seq[T]

proc push[T](s: var Stack[T], item: T) =
  s.data.add(item)

proc pop[T](s: var Stack[T]): T =
  result = s.data[^1]
  s.data.setLen(s.data.len - 1)
```

The type system is strict but inference makes it feel dynamic:

```nim
# This looks dynamic but is fully type-checked at compile time
let numbers = @[1, 2, 3, 4, 5]
let doubled = numbers.map(x => x * 2)
let sum = doubled.foldl(a + b)
```

## Memory Management

Nim gives you choices:

### Garbage Collection (Default)
```nim
# Just use the language, GC handles memory
var data = @[1, 2, 3, 4, 5]
for i in 0..1000000:
  data.add(i)  # GC cleans up automatically
```

### Manual Memory Management
```nim
# Disable GC for performance-critical code
{.push checks: off.}

proc processData(data: ptr UncheckedArray[int], size: int) =
  for i in 0..<size:
    echo data[i]

# Allocate manually
var buffer = cast[ptr UncheckedArray[int]](alloc(sizeof(int) * 1000))
# Use buffer...
dealloc(buffer)  # Manual cleanup

{.pop.}
```

### Destructors and Move Semantics
```nim
type
  Resource = object
    data: string

proc `=destroy`(r: var Resource) =
  echo "Cleaning up: ", r.data
  # Custom cleanup

proc `=copy`(dest: var Resource, src: Resource) =
  dest.data = src.data

proc `=sink`(dest: var Resource, src: Resource) =
  dest.data = src.data  # Move semantics
```

Nim supports:
- Automatic Reference Counting (ARC) - modern, deterministic
- Garbage collection (GC) - traditional, automatic
- Manual management - when you need control

You can mix approaches within the same program.

## Metaprogramming: Nim's Superpower

### Templates (Compile-time code injection)
```nim
template withFile(filename: string, body: untyped) =
  let f = open(filename)
  try:
    body
  finally:
    close(f)

# Usage looks like language feature
withFile("data.txt"):
  echo f.readLine()
```

### Macros (AST manipulation)
```nim
import macros

# Macro that creates logging wrapper
macro log(procedure: untyped): untyped =
  # Manipulate AST at compile time
  result = quote do:
    proc wrapper() =
      echo "Calling function"
      `procedure`()
      echo "Function complete"
    wrapper()

log:
  echo "Doing work"

# Output:
# Calling function
# Doing work
# Function complete
```

### Compile-Time Execution
```nim
# Run code at compile time
const sizes = [1, 2, 4, 8, 16]

# Compile-time loop
const computed = block:
  var result = 0
  for s in sizes:
    result += s
  result

echo computed  # 31, computed at compile time

# Compile-time functions
proc fibonacci(n: int): int =
  if n <= 1: n
  else: fibonacci(n-1) + fibonacci(n-2)

const fib10 = fibonacci(10)  # Computed at compile time
```

This is more powerful than C++ templates and as flexible as Lisp macros.

## Uniform Function Call Syntax (UFCS)

One of Nim's best features:

```nim
# These are equivalent:
echo len("hello")
echo "hello".len()

# Chain operations naturally
let result = "hello world"
  .split(' ')
  .map(s => s.toUpper())
  .join(", ")

echo result  # "HELLO, WORLD"

# Works with any function
proc double(x: int): int = x * 2

echo double(5)    # Function call syntax
echo 5.double()   # Method call syntax
```

This makes Nim code incredibly readable without needing monkey-patching or extension methods.

## Interop: C is Your Stdlib

Nim compiles to C, so C interop is seamless:

```nim
# Import C function
proc printf(format: cstring) {.importc, varargs, header: "<stdio.h>".}

printf("Hello from C: %d\n", 42)

# Use C libraries
{.passL: "-lm".}  # Link math library

proc sin(x: float64): float64 {.importc: "sin", header: "<math.h>".}
proc cos(x: float64): float64 {.importc: "cos", header: "<math.h>".}

echo sin(3.14159 / 2)  # 1.0
```

The entire C ecosystem is available without FFI hassle.

## Real-World Example

Here's a simple HTTP server:

```nim
import asyncdispatch, asynchttpserver

# Async HTTP handler
proc handler(req: Request) {.async.} =
  case req.url.path
  of "/":
    await req.respond(Http200, "Hello, World!")
  of "/api/status":
    await req.respond(Http200, """{"status": "ok"}""",
                      newHttpHeaders([("Content-Type", "application/json")]))
  else:
    await req.respond(Http404, "Not Found")

# Start server
let server = newAsyncHttpServer()
echo "Starting server on port 8080..."
waitFor server.serve(Port(8080), handler)
```

Clean, readable, and fast. Compiles to a single native binary.

## The Good Parts

**Python-like syntax**: If you know Python, Nim feels immediately familiar. Indentation-based, minimal punctuation, readable code.

**C-level performance**: Nim compiles to C, producing executables as fast as hand-written C. No runtime overhead.

**Cross-compilation**: Compile to C, C++, Objective-C, or JavaScript. One language, multiple targets.

**Zero dependencies**: Nim binaries are self-contained. No runtime to install, no DLLs to ship.

**Powerful macros**: Build DSLs, custom control flow, and language extensions. More flexible than C++ templates.

**Great tooling**: `nimble` package manager, formatter, linter, and language server. Small ecosystem but good developer experience.

**Gradual typing**: Inference makes it feel dynamic, but everything is type-checked at compile time.

## The Pain Points

**Small ecosystem**: The package ecosystem is tiny compared to Python, JavaScript, or Rust. You'll often write things from scratch.

**Immature libraries**: Many libraries are abandoned or incomplete. The "bus factor" is high.

**Unstable syntax**: Language evolved rapidly. Old code breaks. The 1.0 release (2019) helped, but compatibility concerns remain.

**Indentation syntax**: If you hate Python's indentation, you'll hate Nim's. (Though brace syntax is technically supported.)

**Limited documentation**: Docs exist but are sparse. You'll read source code often.

**No corporate backing**: No Google, Mozilla, or Microsoft behind it. Pure community-driven. This is a strength and weakness.

**Obscurity**: Explaining Nim to recruiters or teammates is hard. "It's like Python but compiled to C" doesn't capture it.

## Use Cases

**Systems programming without suffering**: You want C performance without manual memory management nightmares or C++ complexity.

**CLI tools**: Single-binary distribution, fast startup, no runtime dependencies. Perfect for command-line tools.

**Game development**: Fast, compiles to C/C++, good for game engines or performance-critical game code.

**Embedded systems**: Small binaries, manual memory management available, compiles to C for hardware compatibility.

**Performance-critical components**: Rewrite Python bottlenecks in Nim. Use Python for glue, Nim for speed.

**Learning systems programming**: Easier entry point than C or Rust. Good concepts, gentler learning curve.

## The Ecosystem

**Package manager**: `nimble` is straightforward. Not as polished as Cargo or npm, but functional.

**Libraries**: Limited but growing. Web frameworks (Jester, Karax), async I/O, database drivers, game libraries exist. Don't expect npm-level selection.

**Tooling**:
- `nimsuggest`: Language server for IDE support
- `nimpretty`: Code formatter
- `nimgrep`: Fast code search
- Good VS Code and Vim support

**Community**: Small, friendly, passionate. Active forum and Discord. People help newcomers.

## Common Phrases You'll Hear

**"Nim compiles to C, so..."**: The answer to "but how does it achieve X?" It compiles to C, inheriting its portability and performance.

**"Use UFCS"**: Uniform Function Call Syntax makes code chainable and readable. If your code looks clunky, UFCS usually helps.

**"Macros can solve that"**: Nim's metaprogramming can implement almost any language feature you're missing. The answer is always "write a macro."

**"The ecosystem is small but..."**: Acknowledgment that library selection is limited, followed by justification why it's not a problem for specific use cases.

**"It's Python speed but C performance"**: Wait, that's backwards. **"It's Python syntax but C performance."** There we go.

## The Verdict

Nim is the underdog language for developers who value readable code and don't want to sacrifice performance. It delivers on its promise: Python-like syntax with C-level speed.

**Choose Nim if**:
- You want systems-level performance without C's complexity
- You're building CLI tools or performance-critical applications
- You like Python's syntax but need speed
- You want metaprogramming power (macros, templates, compile-time execution)
- You're willing to trade ecosystem size for performance and readability
- You enjoy supporting underdog technologies

**Avoid Nim if**:
- You need a large ecosystem and extensive libraries
- You're constrained by hiring concerns
- You want corporate backing and guaranteed longevity
- You need stability and compatibility guarantees
- You prefer braces to indentation
- You want a language everybody's heard of

Nim won't win the popularity contest. It won't dominate job boards. But for developers who've tried it, many find themselves thinking "I wish I could use Nim for this" when forced back to other languages.

It's the language equivalent of a local coffee shop: small, quirky, devoted following, and occasionally brilliant in ways the big chains can't match.

If Python's too slow and C's too painful, Nim splits the difference beautifully.

---

**Next**: [Chapter 31: Crystal - Ruby But Compiled](31-crystal.md)
