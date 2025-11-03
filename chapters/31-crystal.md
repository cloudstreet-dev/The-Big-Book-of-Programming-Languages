# Chapter 31: Crystal - Ruby But Compiled

## Ruby's Performance-Conscious Sibling

Crystal was created in 2014 by Ary Borenszweig, Juan Wajnerman, and Brian Cardiff in Argentina. Their motivation was simple: they loved Ruby but couldn't ignore its performance problems. Ruby was elegant, expressive, and slow. Very slow.

The question: Could you keep Ruby's beautiful syntax while achieving C-level performance?

Crystal's answer: Yes, but you need static typing. The compromise: Type inference so aggressive you rarely write type annotations. The code looks like Ruby, reads like Ruby, feels like Ruby—but compiles to fast native executables.

By 2025, Crystal remains niche but has a devoted following. It's the language for Ruby developers who've hit performance walls and aren't ready to learn Rust or Go.

## The Philosophy

**Ruby-inspired syntax**: Crystal intentionally mimics Ruby. The goal was making Ruby developers feel at home. Methods, blocks, string interpolation—all look familiar.

**Static typing with inference**: Types are checked at compile time, preventing entire classes of bugs. But you rarely write type annotations—the compiler infers them. Feel like Ruby, safety of static typing.

**Compiled to native code**: Crystal uses LLVM to compile to standalone executables. No interpreter, no VM, no runtime. Just your code and the compiled binary.

**Concurrent by default**: Built-in concurrency with fibers (lightweight threads). Write concurrent code without fear of global interpreter locks or threading nightmares.

**Performance without compromise**: Speed comparable to C and Go. Memory efficient. Fast startup. Everything Ruby isn't.

**Nil-safe**: Crystal learned from Kotlin and Swift. Nil is not assignable to regular types. The compiler forces you to handle nil cases explicitly.

The result: A language that looks like Ruby but performs like C, with modern safety features Ruby never had.

## What It Looks Like

### Basic Syntax

```crystal
# Hello World - looks exactly like Ruby
puts "Hello, World!"

# Variables with type inference
x = 42                        # Int32
name = "Alice"                # String
price = 19.99                 # Float64
active = true                 # Bool

# Type annotations (optional but sometimes needed)
age : Int32 = 25
names : Array(String) = ["Alice", "Bob"]

# String interpolation (like Ruby)
puts "Hello, #{name}!"
puts "Age: #{age}, Price: #{price}"

# Methods
def greet(name : String) : String
  "Hello, #{name}!"
end

# Type inference in methods
def add(x, y)
  x + y  # Return type inferred
end

puts add(2, 3)         # 5
puts add(2.5, 3.5)     # 6.0

# Multiple dispatch based on types
def process(x : Int32)
  "Processing integer: #{x}"
end

def process(x : String)
  "Processing string: #{x}"
end

puts process(42)       # "Processing integer: 42"
puts process("hello")  # "Processing string: hello"
```

### Collections

```crystal
# Arrays
numbers = [1, 2, 3, 4, 5]
strings = ["hello", "world"]

# Array methods (like Ruby)
doubled = numbers.map { |x| x * 2 }
evens = numbers.select { |x| x.even? }
sum = numbers.sum

# Blocks with do...end or braces
[1, 2, 3].each do |num|
  puts num * 2
end

squares = [1, 2, 3].map { |x| x ** 2 }

# Hashes (dictionaries)
person = {
  "name" => "Alice",
  "age" => 30,
  "city" => "NYC"
}

# Symbol keys (like Ruby)
config = {
  host: "localhost",
  port: 8080,
  debug: true
}

puts config[:host]     # "localhost"

# Tuples (immutable, heterogeneous)
point = {10, 20}
person_tuple = {"Alice", 30, true}

x, y = point           # Destructuring
```

### Classes and Objects

```crystal
class Person
  property name : String
  property age : Int32

  # @name, @age are instance variables
  def initialize(@name : String, @age : Int32)
  end

  def greet
    "Hello, I'm #{@name} and I'm #{@age} years old"
  end

  def older_than?(other : Person)
    @age > other.age
  end
end

alice = Person.new("Alice", 30)
bob = Person.new("Bob", 25)

puts alice.greet
puts alice.older_than?(bob)  # true

# Inheritance
class Employee < Person
  property salary : Float64

  def initialize(name : String, age : Int32, @salary : Float64)
    super(name, age)
  end

  def annual_salary
    @salary * 12
  end
end

# Modules (mixins, like Ruby)
module Walkable
  def walk
    "#{name} is walking"
  end
end

class Animal
  include Walkable
  property name : String

  def initialize(@name)
  end
end

dog = Animal.new("Rex")
puts dog.walk  # "Rex is walking"
```

### Nil Safety

```crystal
# Nil is its own type
name : String = "Alice"        # OK
# name : String = nil          # ERROR: Can't assign Nil to String

# Nilable types use union syntax
nullable_name : String | Nil = nil
nullable_name = "Alice"        # OK

# Or use shorthand
optional_age : Int32? = nil    # Same as Int32 | Nil

# Must handle nil explicitly
def greet(name : String?)
  if name
    # Inside this block, name is String (not String?)
    "Hello, #{name}!"
  else
    "Hello, stranger!"
  end
end

# Safe navigation operator
person = get_person?  # Returns Person | Nil
puts person.try(&.name) || "Unknown"

# Nil coalescing
default_name = optional_name || "Anonymous"
```

## Type System

Crystal is statically typed with powerful inference:

```crystal
# The compiler figures out types
def process(items)
  items.map { |x| x * 2 }
end

# When calling process([1, 2, 3]), compiler knows:
# - items is Array(Int32)
# - x is Int32
# - Result is Array(Int32)

# Union types
def flexible(value : Int32 | String)
  case value
  when Int32
    value * 2     # value is Int32 here
  when String
    value.upcase  # value is String here
  end
end

# Type restrictions in generics
class Box(T)
  def initialize(@value : T)
  end

  def get : T
    @value
  end
end

int_box = Box.new(42)
string_box = Box.new("hello")

# Type aliases
alias ID = Int32 | String
alias Point = {x: Int32, y: Int32}

user_id : ID = 12345
origin : Point = {x: 0, y: 0}

# Abstract classes and methods
abstract class Animal
  abstract def sound : String

  def speak
    puts sound
  end
end

class Dog < Animal
  def sound : String
    "Woof!"
  end
end

class Cat < Animal
  def sound : String
    "Meow!"
  end
end
```

## Concurrency: Fibers and Channels

Crystal uses fibers (lightweight, cooperative threads):

```crystal
# Spawn a fiber (like goroutine in Go)
spawn do
  puts "Hello from fiber!"
  sleep 1
  puts "Fiber done"
end

puts "Main continues"
sleep 2  # Give fibers time to complete

# Channels for communication (like Go)
channel = Channel(Int32).new

spawn do
  5.times do |i|
    channel.send(i)
    sleep 0.1
  end
  channel.close
end

# Receive from channel
while value = channel.receive?
  puts "Received: #{value}"
end

# Parallel processing
results = Channel(Int32).new

# Spawn multiple fibers
10.times do |i|
  spawn do
    result = expensive_computation(i)
    results.send(result)
  end
end

# Collect results
10.times do
  puts results.receive
end

# Select (like Go)
ch1 = Channel(String).new
ch2 = Channel(String).new

spawn { ch1.send("from ch1") }
spawn { ch2.send("from ch2") }

select
when msg = ch1.receive
  puts "Got #{msg}"
when msg = ch2.receive
  puts "Got #{msg}"
when timeout(1.second)
  puts "Timeout!"
end
```

No global interpreter lock. True concurrency. Channels for safe communication.

## Macros: Compile-Time Metaprogramming

```crystal
# Macros execute at compile time
macro define_property(name)
  def {{name}}
    @{{name}}
  end

  def {{name}}=(value)
    @{{name}} = value
  end
end

class Person
  define_property name
  define_property age
end

# DSL example
macro assert(condition)
  if !{{condition}}
    raise "Assertion failed: {{condition}}"
  end
end

assert 2 + 2 == 4  # OK
# assert 2 + 2 == 5  # Raises: Assertion failed: 2 + 2 == 5

# Generate methods programmatically
macro generate_math_methods
  {% for op in [:add, :subtract, :multiply, :divide] %}
    def {{op.id}}(x, y)
      {% if op == :add %}
        x + y
      {% elsif op == :subtract %}
        x - y
      {% elsif op == :multiply %}
        x * y
      {% elsif op == :divide %}
        x / y
      {% end %}
    end
  {% end %}
end

generate_math_methods

puts add(5, 3)       # 8
puts multiply(4, 7)  # 28
```

Macros are more powerful than Ruby's metaprogramming but happen at compile time.

## Real-World Example: HTTP Server

```crystal
require "http/server"

# Define routes
server = HTTP::Server.new do |context|
  case {context.request.method, context.request.path}
  when {"GET", "/"}
    context.response.content_type = "text/plain"
    context.response.print "Hello, World!"

  when {"GET", "/api/users"}
    context.response.content_type = "application/json"
    context.response.print %([{"name": "Alice", "age": 30}])

  when {"POST", "/api/users"}
    # Parse request body
    data = context.request.body.try(&.gets_to_end) || "{}"
    context.response.status_code = 201
    context.response.print "Created"

  else
    context.response.status_code = 404
    context.response.print "Not Found"
  end
end

address = server.bind_tcp "localhost", 8080
puts "Listening on http://#{address}"
server.listen
```

Fast, concurrent, single binary. No runtime to install.

## Memory Management

Crystal uses automatic memory management with garbage collection:

```crystal
# Garbage collected (like Ruby)
1_000_000.times do
  data = Array.new(100) { rand }
  # GC cleans up when data goes out of scope
end

# Structs are stack-allocated (value types)
struct Point
  property x : Int32
  property y : Int32

  def initialize(@x, @y)
  end
end

# Point instances live on stack, no GC overhead
point = Point.new(10, 20)

# Classes are heap-allocated (reference types)
class Person
  property name : String
  def initialize(@name)
  end
end

# Person instances are heap-allocated, GC-managed
person = Person.new("Alice")
```

The GC is efficient but can pause. No manual memory management like Rust.

## The Good Parts

**Ruby syntax, C performance**: Code looks elegant, runs fast. Best of both worlds for Ruby developers.

**Strong static typing with inference**: Safety without verbosity. Compiler catches errors, you rarely write types.

**Nil safety**: No more "undefined method for nil:NilClass" runtime errors. Compiler enforces nil handling.

**Built-in concurrency**: Fibers and channels make concurrent programming straightforward. No GIL.

**Single binary deployment**: Compile to standalone executable. No dependencies, no runtime to install.

**Great for web services**: Fast HTTP servers, efficient memory usage, good concurrency. Perfect for APIs and microservices.

**C interop**: Can call C libraries directly. Access to massive C ecosystem.

**Fast compilation**: Faster than Rust (though slower than Go).

## The Pain Points

**Small ecosystem**: Package ecosystem (shards) is tiny compared to Ruby, Python, or JavaScript. Many things you'll build yourself.

**Pre-1.0 instability**: Crystal reached 1.0 in 2021, but breaking changes still happen. Libraries may break between versions.

**Slow compiler**: Faster than Rust but slower than Go. Full rebuilds can take time.

**Limited Windows support**: Historically weak on Windows, though improving. Linux and macOS are primary platforms.

**Compilation complexity**: Some Ruby patterns don't translate well. Dynamic metaprogramming becomes compile-time macros.

**Small community**: Fewer resources, fewer libraries, fewer job opportunities.

**No incremental compilation**: Whole-program compilation means slow feedback loops on large projects.

**Breaking changes**: Still evolving. Code that worked in version N might break in N+1.

## Use Cases

**Web APIs and microservices**: Fast, concurrent, low memory. Perfect for HTTP services. Frameworks like Kemal make this easy.

**CLI tools**: Single-binary distribution, fast startup, no runtime. Great for command-line applications.

**DevOps tools**: Like Go but with Ruby syntax. Build deployment tools, monitoring, automation.

**Performance-critical Ruby replacements**: Rewrite Ruby bottlenecks in Crystal. Keep Ruby for business logic, use Crystal for speed.

**Systems programming (light)**: Not as low-level as Rust or C, but more accessible. Good for network tools, data processing.

**Learning static typing**: Ruby developers wanting to learn static types without jumping to dramatically different syntax.

## The Ecosystem

**Package manager**: Shards (like Bundler). Functional but ecosystem is small.

**Web frameworks**:
- Kemal (Sinatra-like)
- Lucky (full-stack, Rails-like)
- Amber (MVC framework)

**Libraries**: HTTP clients, JSON, databases (PostgreSQL, MySQL, Redis), testing frameworks. Coverage is basic.

**Tooling**:
- `crystal` compiler and build tool
- `shards` package manager
- Language server for IDE support
- Good VS Code support

**Community**: Small, friendly, active. Growing slowly but steadily.

## Common Phrases You'll Hear

**"It's like Ruby but fast"**: The elevator pitch. Captures 80% of what Crystal is.

**"Write Ruby, run C"**: The compiler does the heavy lifting. You write elegant code, get performance.

**"Nil-safe by design"**: Unlike Ruby, Crystal won't let nil crash your program at runtime.

**"True concurrency without GIL"**: Fibers give you real parallelism. No global interpreter lock holding you back.

**"The ecosystem is small, but..."**: Standard caveat. Small ecosystem, but what exists is quality.

**"It's pre-1.0, so..."**: Used to explain breaking changes. Actually hit 1.0 in 2021, but the phrase persists.

## The Verdict

Crystal delivers on its promise: Ruby's elegance with compiled performance. For Ruby developers frustrated by speed, it's a natural evolution.

**Choose Crystal if**:
- You love Ruby syntax but need performance
- You're building web services, APIs, or CLI tools
- You want static typing but hate verbosity
- You need true concurrency (no GIL)
- You want single-binary deployment
- You're willing to work with a small ecosystem

**Avoid Crystal if**:
- You need a mature, stable ecosystem
- Windows support is critical
- You need maximum performance (use Rust or C++)
- You want a huge job market
- You need mature tooling and extensive libraries
- Breaking changes between versions would be problematic

Crystal won't replace Ruby for web applications (Rails is too dominant). It won't dethrone Go for cloud tools (too established). It won't beat Rust for systems (not as safe or fast).

But for Ruby developers who've hit performance walls and want familiar syntax with compiled speed, Crystal is exactly what they've been asking for.

It's Ruby's performance-conscious sibling: same family resemblance, much faster metabolism.

If you find yourself writing Ruby and thinking "I wish this were faster," Crystal is worth the afternoon it takes to learn.

---

**Next**: [Chapter 32: Racket - The Language for Making Languages](32-racket.md)
