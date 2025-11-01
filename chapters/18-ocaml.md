# Chapter 18: OCaml - The Practical Functional Language

## The Industrial Functional Language

OCaml (Objective Caml) was born from Caml, which came from ML (Meta Language), created in the 1970s. OCaml emerged in 1996, adding object-oriented features to Caml. Despite the name, you don't need to use objects—most OCaml code is purely functional.

Where Haskell pursues purity and theoretical elegance, OCaml pursues practicality. It's a functional language that doesn't apologize for having mutable state, imperative features, and excellent performance. It's used at Jane Street (financial trading), Facebook (for tools like Flow and Hack), and in academia for teaching and research.

OCaml is proof that functional programming can be fast, practical, and used in production systems handling billions of dollars.

## The Philosophy

OCaml's design principles:

**Practical functional programming**: Functional first, but pragmatic. Mutation when needed.

**Strong static typing**: Catch errors at compile time. Type inference minimizes annotations.

**Performance**: Fast compilation, fast execution. Competitive with C.

**Correctness**: The type system prevents entire classes of bugs.

**Expressiveness**: Pattern matching, algebraic types, higher-order functions.

**Industrial strength**: Built for real-world use, not just research.

The result is a language that feels academic but performs in production.

## What It Looks Like

```ocaml
(* Simple functions *)
let add x y = x + y

let square x = x * x

(* Lists *)
let numbers = [1; 2; 3; 4; 5]

let doubled = List.map (fun x -> x * 2) numbers

(* Pattern matching *)
let rec factorial n =
  match n with
  | 0 -> 1
  | n -> n * factorial (n - 1)

(* Algebraic data types *)
type color =
  | Red
  | Green
  | Blue
  | RGB of int * int * int

type 'a tree =
  | Leaf
  | Node of 'a * 'a tree * 'a tree

(* Options (like Maybe in Haskell) *)
type 'a option =
  | None
  | Some of 'a

let safe_div x y =
  if y = 0 then None
  else Some (x / y)

(* Records *)
type person = {
  name: string;
  age: int;
}

let alice = { name = "Alice"; age = 30 }

(* Modules *)
module Stack = struct
  type 'a t = 'a list

  let empty = []

  let push x stack = x :: stack

  let pop = function
    | [] -> None
    | x :: rest -> Some (x, rest)
end
```

Key features:
- **Type inference**: Rarely need type annotations
- **Immutability by default**: Values don't change
- **Pattern matching**: Powerful and pervasive
- **Algebraic data types**: Sum types and product types
- **First-class functions**: Functions as values

## The Type System

OCaml's type system is powerful and inferred:

```ocaml
(* Basic types - inferred! *)
let x = 42            (* int *)
let y = 3.14          (* float *)
let s = "hello"       (* string *)
let b = true          (* bool *)

(* Function types are inferred *)
let add x y = x + y   (* int -> int -> int *)

(* Explicit type annotations (rarely needed) *)
let add (x: int) (y: int): int = x + y

(* Polymorphic types *)
let identity x = x    (* 'a -> 'a *)

let first x y = x     (* 'a -> 'b -> 'a *)

(* Lists are homogeneous *)
let numbers = [1; 2; 3]       (* int list *)
let strings = ["a"; "b"; "c"] (* string list *)

(* Tuples *)
let pair = (1, "hello")       (* int * string *)
let triple = (1, 2.0, "hi")   (* int * float * string *)

(* Algebraic data types *)
type shape =
  | Circle of float
  | Rectangle of float * float
  | Triangle of float * float * float

(* Pattern matching with types *)
let area = function
  | Circle r -> 3.14159 *. r *. r
  | Rectangle (w, h) -> w *. h
  | Triangle (a, b, c) ->
      (* Heron's formula *)
      let s = (a +. b +. c) /. 2.0 in
      sqrt (s *. (s -. a) *. (s -. b) *. (s -. c))

(* Parametric polymorphism *)
type 'a option =
  | None
  | Some of 'a

let map_option f opt =
  match opt with
  | None -> None
  | Some x -> Some (f x)
(* ('a -> 'b) -> 'a option -> 'b option *)

(* Records *)
type point = { x: float; y: float }

(* Variant types with fields *)
type 'a tree =
  | Leaf
  | Node of {
      value: 'a;
      left: 'a tree;
      right: 'a tree;
    }
```

The type system is expressive without being overwhelming.

## Pattern Matching: The Central Feature

Pattern matching is idiomatic OCaml:

```ocaml
(* Basic matching *)
let describe_number n =
  match n with
  | 0 -> "zero"
  | 1 -> "one"
  | 2 -> "two"
  | _ -> "many"

(* Matching on structure *)
let rec sum_list lst =
  match lst with
  | [] -> 0
  | head :: tail -> head + sum_list tail

(* Matching algebraic types *)
type expr =
  | Num of int
  | Add of expr * expr
  | Mul of expr * expr

let rec eval = function
  | Num n -> n
  | Add (e1, e2) -> eval e1 + eval e2
  | Mul (e1, e2) -> eval e1 * eval e2

(* Nested patterns *)
let rec flatten_option = function
  | Some (Some x) -> Some x
  | Some None -> None
  | None -> None

(* Guards *)
let classify n =
  match n with
  | n when n < 0 -> "negative"
  | 0 -> "zero"
  | n when n > 0 -> "positive"
  | _ -> failwith "impossible"

(* As patterns *)
let duplicate_head lst =
  match lst with
  | (x :: _) as lst -> x :: lst
  | [] -> []

(* Or patterns *)
let is_weekend = function
  | "Saturday" | "Sunday" -> true
  | _ -> false
```

Pattern matching replaces many if-else chains and makes code declarative.

## Modules: Powerful Code Organization

OCaml's module system is sophisticated:

```ocaml
(* Basic module *)
module Math = struct
  let pi = 3.14159

  let circle_area r = pi *. r *. r

  let square_area s = s *. s
end

(* Using modules *)
let area = Math.circle_area 5.0

(* Module signatures (interfaces) *)
module type STACK = sig
  type 'a t
  val empty : 'a t
  val push : 'a -> 'a t -> 'a t
  val pop : 'a t -> ('a * 'a t) option
end

(* Implementing a signature *)
module Stack : STACK = struct
  type 'a t = 'a list

  let empty = []

  let push x stack = x :: stack

  let pop = function
    | [] -> None
    | x :: rest -> Some (x, rest)
end

(* Functors (modules parameterized by modules) *)
module type COMPARABLE = sig
  type t
  val compare : t -> t -> int
end

module MakeSet (Ord : COMPARABLE) = struct
  type element = Ord.t
  type t = element list

  let empty = []

  let rec mem x = function
    | [] -> false
    | y :: rest ->
        match Ord.compare x y with
        | 0 -> true
        | _ -> mem x rest

  let rec add x set =
    if mem x set then set else x :: set
end

(* Using functors *)
module IntCmp = struct
  type t = int
  let compare = compare
end

module IntSet = MakeSet(IntCmp)
```

Functors enable powerful abstraction and code reuse.

## Immutability vs Mutability

OCaml is functional but pragmatic:

```ocaml
(* Immutable by default *)
let x = 42
(* x = 43 is a comparison, not assignment! *)

(* Mutable references *)
let counter = ref 0

let increment () =
  counter := !counter + 1

(* ! dereferences, := assigns *)
let value = !counter

(* Mutable record fields *)
type person = {
  name: string;
  mutable age: int;
}

let alice = { name = "Alice"; age = 30 }

let have_birthday person =
  person.age <- person.age + 1

(* Arrays are mutable *)
let arr = [| 1; 2; 3; 4; 5 |]

arr.(0) <- 10  (* Mutate first element *)

(* Imperative loops *)
for i = 0 to Array.length arr - 1 do
  print_int arr.(i)
done

while !counter < 10 do
  increment ()
done
```

You can use mutation when performance demands it, but immutability is the default.

## What OCaml Is Best For

**Financial systems**: Jane Street uses OCaml for trading. Correctness and performance matter.

**Compilers and interpreters**: The type system and pattern matching are perfect for this.

**Static analysis tools**: Facebook's Flow and Hack are written in OCaml.

**Formal verification**: Proving correctness of code.

**Research**: Teaching functional programming, exploring type systems.

**Symbolic computation**: Theorem provers, computer algebra systems.

## What OCaml Is Worst For

**Web development**: Possible, but niche. Use JavaScript, Python, or Ruby.

**Systems programming**: Garbage collection is a limitation. Use Rust or C.

**Mobile apps**: No ecosystem. Use Swift, Kotlin, or Flutter.

**Data science**: Python dominates. OCaml has no equivalent to NumPy/Pandas.

**General application development**: Small ecosystem compared to mainstream languages.

## The Ecosystem

**Build system**: Dune (modern and good), OCamlbuild (older)

**Package manager**: OPAM

**Libraries**:
- **Core/Base**: Jane Street's standard library replacement
- **Lwt/Async**: Concurrent programming
- **Js_of_ocaml**: Compile OCaml to JavaScript
- **Cmdliner**: Command-line parsing

**Tools**:
- **Merlin**: IDE features (autocomplete, type inspection)
- **OCamlformat**: Code formatter
- **odoc**: Documentation generator

**IDEs**: VS Code (with OCaml extension), Emacs, Vim

The ecosystem is smaller than mainstream languages but high quality.

## Jane Street: OCaml in Production

Jane Street is a major trading firm that uses OCaml extensively:

- Billions of dollars traded using OCaml code
- Developed Core, Async, and other libraries
- Contributes significantly to OCaml development
- Proves OCaml works at scale in finance

They chose OCaml for:
- Strong typing catches bugs before production
- Fast execution for low-latency trading
- Expressive code reduces complexity
- Refactoring safety

If OCaml can handle high-frequency trading, it's industrial strength.

## The Community

**The Finance People**: Jane Street and others. Use OCaml for correctness and speed.

**The Academics**: Research programming languages, teach functional programming.

**The Compiler Writers**: Build compilers, interpreters, and analysis tools.

**The Type System Enthusiasts**: Explore advanced type features.

**The Pragmatic Functionalists**: Want functional programming without Haskell's purity.

### Common Phrases
- "The type system catches it"
- "Pattern matching makes it obvious"
- "It's fast and correct"
- "Functors are powerful"
- "Jane Street uses it"
- "It's like Haskell but practical"

## OCaml vs Haskell

The inevitable comparison:

**Haskell**:
- Pure functional (all functions are pure)
- Lazy evaluation
- More exotic type features
- Larger community and ecosystem
- Monads everywhere

**OCaml**:
- Functional but pragmatic (mutation allowed)
- Eager evaluation (strict)
- Simpler, more predictable
- Smaller community
- Better performance (generally)

**When to choose OCaml**: You want functional programming with practical performance and don't need lazy evaluation or purity.

**When to choose Haskell**: You want maximum expressiveness and are willing to embrace laziness and monads.

## ReasonML and ReScript

OCaml has influenced other languages:

**ReasonML**: OCaml with JavaScript-like syntax. Facebook created it for React developers.

**ReScript**: Fork of ReasonML. Compiles to readable JavaScript.

These bring OCaml's benefits to the JavaScript ecosystem with familiar syntax.

## Should You Learn OCaml?

**Yes, if**:
- You're interested in functional programming
- You want to build compilers or analysis tools
- You work in finance or formal verification
- You want performance with strong typing
- You're studying computer science

**No, if**:
- You need a large ecosystem for general development
- You're building web apps or mobile apps
- You prefer imperative or OOP
- You want maximum job opportunities

**Maybe, if**:
- You want to understand advanced type systems
- You're curious about industrial functional programming
- You want to learn ML-family languages

## The Verdict

OCaml is the functional programmer's secret weapon. It's not as famous as Haskell, not as popular as JavaScript, and not as trendy as Rust. But it's fast, correct, and practical.

In finance, OCaml handles billions of dollars. In tooling, it powers analysis tools used by millions. In academia, it teaches fundamental concepts. This isn't a toy language—it's battle-tested.

The type system is one of the best in the industry. Type inference means you get strong typing without verbosity. Pattern matching makes code declarative and clear. Modules and functors enable sophisticated abstraction.

Is OCaml for everyone? No. The ecosystem is small. The community is niche. You won't find as many resources as for Python or JavaScript.

But if you need correctness, performance, and expressiveness—if you're building compilers, financial systems, or verification tools—OCaml delivers.

OCaml proves that functional programming isn't just academic. It's practical, fast, and production-ready. It's Haskell's pragmatic cousin: less theoretical purity, more real-world applicability.

Learn OCaml to understand how functional programming works in production. Stay for the type system, the pattern matching, and the satisfaction of code that compiles and just works.

---

**Next**: [Chapter 19: Elixir - Ruby Meets Erlang's Reliability](19-elixir.md)
