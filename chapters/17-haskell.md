# Chapter 17: Haskell - Pure Functional or Bust

## The Mathematical Language

Haskell was created in 1990 by a committee of academics who wanted a standard lazy functional language for research. Named after logician Haskell Curry, it was designed to explore ideas from category theory, type theory, and lambda calculus.

The result is a language that feels like executable mathematics. Where other languages have side effects and mutation, Haskell has purity and immutability. Where others have statements, Haskell has expressions. Where others compromise, Haskell commits.

Haskell is not for everyone. It has a steep learning curve, demands a different way of thinking, and its abstractions can be intimidating. But for those who embrace it, Haskell offers a glimpse of what programming could be: elegant, composable, and provably correct.

## The Philosophy

Haskell's core principles:

**Purely functional**: Functions have no side effects. Given the same inputs, they always return the same output.

**Lazy evaluation**: Expressions aren't evaluated until needed. Infinite data structures are possible.

**Strong static typing**: Types are checked at compile time. Type errors are impossible at runtime.

**Type inference**: The compiler figures out types. You rarely write them explicitly.

**Immutability**: Values can't be changed. Variables don't vary.

**Composition over configuration**: Build complex things from simple, composable parts.

The language asks: "What if we took functional programming seriously? All the way?"

## What It Looks Like

```haskell
-- Functions are defined with pattern matching
factorial :: Int -> Int
factorial 0 = 1
factorial n = n * factorial (n - 1)

-- Higher-order functions
doubleAll :: [Int] -> [Int]
doubleAll = map (*2)

-- List comprehensions
evens = [x | x <- [1..100], even x]

-- Type classes for polymorphism
class Eq a where
    (==) :: a -> a -> Bool

-- Everything is an expression
result = if x > 0 then "positive" else "non-positive"

-- Infinite lists (lazy evaluation!)
allNumbers = [1..]
firstTen = take 10 allNumbers
```

Key features:
- **No assignments**: `x = 5` doesn't assign, it defines. `x` is always 5.
- **No loops**: Use recursion or higher-order functions like `map`, `filter`, `fold`.
- **Pattern matching**: Define functions by cases.
- **Type signatures**: Optional but recommended. `factorial :: Int -> Int`
- **Whitespace matters**: Indentation defines scope (like Python).

## The Type System

Haskell's type system is famously powerful:

**Basic types**:
```haskell
x :: Int              -- Fixed-precision integer
y :: Integer          -- Arbitrary-precision integer
z :: Double           -- Floating point
c :: Char             -- Character
s :: String           -- List of characters (actually [Char])
b :: Bool             -- True or False
```

**Function types**:
```haskell
add :: Int -> Int -> Int
add x y = x + y

-- Partial application
addFive :: Int -> Int
addFive = add 5
```

**Algebraic data types**:
```haskell
-- Sum types (like enums)
data Color = Red | Green | Blue

-- Product types (like structs)
data Point = Point Double Double

-- Combining both
data Shape = Circle Double
           | Rectangle Double Double
           | Triangle Double Double Double

-- Parametric types (generics)
data Maybe a = Nothing | Just a
data Either a b = Left a | Right b
```

**Type classes** (like interfaces):
```haskell
class Eq a where
    (==) :: a -> a -> Bool

instance Eq Color where
    Red == Red = True
    Green == Green = True
    Blue == Blue = True
    _ == _ = False
```

**Polymorphism**:
```haskell
-- Works with any type
id :: a -> a
id x = x

-- Works with any list
length :: [a] -> Int

-- Works with anything that has Eq
elem :: Eq a => a -> [a] -> Bool
```

## Purity and Side Effects

Functions in Haskell are pure—they don't have side effects:

```haskell
-- Pure function: always returns the same output for the same input
add :: Int -> Int -> Int
add x y = x + y

-- But how do we do I/O?
-- The type system tracks side effects!
main :: IO ()
main = do
    putStrLn "What's your name?"
    name <- getLine
    putStrLn ("Hello, " ++ name)
```

The `IO` type marks functions that interact with the outside world. Pure functions can't call `IO` functions. This prevents accidental side effects.

This seems restrictive, but it means:
- Pure functions are easy to test
- Pure functions can be parallelized safely
- Pure functions can be memoized
- You know from the type signature whether a function has side effects

## Lazy Evaluation

Haskell evaluates expressions only when needed:

```haskell
-- Infinite list of all positive integers
naturals = [1..]

-- Take the first 10 (only these are computed)
firstTen = take 10 naturals

-- Fibonacci sequence (infinite!)
fibs = 0 : 1 : zipWith (+) fibs (tail fibs)

-- First 10 Fibonacci numbers
firstTenFibs = take 10 fibs
```

Lazy evaluation enables:
- Infinite data structures
- Efficient short-circuiting
- Separation of data generation from consumption
- Elegant solutions to some problems

But it also makes performance harder to reason about. Space leaks can occur if you're not careful.

## Monads: The Infamous Abstraction

Monads are Haskell's most famous (and feared) concept:

```haskell
-- Maybe monad (for computations that might fail)
safeDivide :: Double -> Double -> Maybe Double
safeDivide _ 0 = Nothing
safeDivide x y = Just (x / y)

-- Chaining Maybe computations
compute :: Double -> Double -> Double -> Maybe Double
compute x y z = do
    a <- safeDivide x y
    b <- safeDivide a z
    return b

-- List monad (for non-deterministic computations)
pairs = do
    x <- [1, 2, 3]
    y <- [1, 2, 3]
    return (x, y)
-- Result: [(1,1), (1,2), (1,3), (2,1), (2,2), (2,3), (3,1), (3,2), (3,3)]

-- IO monad (for side effects)
main :: IO ()
main = do
    putStrLn "Hello"
    name <- getLine
    putStrLn ("Hi, " ++ name)
```

Monads are a design pattern for sequencing computations. They're abstract, but incredibly useful. The problem is explaining them—every explanation uses a different metaphor (burritos, boxes, railways, etc.).

The truth: Monads are just a way to chain operations that have some context (Maybe for failure, IO for side effects, List for multiple values, etc.). You don't need to understand category theory to use them.

## What Haskell Is Best For

**Domain-specific languages**: Haskell excels at building embedded DSLs.

**Compilers and interpreters**: Pure functions and algebraic data types shine here.

**Financial systems**: Correctness and immutability matter. Haskell delivers.

**Type-safe APIs**: The type system catches errors at compile time.

**Concurrent programming**: Purity makes parallelism safe and easy.

**Research and prototyping**: Exploring new programming ideas.

**Web backends** (surprisingly): Frameworks like Yesod, Servant, and Scotty work well.

## What Haskell Is Worst For

**Systems programming**: No direct memory control. Lazy evaluation is unpredictable.

**Real-time systems**: Garbage collection and lazy evaluation make timing hard.

**Game development**: Mutation is often natural for game state. Haskell fights this.

**GUIs**: Possible, but awkward. The ecosystem is limited.

**Quick scripts**: Too much overhead for simple automation.

**Teams of beginners**: The learning curve is steep.

## The Ecosystem

**Build tool**: Stack or Cabal for package management and builds.

**Package repository**: Hackage has thousands of packages.

**Key libraries**:
- **Web**: Yesod, Servant, Scotty, Warp
- **Parsing**: Parsec, Megaparsec, Attoparsec
- **Testing**: QuickCheck (property-based testing), HUnit
- **Concurrency**: async, STM (software transactional memory)
- **Data**: Aeson (JSON), text, bytestring

**GHC**: The Glasgow Haskell Compiler. Excellent optimizer, helpful error messages.

**GHCi**: Interactive REPL for experimentation.

## The Community

**The Academics**: Use Haskell for research. Publish papers. Explore type theory.

**The Pragmatists**: Use Haskell in production. Prove it's not just for research.

**The Type Enthusiasts**: Explore exotic type system features. Dependent types, GADTs, type families.

**The Learners**: Currently stuck understanding functors, applicatives, or monads.

**The Evangelists**: "If you're not using Haskell, you're doing it wrong." (Rarely helpful.)

### Common Phrases
- "A monad is just a monoid in the category of endofunctors, what's the problem?"
- "It compiles, so it works" (more true than in other languages)
- "Have you tried QuickCheck?"
- "Laziness strikes again" (when debugging performance)
- "Just use a monad transformer stack"
- "The type system won't let me do that" (said with both frustration and relief)

## Type-Level Programming

Haskell allows programming at the type level:

```haskell
-- Phantom types
data Validated
data Unvalidated

data User a = User { name :: String, email :: String }

validateUser :: User Unvalidated -> Maybe (User Validated)

-- GADTs (Generalized Algebraic Data Types)
data Expr a where
    IntVal :: Int -> Expr Int
    BoolVal :: Bool -> Expr Bool
    Add :: Expr Int -> Expr Int -> Expr Int
    If :: Expr Bool -> Expr a -> Expr a -> Expr a

-- Type families
type family Element c where
    Element [a] = a
    Element (Map k v) = (k, v)
```

This is powerful but complex. You can encode invariants in the type system that make certain bugs impossible.

## QuickCheck: Property-Based Testing

Haskell pioneered property-based testing:

```haskell
import Test.QuickCheck

-- Property: reverse of reverse is original
prop_reverseReverse :: [Int] -> Bool
prop_reverseReverse xs = reverse (reverse xs) == xs

-- QuickCheck generates random inputs to test the property
main = quickCheck prop_reverseReverse
```

QuickCheck generates hundreds of test cases automatically. When it finds a failing case, it shrinks it to the minimal example. This idea has been ported to many other languages.

## Should You Learn Haskell?

**Yes, if**:
- You want to deeply understand functional programming
- You're interested in type theory and programming language research
- You value correctness over ease
- You want to expand your programming perspective
- You're building compilers, parsers, or DSLs

**No, if**:
- You need to ship quickly with junior developers
- You're doing systems programming or game development
- You want a large ecosystem of libraries for everything
- You prefer imperative or object-oriented thinking

**Maybe, if**:
- You're curious about pure functional programming
- You want to understand what other languages borrow from Haskell (monads, type classes, etc.)
- You have time to invest in learning

## The Verdict

Haskell is programming's monastery. It demands discipline, contemplation, and a willingness to embrace abstraction. In return, it offers elegance, correctness, and a fundamentally different way of thinking about programs.

Is it practical? More than you'd think. Companies like Facebook, Standard Chartered, and Barclays use Haskell in production. It's not just academic.

Is it for everyone? Absolutely not. The learning curve is real. The abstractions are intimidating. The ecosystem is smaller than mainstream languages.

But Haskell is important beyond its direct usage. Ideas from Haskell—type classes, algebraic data types, pattern matching, immutability, monads—have influenced every major modern language. Rust, Swift, Kotlin, TypeScript, and Scala all borrowed from Haskell.

Learning Haskell makes you a better programmer in any language. It teaches you to think about types, purity, and composition. It shows you that programs can be mathematical and elegant, not just pragmatic.

You might never write production Haskell. But once you understand it, you'll write better Python, better JavaScript, better everything. Haskell is not just a language; it's an education.

And yes, a monad is just a monoid in the category of endofunctors. You'll understand that joke eventually. Maybe.

---

**Next**: [Chapter 18: OCaml - The Practical Functional Language](18-ocaml.md)
