# Chapter 30: Nim - Python-like Syntax, C-like Speed

## The Best of Both Worlds

Nim (formerly Nimrod) was created in 2008 by Andreas Rumpf, who wanted a language that combined Python's elegant syntax with C's performance and low-level control. The goal was ambitious: readable, expressive code that compiles to efficient executables.

By 2025, Nim remains a niche language but has a devoted following among developers who want systems programming capabilities without C's complexity or Rust's steep learning curve. It's the language for people who think "I wish Python were fast and compiled."

## The Philosophy

**Efficiency**: Compile to C/C++/JavaScript. Performance comparable to C.

**Expressiveness**: Python-like syntax. Significant indentation.

**Elegance**: Clean, readable code. Minimal punctuation.

**Metaprogramming**: Powerful macro system. AST manipulation.

**Flexibility**: Multiple paradigms. Imperative, OOP, functional.

The result is a language that feels like Python but runs like C.

## What It Looks Like

```nim
# Simple and Python-like
echo "Hello, World!"

# Variables
var x = 42
let y = 100  # Immutable
const Pi = 3.14159

# Functions
proc greet(name: string): void =
  echo "Hello, ", name, "!"

# Type inference
proc add(x, y: int): int =
  x + y

# Collections
var numbers = @[1, 2, 3, 4, 5]  # Seq (like Python list)
var squares = numbers.map(proc(x: int): int = x * x)

# Control flow
if x > 0:
  echo "Positive"
elif x < 0:
  echo "Negative"
else:
  echo "Zero"

# Loops
for i in 1..10:
  echo i

for item in numbers:
  echo item

# Objects
type
  Person = object
    name: string
    age: int

var alice = Person(name: "Alice", age: 30)

# Generics
proc identity[T](x: T): T =
  x
```

Nim looks like Python but compiles to native code.

## The Verdict (Brief)

Nim is the underdog language for people who want Python's syntax with C's performance. It compiles to C, runs fast, has powerful metaprogramming, and offers low-level control.

It's perfect for systems programming without C's complexity, game development, embedded systems, and performance-critical applications where Python is too slow.

The ecosystem is small, the community is niche, but for developers who value readable code that compiles to fast binaries, Nim delivers.

---

**Next**: [Chapter 31: Crystal - Ruby But Compiled](31-crystal.md)
