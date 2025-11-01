# Chapter 31: Crystal - Ruby But Compiled

## Ruby's Fast Sibling

Crystal was created in 2014 by Ary Borenszweig, Juan Wajnerman, and Brian Cardiff, who loved Ruby's syntax but wanted compiled performance. The pitch: "Ruby-like syntax, C-like speed, type-safe."

The language is inspired by Ruby but adds static typing (with inference), compiles to native code, and achieves performance comparable to C. For Ruby developers frustrated by performance, Crystal offers a compelling alternative.

## The Philosophy

**Ruby-inspired syntax**: Familiar to Ruby developers.

**Static typing with inference**: Types are checked at compile time but rarely written.

**Compiled**: Native binaries, no interpreter overhead.

**Concurrent**: Built-in concurrency with fibers.

**Performance**: Fast execution, low memory usage.

## What It Looks Like

```crystal
# Ruby-like syntax
puts "Hello, World!"

# Type inference
x = 42  # Int32
name = "Alice"  # String

# Methods
def greet(name : String) : String
  "Hello, #{name}!"
end

# Type inference in methods
def add(x, y)
  x + y  # Type inferred from usage
end

# Classes
class Person
  property name : String
  property age : Int32

  def initialize(@name, @age)
  end

  def greet
    "Hello, I'm #{@name}!"
  end
end

alice = Person.new("Alice", 30)

# Blocks (like Ruby)
[1, 2, 3, 4, 5].each do |num|
  puts num * 2
end

# Or with braces
numbers.map { |x| x * x }

# Macros
macro define_method(name, content)
  def {{name}}
    {{content}}
  end
end

define_method hello, "Hello, World!"
```

## The Verdict

Crystal gives Ruby developers what they've always wanted: compiled performance without sacrificing syntax elegance. It's type-safe, fast, and looks like Ruby.

Perfect for web services, CLI tools, and anywhere Ruby is too slow. The ecosystem is growing but still small. For Ruby lovers seeking performance, Crystal is the answer.

---

**Next**: [Chapter 32: Racket - The Language for Making Languages](32-racket.md)
