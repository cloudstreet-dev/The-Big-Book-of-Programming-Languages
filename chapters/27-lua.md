# Chapter 27: Lua - Small, Fast, Everywhere

## The Embeddable Language

Lua was created in 1993 in Brazil by Roberto Ierusalimschy, Luiz Henrique de Figueiredo, and Waldemar Celes. They needed a lightweight scripting language for extending applications, and nothing fit their needs, so they built their own.

The name "Lua" means "moon" in Portuguese. The language is small (the entire interpreter is about 280 KB), fast, and designed to be embedded in other applications. If you've played video games, used Adobe Lightroom, configured Nginx, or worked with Redis, you've encountered Lua.

By 2025, Lua is everywhere you don't see it—powering game scripts, configuration files, and embedded systems. It's the quiet language that makes other software scriptable.

## The Philosophy

Lua's design principles:

**Simplicity**: Small, clean language with minimal concepts.

**Efficiency**: Fast interpreter, small memory footprint.

**Portability**: Pure ANSI C. Runs anywhere C runs.

**Embeddability**: Designed to be embedded in applications.

**Extensibility**: Easy to call C from Lua and Lua from C.

**Powerful but simple mechanisms**: Tables implement everything.

The result is a language that does one thing exceptionally well: extending other applications.

## What It Looks Like

```lua
-- Comments start with --

-- Variables (global by default)
x = 42
name = "Alice"

-- Local variables (preferred)
local y = 100

-- Functions
function greet(name)
    print("Hello, " .. name .. "!")  -- .. concatenates strings
end

-- Functions are first-class
local greet = function(name)
    print("Hello, " .. name .. "!")
end

-- Tables (the only data structure)
local person = {
    name = "Alice",
    age = 30
}

print(person.name)  -- Alice
print(person["age"])  -- 30

-- Arrays (tables with numeric keys, 1-indexed!)
local numbers = {10, 20, 30, 40, 50}
print(numbers[1])  -- 10 (not 0!)

-- Iteration
for i = 1, 10 do
    print(i)
end

for i, v in ipairs(numbers) do
    print(i, v)  -- index, value
end

for key, value in pairs(person) do
    print(key, value)  -- key, value pairs
end

-- Conditionals
if x > 0 then
    print("Positive")
elseif x < 0 then
    print("Negative")
else
    print("Zero")
end

-- Multiple return values
function divide(a, b)
    if b == 0 then
        return nil, "division by zero"
    else
        return a / b, nil
    end
end

local result, err = divide(10, 2)
if err then
    print("Error: " .. err)
else
    print("Result: " .. result)
end

-- Closures
function makeCounter()
    local count = 0
    return function()
        count = count + 1
        return count
    end
end

local counter = makeCounter()
print(counter())  -- 1
print(counter())  -- 2

-- Metatables (for OOP and operator overloading)
local mt = {
    __add = function(a, b)
        return {x = a.x + b.x, y = a.y + b.y}
    end
}

local v1 = {x = 1, y = 2}
local v2 = {x = 3, y = 4}
setmetatable(v1, mt)
setmetatable(v2, mt)

local v3 = v1 + v2  -- Uses __add metamethod
```

Key features:
- **Tables for everything**: Arrays, dictionaries, objects
- **1-indexed**: Arrays start at 1
- **Dynamic typing**: No type declarations
- **First-class functions**: Functions are values
- **Multiple returns**: Functions can return multiple values
- **Metatables**: Customize behavior

## The Type System

Lua has eight basic types:

```lua
-- nil (absence of value)
local x = nil

-- boolean
local flag = true

-- number (double-precision float by default)
local n = 42
local pi = 3.14159

-- string
local s = "hello"
local multiline = [[
    This is a
    multiline string
]]

-- function
local f = function(x) return x * 2 end

-- userdata (C data)
-- (Used when embedding Lua in C programs)

-- thread (coroutines)
local co = coroutine.create(function()
    print("In coroutine")
end)

-- table
local t = {1, 2, 3}

-- Type checking
type(x)  -- "nil"
type(42)  -- "number"
type("hello")  -- "string"
type({})  -- "table"
type(function() end)  -- "function"

-- Type coercion
"10" + 1  -- 11 (string coerced to number)
"hello" + 1  -- Error!

-- Truthy/falsy
-- Only nil and false are falsy
-- Everything else (including 0 and "") is truthy
if 0 then print("Yes!") end  -- Prints!
if "" then print("Yes!") end  -- Prints!
```

Simple and effective for a small language.

## Tables: The Universal Data Structure

Tables are Lua's only data structure but incredibly versatile:

```lua
-- As arrays
local arr = {10, 20, 30}
print(arr[1])  -- 10

-- As dictionaries
local dict = {
    name = "Alice",
    age = 30
}

-- Mixed (not recommended but possible)
local mixed = {
    10, 20, 30,  -- Indices 1, 2, 3
    name = "Alice",  -- String key
    age = 30
}

-- Nested tables
local matrix = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
}

print(matrix[2][3])  -- 6

-- Tables as objects
local obj = {
    value = 0,
    increment = function(self)
        self.value = self.value + 1
    end
}

obj:increment()  -- Colon syntax passes obj as self
print(obj.value)  -- 1

-- Or more idiomatically:
function obj:decrement()
    self.value = self.value - 1
end

-- Table library functions
table.insert(arr, 40)  -- Append
table.remove(arr, 1)  -- Remove first
table.concat(arr, ", ")  -- Join into string
table.sort(arr)  -- Sort in place
```

Everything is a table: arrays, dictionaries, objects, modules, and more.

## Metatables: Lua's Metaprogramming

Metatables customize table behavior:

```lua
-- Operator overloading
local Vector = {}
Vector.__index = Vector

function Vector.new(x, y)
    local v = {x = x, y = y}
    setmetatable(v, Vector)
    return v
end

function Vector.__add(a, b)
    return Vector.new(a.x + b.x, a.y + b.y)
end

function Vector.__tostring(v)
    return string.format("(%d, %d)", v.x, v.y)
end

local v1 = Vector.new(1, 2)
local v2 = Vector.new(3, 4)
local v3 = v1 + v2  -- Calls __add
print(v3)  -- Calls __tostring

-- __index for missing keys
local defaults = {color = "red", size = 10}
local mt = {__index = defaults}
local obj = {}
setmetatable(obj, mt)
print(obj.color)  -- "red" (from defaults)

-- __newindex for assignment control
local mt = {
    __newindex = function(t, k, v)
        print("Setting " .. k .. " to " .. tostring(v))
        rawset(t, k, v)  -- Actually set the value
    end
}

local t = {}
setmetatable(t, mt)
t.x = 10  -- Prints: Setting x to 10

-- Other metamethods:
-- __add, __sub, __mul, __div, __mod, __pow
-- __eq, __lt, __le
-- __concat (for ..)
-- __call (make tables callable)
-- __gc (garbage collection)
```

Metatables enable powerful abstractions in a simple language.

## Coroutines: Cooperative Multitasking

Lua has first-class coroutines:

```lua
-- Create a coroutine
local co = coroutine.create(function()
    for i = 1, 3 do
        print("Coroutine iteration " .. i)
        coroutine.yield()
    end
end)

-- Resume the coroutine
coroutine.resume(co)  -- Iteration 1
coroutine.resume(co)  -- Iteration 2
coroutine.resume(co)  -- Iteration 3

-- Producer-consumer
function producer()
    return coroutine.create(function()
        for i = 1, 10 do
            coroutine.yield(i)
        end
    end)
end

local p = producer()
while true do
    local status, value = coroutine.resume(p)
    if not status then break end
    print("Received: " .. value)
end

-- Coroutines for iteration
function permutations(a)
    local n = #a
    local co = coroutine.create(function()
        -- Generate permutations and yield each
    end)
    return function()
        local status, value = coroutine.resume(co)
        return value
    end
end
```

Coroutines enable cooperative multitasking without OS threads.

## What Lua Is Best For

**Game scripting**: World of Warcraft, Roblox, Angry Birds use Lua.

**Embedded scripting**: Adobe Lightroom, VLC, Nginx, Redis.

**Configuration**: Awesome WM, Neovim, WeeChat configuration.

**Lightweight automation**: Quick scripts where Python is overkill.

**Teaching**: Simple enough for beginners, powerful enough to be useful.

**Resource-constrained environments**: Embedded systems, IoT.

## What Lua Is Worst For

**Standalone applications**: Designed to be embedded, not standalone.

**Web development**: Possible (OpenResty/Lapis) but niche. Use Node.js or Python.

**Data science**: No ecosystem. Use Python or R.

**Systems programming**: Too high-level. Use C or Rust.

**Large-scale applications**: Limited standard library. Use a more comprehensive language.

## The Ecosystem

**Standard library**: Minimal. Just the essentials.

**LuaRocks**: Package manager (like npm or pip)

**Notable packages**:
- **LuaSocket**: Networking
- **LuaFileSystem**: File system operations
- **Penlight**: Extended standard library
- **LÖVE**: 2D game framework

**Implementations**:
- **Lua**: The original (written in C)
- **LuaJIT**: Just-in-time compiler (much faster)
- **MoonScript**: Language that compiles to Lua

**Embedding APIs**: C API for embedding Lua is clean and well-designed.

The ecosystem is small because Lua is meant to be embedded, not standalone.

## LuaJIT: The Fast Implementation

LuaJIT is an alternative implementation with a JIT compiler:

```lua
-- Same syntax, much faster execution
-- Often 10-100x faster than standard Lua

-- FFI (Foreign Function Interface) for calling C
local ffi = require("ffi")

ffi.cdef[[
    int printf(const char *fmt, ...);
]]

ffi.C.printf("Hello from C!\n")
```

Many performance-critical Lua applications use LuaJIT instead of standard Lua.

## Object-Oriented Programming in Lua

Lua doesn't have classes, but you can implement them with tables and metatables:

```lua
-- Class-like behavior
local Animal = {}
Animal.__index = Animal

function Animal.new(name)
    local self = setmetatable({}, Animal)
    self.name = name
    return self
end

function Animal:speak()
    print("Some generic sound")
end

-- Inheritance
local Dog = setmetatable({}, {__index = Animal})
Dog.__index = Dog

function Dog.new(name, breed)
    local self = setmetatable(Animal.new(name), Dog)
    self.breed = breed
    return self
end

function Dog:speak()
    print("Woof! I'm " .. self.name)
end

local dog = Dog.new("Buddy", "Golden Retriever")
dog:speak()  -- Woof! I'm Buddy
```

It's not built-in OOP, but it's flexible and powerful.

## The Community

**The Game Developers**: Use Lua for scripting game logic.

**The Embedded Systems Engineers**: Embed Lua in hardware and software.

**The Power Users**: Configure everything with Lua (Neovim, Awesome WM).

**The Minimalists**: Appreciate Lua's simplicity and small footprint.

**The Performance Seekers**: Use LuaJIT for speed.

### Common Phrases
- "Tables all the way down"
- "Use LuaJIT for speed"
- "It's embedded, not standalone"
- "Small but powerful"
- "1-indexed is the one true way" (said with irony)
- "Metatables are magic"

## Why Lua Succeeded

Lua's success comes from being excellent at its niche:

1. **Small**: Entire language implementation is tiny
2. **Fast**: Especially with LuaJIT
3. **Embeddable**: Clean C API makes integration easy
4. **Portable**: Runs anywhere C runs
5. **Simple**: Easy to learn and use
6. **Licensin**g: MIT license (very permissive)

Companies embed Lua because it's small, fast, and doesn't bloat their applications.

## Lua vs Python vs JavaScript

**Lua**:
- Smallest
- Fastest (with LuaJIT)
- Best for embedding
- Smallest ecosystem

**Python**:
- Largest ecosystem
- Best for standalone scripts
- Slowest
- Heavy runtime

**JavaScript**:
- Web dominance
- Large ecosystem
- Medium size/speed
- V8 is fast but heavy

## Should You Learn Lua?

**Yes, if**:
- You're developing games (scripting)
- You're embedding a scripting language in an application
- You work with software that uses Lua (Neovim, Redis, Nginx)
- You want a simple, elegant language
- You like functional programming with minimal syntax

**No, if**:
- You need a comprehensive standard library
- You're building standalone web or data applications
- You need a large ecosystem of packages

**Maybe, if**:
- You're interested in language design
- You want to understand embeddable languages
- You configure power-user tools

## The Verdict

Lua is the little language that could. It's small, fast, and everywhere you don't notice. It powers game scripts, configuration files, and embedded systems across the industry.

Is Lua the most feature-rich language? No. The standard library is minimal. The ecosystem is small. The syntax has quirks (1-indexing, anyone?).

But Lua does one thing brilliantly: it extends other applications. The small size means it embeds easily. The clean C API means integration is straightforward. The fast execution means scripts don't slow down applications.

In 2025, Lua remains the go-to choice for embedded scripting. Games use it because it's fast and doesn't bloat the executable. Applications use it because users can customize behavior without recompiling. System administrators use it because it makes software scriptable.

Lua proves that you don't need to be big to be successful. Sometimes, being small, fast, and excellent at one thing beats being large and mediocre at many things.

Learn Lua if you're extending applications, scripting games, or curious about language minimalism. Appreciate its elegance and simplicity. Use it where it fits. Don't force it where it doesn't.

The moon language shines quietly, embedded in software everywhere, making the digital world more flexible and scriptable.

---

**Next**: [Chapter 28: C# - Microsoft's Answer to Java](28-csharp.md)
