# Chapter 15: Swift - Not Just for Apple Anymore

## Apple's Modern Language

Swift was announced by Apple at WWDC 2014 and released as open source in 2015. Created by Chris Lattner (who also created LLVM and Clang), Swift was designed to replace Objective-C for Apple platform development. The goal: a modern, safe, fast language that's pleasant to write.

Swift succeeded. By 2025, it's the dominant language for iOS, macOS, watchOS, and tvOS development. It's also expanding beyond Apple: Swift runs on Linux, Windows, and is used for server-side development. It's not just for Apple anymore, though that's still its primary home.

Swift feels like a blend of modern ideas: type safety from Haskell, protocols from Go/Rust, optionals from functional languages, and syntax that's clean and readable. It's a well-designed language that learned from decades of programming language research.

## The Philosophy

Swift's design principles:

**Safety**: Eliminate entire classes of bugs at compile time. No null pointer crashes.

**Speed**: Performance comparable to C and C++. Modern and fast.

**Expressiveness**: Code should be clear and concise. Type inference reduces boilerplate.

**Modernity**: Learn from past languages. Include the best features.

**Progressive disclosure**: Simple things should be simple, complex things should be possible.

**Open source**: Not locked to Apple (though primarily used there).

The result is a language that feels approachable for beginners but powerful for experts.

## What It Looks Like

```swift
// Simple and clean
var name = "Alice"
let age = 30  // let = immutable, var = mutable

print("Hello, \(name)!")  // String interpolation

// Functions
func greet(name: String) -> String {
    return "Hello, \(name)!"
}

// Shortened
func greet(name: String) -> String { "Hello, \(name)!" }

// Optionals (nullable types)
var optionalName: String? = "Bob"
optionalName = nil

// Unwrapping optionals
if let name = optionalName {
    print("Hello, \(name)")
}

// Guard statement
func process(name: String?) {
    guard let name = name else {
        return
    }
    print("Processing \(name)")
}

// Nil coalescing
let displayName = optionalName ?? "Guest"

// Structs (value types)
struct Point {
    var x: Double
    var y: Double

    func distance(to other: Point) -> Double {
        let dx = x - other.x
        let dy = y - other.y
        return (dx * dx + dy * dy).squareRoot()
    }
}

// Classes (reference types)
class Person {
    var name: String
    var age: Int

    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }

    func greet() -> String {
        "Hello, I'm \(name)"
    }
}

// Enums with associated values
enum Result {
    case success(String)
    case failure(Error)
}

// Pattern matching
switch result {
case .success(let data):
    print("Got data: \(data)")
case .failure(let error):
    print("Error: \(error)")
}

// Closures (lambdas)
let numbers = [1, 2, 3, 4, 5]
let doubled = numbers.map { $0 * 2 }
let evens = numbers.filter { $0 % 2 == 0 }
```

Key features:
- **Type inference**: Types are inferred but statically checked
- **Optionals**: Built-in null safety
- **Pattern matching**: Powerful switch statements
- **Value types**: Structs are preferred over classes
- **Protocol-oriented**: Composition over inheritance

## The Type System

Swift has a sophisticated type system:

```swift
// Basic types
let integer: Int = 42
let floating: Double = 3.14
let string: String = "Hello"
let boolean: Bool = true

// Type inference
let inferredInt = 42       // Int
let inferredDouble = 3.14  // Double

// Collections
let array: [Int] = [1, 2, 3]
let dictionary: [String: Int] = ["a": 1, "b": 2]
let set: Set<Int> = [1, 2, 3]

// Tuples
let coordinate = (x: 10, y: 20)
print(coordinate.x)  // 10

// Type aliases
typealias Coordinate = (x: Int, y: Int)

// Generics
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

// Generic types
struct Stack<Element> {
    private var items: [Element] = []

    mutating func push(_ item: Element) {
        items.append(item)
    }

    mutating func pop() -> Element? {
        return items.popLast()
    }
}

// Associated types (in protocols)
protocol Container {
    associatedtype Item
    mutating func add(_ item: Item)
    var count: Int { get }
}
```

The type system is expressive and catches errors at compile time.

## Optionals: Null Safety Done Right

Swift eliminated null pointer crashes with optionals:

```swift
// Optional declaration
var name: String? = "Alice"
name = nil  // OK

var nonOptional: String = "Bob"
// nonOptional = nil  // Compile error!

// Optional binding (if let)
if let unwrappedName = name {
    print("Hello, \(unwrappedName)")
} else {
    print("No name")
}

// Guard statement (early exit)
func greet(name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    print("Hello, \(name)")
}

// Optional chaining
let length = name?.count  // Optional<Int>

class Person {
    var residence: Residence?
}

class Residence {
    var address: Address?
}

class Address {
    var street: String?
}

// Chain optionals
let street = person.residence?.address?.street

// Nil coalescing operator
let displayName = name ?? "Guest"

// Force unwrapping (use sparingly!)
let forcedName = name!  // Crashes if nil

// Implicitly unwrapped optionals
var assumedString: String! = "Hello"
// Treated as optional but can be used without unwrapping
```

Optionals make null handling explicit and safe.

## Value Types vs Reference Types

Swift emphasizes value types (structs) over reference types (classes):

```swift
// Struct (value type - copied on assignment)
struct Point {
    var x: Int
    var y: Int
}

var point1 = Point(x: 0, y: 0)
var point2 = point1  // Copy
point2.x = 10
print(point1.x)  // 0 (unchanged)

// Class (reference type - shared reference)
class Person {
    var name: String
    init(name: String) { self.name = name }
}

let person1 = Person(name: "Alice")
let person2 = person1  // Reference
person2.name = "Bob"
print(person1.name)  // "Bob" (changed!)

// When to use what:
// - Structs: Data models, small objects, value semantics
// - Classes: Shared mutable state, inheritance needed
```

Value semantics avoid many concurrency bugs and make code easier to reason about.

## Protocols: Swift's Interfaces

Protocols define contracts:

```swift
// Basic protocol
protocol Drawable {
    func draw()
}

struct Circle: Drawable {
    func draw() {
        print("Drawing circle")
    }
}

// Protocol with properties
protocol Named {
    var name: String { get }
}

// Protocol inheritance
protocol FullyNamed: Named {
    var fullName: String { get }
}

// Protocol extensions (default implementations)
extension Drawable {
    func drawWithBorder() {
        print("Border")
        draw()
        print("Border")
    }
}

// Protocol-oriented programming
protocol Vehicle {
    var speed: Double { get }
    func accelerate()
}

extension Vehicle {
    func accelerate() {
        print("Accelerating at \(speed) km/h")
    }
}

struct Car: Vehicle {
    var speed: Double = 0
}

// Protocol composition
func process(_ object: Drawable & Named) {
    print("Drawing \(object.name)")
    object.draw()
}
```

Protocol-oriented programming is Swift's alternative to inheritance-heavy OOP.

## Enums: More Than Just Constants

Swift enums are powerful:

```swift
// Simple enum
enum Direction {
    case north, south, east, west
}

// Associated values
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}

let barcode = Barcode.upc(8, 85909, 51226, 3)

// Pattern matching
switch barcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check)")
case .qrCode(let code):
    print("QR code: \(code)")
}

// Raw values
enum Planet: Int {
    case mercury = 1, venus, earth, mars
}

// Methods on enums
enum TrafficLight {
    case red, yellow, green

    func canGo() -> Bool {
        switch self {
        case .green: return true
        default: return false
        }
    }
}

// Recursive enums
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}
```

Enums with associated values are like Rust's enums or Haskell's algebraic data types.

## Error Handling

Swift uses explicit error handling:

```swift
// Define errors
enum FileError: Error {
    case notFound
    case permissionDenied
    case corruptedData
}

// Throwing function
func readFile(path: String) throws -> String {
    if !fileExists(path) {
        throw FileError.notFound
    }
    return "File contents"
}

// Calling throwing functions
do {
    let contents = try readFile(path: "data.txt")
    print(contents)
} catch FileError.notFound {
    print("File not found")
} catch {
    print("Other error: \(error)")
}

// Try? (converts to optional)
let contents = try? readFile(path: "data.txt")  // Optional<String>

// Try! (force try, crashes on error)
let contents = try! readFile(path: "data.txt")  // Don't use unless you're sure

// Defer (cleanup code)
func process() {
    let file = openFile()
    defer { file.close() }  // Runs when scope exits

    // Work with file...
}
```

Errors are values, not exceptions flying through the call stack.

## Property Wrappers: Metaprogramming Made Easy

Property wrappers reduce boilerplate:

```swift
// Define a property wrapper
@propertyWrapper
struct Clamped<Value: Comparable> {
    var value: Value
    let range: ClosedRange<Value>

    init(wrappedValue: Value, _ range: ClosedRange<Value>) {
        self.range = range
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }

    var wrappedValue: Value {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }
}

// Use it
struct Game {
    @Clamped(0...100) var health = 100
}

var game = Game()
game.health = 150  // Clamped to 100
game.health = -10  // Clamped to 0

// SwiftUI uses property wrappers extensively
// @State, @Binding, @ObservedObject, @Published, etc.
```

Property wrappers enable powerful abstractions with minimal syntax.

## What Swift Is Best For

**iOS/macOS development**: The primary and best use case. SwiftUI is excellent.

**watchOS/tvOS**: All Apple platforms.

**Server-side Swift**: Vapor and Kitura frameworks. Growing but niche.

**Cross-platform apps**: With SwiftUI, write once for all Apple platforms.

**Command-line tools**: Fast, safe, good for utilities.

**Learning**: Great first language. Safe, modern, well-documented.

## What Swift Is Worst For

**Android development**: Use Kotlin or Java.

**Windows desktop apps**: Use C# or Electron.

**Web frontend**: Use JavaScript/TypeScript.

**Legacy systems**: Swift is too new for old infrastructure.

**Embedded systems**: Too high-level. Use C or Rust.

**Cross-platform mobile**: Flutter or React Native are better for iOS + Android.

## The Ecosystem

**Frameworks**:
- **SwiftUI**: Declarative UI framework (game-changer)
- **UIKit**: Legacy UI framework (still widely used)
- **Combine**: Reactive programming
- **Foundation**: Standard library extensions

**Server-side**:
- **Vapor**: Web framework
- **Kitura**: IBM's server framework
- **Hummingbird**: Modern, lightweight

**Package management**: Swift Package Manager (SPM)

**Build system**: Xcode (Apple platforms), SPM (cross-platform)

**Testing**: XCTest

The ecosystem is rich on Apple platforms, growing elsewhere.

## SwiftUI: The Future of Apple Development

SwiftUI revolutionized Apple UI development:

```swift
import SwiftUI

struct ContentView: View {
    @State private var name = ""

    var body: some View {
        VStack {
            TextField("Enter name", text: $name)
            Text("Hello, \(name)!")
                .font(.largeTitle)
            Button("Clear") {
                name = ""
            }
        }
        .padding()
    }
}
```

Declarative, reactive, cross-platform (all Apple platforms). It's what modern UI development should be.

## The Community

**The iOS Developers**: Building apps for Apple's ecosystem. Love SwiftUI.

**The Objective-C Refugees**: Escaped Objective-C's syntax. Never going back.

**The Server-Side Swift Advocates**: Building backends in Swift. Small but passionate.

**The Learners**: Using Swift as a first language. Apple promotes it for education.

**The Cross-Platform Dreamers**: Hoping Swift becomes more viable outside Apple.

### Common Phrases
- "It's not Objective-C!"
- "Protocol-oriented programming"
- "Value semantics for the win"
- "SwiftUI is the future"
- "Optionals eliminate crashes"
- "It's actually open source"
- "Just use guard let"

## Memory Management: ARC

Swift uses Automatic Reference Counting (ARC):

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    deinit { print("\(name) is being deinitialized") }
}

var reference1: Person? = Person(name: "Alice")
var reference2 = reference1
var reference3 = reference1

// Object deallocated when all references are nil
reference1 = nil
reference2 = nil
reference3 = nil  // "Alice is being deinitialized"

// Strong reference cycles (problem)
class Apartment {
    var tenant: Person?
}

class Person {
    var apartment: Apartment?
}

// Solution: weak or unowned references
class Apartment {
    weak var tenant: Person?  // Weak: optional, becomes nil
}

class Person {
    unowned var apartment: Apartment  // Unowned: non-optional, assumes valid
}
```

ARC is deterministic (unlike garbage collection) but requires understanding reference cycles.

## Should You Learn Swift?

**Yes, if**:
- You're developing for Apple platforms (essential)
- You want to learn a modern, well-designed language
- You're interested in iOS/macOS app development
- You want a safe, fast compiled language

**No, if**:
- You're developing for Android or Windows primarily
- You need maximum cross-platform compatibility
- You're building web applications

**Maybe, if**:
- You're interested in server-side Swift
- You want to understand modern language design
- You're learning programming (it's a good choice)

## The Verdict

Swift is Apple's vision of what a modern programming language should be. It's safe without being restrictive, fast without being low-level, and expressive without being cryptic.

For Apple platform development, Swift is the obvious choice. It's modern, well-supported, and continuously improving. SwiftUI makes cross-platform Apple development delightful.

Outside Apple's ecosystem, Swift is still finding its place. Server-side Swift exists but hasn't achieved widespread adoption. Cross-platform Swift is possible but not common.

Is Swift the most portable language? No. Is it the most mature? Not yet. But is it the best language for Apple platforms? Absolutely.

Swift shows what happens when a major company invests in language design. It borrows the best ideas from functional and imperative languages, adds protocol-oriented programming, and wraps it in syntax that's clean and approachable.

If you're in the Apple ecosystem, Swift isn't optional—it's essential. If you're not, Swift is still worth studying as an example of modern language design done well.

The language proves that you can be safe and fast, modern and practical, expressive and beginner-friendly. Swift isn't just for Apple anymore, but Apple is still where it shines brightest.

---

**Next**: [Chapter 16: Dart - Flutter's Secret Weapon](16-dart.md)
