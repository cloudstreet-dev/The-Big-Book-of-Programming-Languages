# Chapter 21: F# - Functional .NET Done Right

## Microsoft's Functional Secret Weapon

F# (pronounced "F sharp") was created by Don Syme at Microsoft Research in 2005, inspired by OCaml. It's a functional-first language that runs on .NET, giving you functional programming with access to the entire .NET ecosystem.

Unlike C# (Microsoft's flagship language), F# embraces immutability, type inference, and functional patterns. It's proof that Microsoft understands the value of functional programming, even if they don't market it as heavily as C#.

By 2025, F# is used in finance, data science, web development, and anywhere .NET developers want functional programming's benefits without leaving the .NET ecosystem.

## The Philosophy

F#'s design principles:

**Functional-first**: Functional is the default, imperative is opt-in.

**Concise**: Type inference and expressive syntax reduce boilerplate.

**Interoperable**: Seamless .NET integration. Call C# libraries, be called from C#.

**Practical**: Solve real problems. Not academic purity.

**Immutable by default**: Values don't change unless you explicitly choose otherwise.

**Strong typing**: Type inference means you get type safety without verbosity.

The result is OCaml's elegance on the .NET platform.

## What It Looks Like

```fsharp
// Simple functions
let add x y = x + y

let square x = x * x

// Type inference works
let numbers = [1; 2; 3; 4; 5]

let doubled = List.map (fun x -> x * 2) numbers

// Pattern matching
let rec factorial n =
    match n with
    | 0 -> 1
    | n -> n * factorial (n - 1)

// Discriminated unions (algebraic data types)
type Shape =
    | Circle of radius: float
    | Rectangle of width: float * height: float
    | Triangle of a: float * b: float * c: float

let area shape =
    match shape with
    | Circle r -> System.Math.PI * r * r
    | Rectangle (w, h) -> w * h
    | Triangle (a, b, c) ->
        let s = (a + b + c) / 2.0
        sqrt (s * (s - a) * (s - b) * (s - c))

// Option type (no null!)
type Option<'T> =
    | Some of 'T
    | None

let safeDivide x y =
    if y = 0.0 then None
    else Some (x / y)

// Records
type Person = {
    Name: string
    Age: int
}

let alice = { Name = "Alice"; Age = 30 }

// Pipe operator
[1; 2; 3; 4; 5]
|> List.map ((*) 2)
|> List.filter (fun x -> x > 5)
|> List.sum

// Async workflows
async {
    let! data = fetchDataAsync()
    let! result = processAsync data
    return result
}

// Computation expressions
seq {
    for i in 1 .. 10 do
        yield i * i
}
```

Key features:
- **Type inference**: Types are inferred, annotations are optional
- **Immutability**: Default behavior
- **Pattern matching**: Exhaustive and powerful
- **Pipe operator**: Data transformation pipelines
- **No null**: Option types enforce null handling

## The Type System

F# has a strong, inferred type system:

```fsharp
// Basic types - all inferred
let x = 42              // int
let y = 3.14            // float
let s = "hello"         // string
let b = true            // bool

// Explicit type annotations (optional)
let add (x: int) (y: int): int = x + y

// Lists
let numbers = [1; 2; 3; 4; 5]  // int list

// Arrays
let array = [| 1; 2; 3 |]      // int[]

// Tuples
let pair = (1, "hello")         // int * string

// Records
type Point = {
    X: float
    Y: float
}

let origin = { X = 0.0; Y = 0.0 }

// Discriminated unions
type Result<'T, 'E> =
    | Ok of 'T
    | Error of 'E

let divide x y =
    if y = 0.0 then
        Error "Division by zero"
    else
        Ok (x / y)

// Generic types
type Tree<'T> =
    | Leaf
    | Node of 'T * Tree<'T> * Tree<'T>

// Type aliases
type Name = string
type Age = int

// Units of measure (unique to F#!)
[<Measure>] type kg
[<Measure>] type m
[<Measure>] type s

let mass = 10.0<kg>
let distance = 5.0<m>
let time = 2.0<s>
let velocity = distance / time  // float<m/s>

// Compile error: mass + distance  // Can't add kg to m!
```

Units of measure prevent unit conversion errors at compile time—a unique F# feature.

## Pattern Matching: Comprehensive

F# pattern matching is extensive:

```fsharp
// Basic matching
let describe n =
    match n with
    | 0 -> "zero"
    | 1 -> "one"
    | 2 -> "two"
    | _ -> "many"

// List matching
let rec sum list =
    match list with
    | [] -> 0
    | head :: tail -> head + sum tail

// Tuple matching
let classify point =
    match point with
    | (0, 0) -> "origin"
    | (x, 0) -> sprintf "on x-axis at %d" x
    | (0, y) -> sprintf "on y-axis at %d" y
    | (x, y) -> sprintf "at (%d, %d)" x y

// Record matching
let greet person =
    match person with
    | { Name = "Alice" } -> "Hi Alice!"
    | { Age = age } when age < 18 -> "Hello, young one!"
    | { Name = name } -> sprintf "Hello, %s!" name

// Discriminated union matching
let processResult result =
    match result with
    | Ok value -> printfn "Success: %A" value
    | Error msg -> printfn "Error: %s" msg

// Active patterns (custom patterns)
let (|Even|Odd|) n =
    if n % 2 = 0 then Even else Odd

let checkParity n =
    match n with
    | Even -> "even"
    | Odd -> "odd"

// Partial active patterns
let (|Integer|_|) (str: string) =
    match System.Int32.TryParse(str) with
    | (true, int) -> Some int
    | _ -> None

match "42" with
| Integer i -> printfn "Got integer: %d" i
| _ -> printfn "Not an integer"
```

Pattern matching makes code declarative and exhaustive.

## Computation Expressions: Powerful Syntax

Computation expressions are F#'s abstraction for workflows:

```fsharp
// Async computation expression
let fetchAndProcess url = async {
    let! html = fetchAsync url
    let! data = parseAsync html
    return processData data
}

// Seq computation expression
let squares = seq {
    for i in 1 .. 100 do
        yield i * i
}

// Option computation expression (with custom builder)
type OptionBuilder() =
    member __.Bind(opt, f) =
        match opt with
        | Some x -> f x
        | None -> None

    member __.Return(x) = Some x

let option = OptionBuilder()

let safeDivideChain a b c = option {
    let! x = safeDivide a b
    let! y = safeDivide x c
    return y
}

// List computation expression
let pairs = [
    for x in 1 .. 5 do
        for y in 1 .. 5 do
            yield (x, y)
]

// Result computation expression
type ResultBuilder() =
    member __.Bind(result, f) =
        match result with
        | Ok x -> f x
        | Error e -> Error e

    member __.Return(x) = Ok x

let result = ResultBuilder()

let processChain = result {
    let! x = step1()
    let! y = step2 x
    let! z = step3 y
    return z
}
```

Computation expressions abstract away boilerplate for monadic operations.

## Immutability and Functional Style

F# defaults to immutability:

```fsharp
// Immutable by default
let x = 42
// x <- 43  // Compile error!

// Mutable (opt-in)
let mutable counter = 0
counter <- counter + 1

// Immutable records
type Person = { Name: string; Age: int }

let alice = { Name = "Alice"; Age = 30 }

// "Updating" creates a new record
let olderAlice = { alice with Age = 31 }
// alice is unchanged

// Immutable collections
let list1 = [1; 2; 3]
let list2 = 4 :: list1  // [4; 1; 2; 3]
// list1 is unchanged

// Functional data transformation
let processData data =
    data
    |> List.filter (fun x -> x > 0)
    |> List.map ((*) 2)
    |> List.sort
    |> List.take 10

// Recursion over loops
let rec sumList list =
    match list with
    | [] -> 0
    | head :: tail -> head + sumList tail

// Tail recursion (optimized)
let sumListTail list =
    let rec loop acc list =
        match list with
        | [] -> acc
        | head :: tail -> loop (acc + head) tail
    loop 0 list
```

Immutability prevents bugs and enables safe parallelism.

## .NET Interop

F# has seamless .NET integration:

```fsharp
// Using .NET libraries
open System
open System.IO
open System.Net.Http

// Call C# code
let now = DateTime.Now
let files = Directory.GetFiles("C:\\Data")

// HTTP request
let fetchUrl url = async {
    use client = new HttpClient()
    let! response = client.GetStringAsync(url) |> Async.AwaitTask
    return response
}

// LINQ (but F# has better alternatives)
open System.Linq

let query =
    [1; 2; 3; 4; 5]
    |> Seq.where (fun x -> x % 2 = 0)
    |> Seq.select (fun x -> x * 2)

// Calling from C#
// F# module:
module Math =
    let add x y = x + y

// C#:
// var result = Math.add(5, 3);

// F# class for C# interop
type Person(name: string, age: int) =
    member _.Name = name
    member _.Age = age
    member _.Greet() = sprintf "Hello, I'm %s" name
```

F# and C# work together seamlessly in the same solution.

## What F# Is Best For

**Financial modeling**: Type safety and immutability prevent costly bugs.

**Data science**: Type providers make data access easy.

**Web services**: Giraffe, Saturn, and other frameworks.

**Domain modeling**: Discriminated unions model domains well.

**Concurrent programming**: Immutability makes parallelism safe.

**Anywhere .NET is used**: F# is a .NET language.

## What F# Is Worst For

**Game development**: Unity uses C#. Unreal uses C++.

**Mobile apps (without .NET MAUI)**: Swift and Kotlin are preferred.

**Frontend web**: JavaScript/TypeScript dominate.

**Systems programming**: Use C, Rust, or C++.

**Teams unfamiliar with functional programming**: Steep learning curve.

## The Ecosystem

**Build tools**: .NET SDK, MSBuild, FAKE (F# build tool)

**Package management**: NuGet (same as C#)

**Web frameworks**:
- **Giraffe**: Functional web framework
- **Saturn**: Rails-like framework for F#
- **Fable**: F# to JavaScript compiler (like ClojureScript)

**Data science**: Type providers, FsLab

**Testing**: Expecto, FsUnit, Unquote

**IDEs**: Visual Studio, VS Code (with Ionide), JetBrains Rider

**Type providers**: Query databases, JSON, XML, CSV as if they were types

The ecosystem is smaller than C# but growing.

## Type Providers: Compile-Time Magic

Type providers are an F# superpower:

```fsharp
// JSON type provider
open FSharp.Data

type Weather = JsonProvider<"https://api.weather.com/sample.json">

let getWeather city =
    let data = Weather.Load(sprintf "https://api.weather.com/%s" city)
    data.Temperature

// SQL type provider
open FSharp.Data.Sql

type Sql = SqlDataProvider<
    ConnectionString = "...",
    DatabaseVendor = Common.DatabaseProviderTypes.MSSQLSERVER>

let ctx = Sql.GetDataContext()

let customers =
    query {
        for customer in ctx.Dbo.Customers do
        where (customer.Age > 18)
        select customer.Name
    }

// CSV type provider
type MyCsv = CsvProvider<"data.csv">

let data = MyCsv.Load("newdata.csv")
for row in data.Rows do
    printfn "%s is %d years old" row.Name row.Age
```

Type providers give you compile-time intellisense and type-safety for external data.

## The Community

**The Functional Enthusiasts**: Love functional programming, chose F# for .NET.

**The C# Refugees**: Escaped C#'s verbosity and null references.

**The Finance Developers**: Use F# for trading, risk modeling, quant work.

**The Data Scientists**: Use type providers and FsLab for data work.

**The Pragmatists**: Want functional programming without leaving .NET.

### Common Phrases
- "No null references!"
- "Type providers are magic"
- "It's like OCaml on .NET"
- "Pattern matching makes it obvious"
- "Immutability by default"
- "Less code, fewer bugs"

## F# vs C#

The comparison within the .NET ecosystem:

**C#**:
- Object-oriented focus
- Nullable references (recently added null safety)
- More verbose
- Larger community and ecosystem
- Microsoft's primary .NET language

**F#**:
- Functional-first
- No null (option types)
- More concise
- Smaller but dedicated community
- Better for certain domains (finance, data)

**When to choose F#**: You value conciseness, immutability, and functional programming.

**When to choose C#**: Your team knows C#, you need maximum ecosystem, or you're building games/Unity.

Both are excellent. Both run on .NET. They interoperate seamlessly.

## Should You Learn F#?

**Yes, if**:
- You work in the .NET ecosystem
- You want functional programming without leaving .NET
- You're in finance or data science
- You value type safety and conciseness
- You want to understand functional programming

**No, if**:
- You're not using .NET
- Your team is committed to C# and won't adopt F#
- You prefer object-oriented programming
- You need the absolute largest ecosystem

**Maybe, if**:
- You know C# and want to level up
- You're curious about functional .NET
- You want to see how functional and OOP can coexist

## The Verdict

F# is Microsoft's best-kept secret. It's a beautiful functional language that runs on one of the industry's most mature platforms. It gives you OCaml's elegance with .NET's libraries, tooling, and ecosystem.

Is F# popular? Not like C# or Python. But it's used successfully in production at financial firms, data-heavy companies, and by developers who value correctness and conciseness.

F# proves that functional programming and the .NET platform are compatible. You can write elegant, concise, type-safe code and still call into decades of .NET libraries. You can interop with C# codebases. You can use Visual Studio or VS Code.

The type system is excellent. Units of measure prevent scientific computation errors. Type providers make working with external data delightful. Pattern matching makes code declarative. Immutability prevents entire classes of bugs.

If you're in the .NET world, F# is worth learning. It will change how you think about C#. It will show you that functional programming is practical, not academic. It will make you appreciate types, immutability, and expressions over statements.

F# won't replace C# as Microsoft's flagship language. But for developers who understand its strengths, F# is indispensable. It's the right tool for the right job—and that job often involves complex domains, data transformation, or systems where correctness matters more than familiarity.

Functional .NET done right. That's F#.

---

**Next**: [Chapter 22: Scala - The Kitchen Sink of Type Systems](22-scala.md)
