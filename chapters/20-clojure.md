# Chapter 20: Clojure - Lisp for the JVM

## The Modern Lisp

Clojure was created in 2007 by Rich Hickey, who wanted a practical Lisp for the modern age. He chose the JVM for its mature ecosystem, performance, and ubiquity. The result is a Lisp that can use Java libraries, runs anywhere Java runs, and brings functional programming to enterprise environments.

Clojure looks strange if you've never seen Lisp: parentheses everywhere, prefix notation, and code that looks like data. But it's loved by its users for its simplicity, power, and elegance. Once the syntax clicks, many developers never want to go back.

By 2025, Clojure powers web services, data processing pipelines, and financial systems. It's not mainstream, but it's proven in production at companies that value correctness and developer productivity.

## The Philosophy

Clojure's design principles:

**Simplicity over ease**: Simple (one concept) beats easy (familiar). Prefer simple composable parts.

**Immutability by default**: Data doesn't change. This prevents bugs and enables concurrency.

**Functional programming**: Functions are first-class. Data transformation over mutation.

**Lisp heritage**: Code is data (homoiconicity). Macros for abstraction.

**JVM interop**: Seamless Java integration. Use the vast Java ecosystem.

**Practical**: Solve real problems. Not academic purity.

The result is a language that's radically different but deeply pragmatic.

## What It Looks Like

```clojure
;; Simple expressions
(+ 1 2 3)  ; 6
(* 4 5)    ; 20

;; Everything is prefix notation
(println "Hello, World!")

;; Variables (actually immutable bindings)
(def x 42)
(def name "Alice")

;; Functions
(defn greet [name]
  (str "Hello, " name "!"))

(greet "Bob")  ; "Hello, Bob!"

;; Anonymous functions
(fn [x] (* x 2))

;; Shorthand
#(* % 2)

;; Let bindings (local scope)
(let [x 10
      y 20]
  (+ x y))  ; 30

;; Data structures (all immutable)
[1 2 3 4 5]                    ; Vector
{:name "Alice" :age 30}        ; Map
#{1 2 3 4 5}                   ; Set
'(1 2 3 4 5)                   ; List

;; Keywords (like symbols, used as map keys)
:name
:age
:user/id  ; Namespaced keyword

;; Destructuring
(let [[a b c] [1 2 3]]
  (+ a b c))  ; 6

(let [{:keys [name age]} {:name "Alice" :age 30}]
  (str name " is " age))  ; "Alice is 30"

;; Threading macros
(-> "hello world"
    (clojure.string/upper-case)
    (clojure.string/split #" ")
    (first))  ; "HELLO"

(->> [1 2 3 4 5]
     (map #(* % 2))
     (filter even?)
     (reduce +))  ; 30

;; Lazy sequences
(take 10 (range))  ; (0 1 2 3 4 5 6 7 8 9)

;; Pattern matching (requires core.match)
(require '[clojure.core.match :refer [match]])

(defn classify [x]
  (match x
    0 "zero"
    1 "one"
    n "many"))
```

Key features:
- **Prefix notation**: `(function arg1 arg2)`
- **Immutable data**: Everything is immutable by default
- **Sequence abstraction**: Unified operations on collections
- **Macros**: Transform code before compilation
- **REPL-driven**: Interactive development

## The Type System

Clojure is dynamically typed:

```clojure
;; No type declarations
(def x 42)
(def x "hello")  ; Same var, different value, different type

;; Basic types
42                ; Long
3.14              ; Double
"hello"           ; String
:keyword          ; Keyword
true false        ; Boolean
nil               ; null

;; Collections
[1 2 3]           ; Vector (indexed)
'(1 2 3)          ; List (sequential)
{:a 1 :b 2}       ; Map
#{1 2 3}          ; Set

;; Symbols
'foo              ; Symbol
foo               ; Evaluates to value bound to foo

;; Functions are values
(def add-five #(+ % 5))
(add-five 10)     ; 15

;; Optional type hints (for Java interop performance)
(defn add ^long [^long a ^long b]
  (+ a b))

;; Spec (optional runtime validation)
(require '[clojure.spec.alpha :as s])

(s/def ::age (s/and int? #(>= % 0)))
(s/def ::name string?)
(s/def ::person (s/keys :req [::name ::age]))

(s/valid? ::person {::name "Alice" ::age 30})  ; true
(s/valid? ::person {::name "Bob" ::age -5})    ; false
```

Dynamic typing with optional runtime validation via spec.

## Immutability and Persistent Data Structures

Everything is immutable:

```clojure
;; "Changing" data returns new values
(def v [1 2 3])
(def v2 (conj v 4))  ; [1 2 3 4]
;; v is still [1 2 3]

;; Maps
(def m {:a 1 :b 2})
(def m2 (assoc m :c 3))  ; {:a 1, :b 2, :c 3}
;; m is still {:a 1 :b 2}

;; Persistent data structures share structure
;; "Changes" are fast (not full copies)
(def big-vec (vec (range 1000000)))
(def big-vec2 (assoc big-vec 500000 :changed))
;; Shares most structure with big-vec
;; Fast operation, minimal memory

;; Update nested structures
(def user {:name "Alice"
           :address {:city "NYC" :zip 10001}})

(update-in user [:address :city] clojure.string/upper-case)
;; {:name "Alice", :address {:city "NYC" :zip 10001}}

;; Atoms for managed mutation
(def counter (atom 0))

(swap! counter inc)  ; Atomically increment
@counter             ; Dereference to get value

;; Agents for async updates
(def log-agent (agent []))

(send log-agent conj "Event 1")  ; Async update
@log-agent  ; Eventually contains ["Event 1"]
```

Immutability eliminates whole classes of bugs, especially in concurrent code.

## Sequences and Laziness

Clojure's sequence abstraction unifies operations:

```clojure
;; Sequence operations work on vectors, lists, maps, sets
(map inc [1 2 3])        ; (2 3 4)
(filter even? [1 2 3 4]) ; (2 4)
(reduce + [1 2 3 4 5])   ; 15

;; Lazy sequences
(def infinite (range))   ; Infinite sequence
(take 5 infinite)        ; (0 1 2 3 4)

;; Lazy computation (not evaluated until needed)
(defn fib-seq
  ([] (fib-seq 0 1))
  ([a b] (lazy-seq (cons a (fib-seq b (+ a b))))))

(take 10 (fib-seq))  ; (0 1 1 2 3 5 8 13 21 34)

;; Transducers (composable transformations)
(def xf (comp
          (map inc)
          (filter even?)))

(transduce xf + [1 2 3 4 5])  ; 12

;; Works with different contexts
(into [] xf [1 2 3 4 5])  ; [2 4 6]
```

Laziness and the sequence abstraction enable elegant data processing.

## Macros: Code as Data

Lisp's superpower is macros:

```clojure
;; Simple macro
(defmacro unless [condition & body]
  `(if (not ~condition)
     (do ~@body)))

(unless false
  (println "This runs"))

;; Macros transform code before evaluation
(defmacro defn-memo [name args & body]
  `(def ~name (memoize (fn ~args ~@body))))

(defn-memo fib [n]
  (if (<= n 1)
    n
    (+ (fib (- n 1)) (fib (- n 2)))))
;; Automatically memoized

;; Threading macros (built-in)
(-> x
    (fn1 arg)
    (fn2 arg)
    (fn3 arg))
;; Expands to: (fn3 (fn2 (fn1 x arg) arg) arg)

(->> x
     (fn1 arg)
     (fn2 arg))
;; Expands to: (fn2 arg (fn1 arg x))

;; cond-> (conditional threading)
(cond-> {}
  true  (assoc :a 1)
  false (assoc :b 2)
  true  (assoc :c 3))
;; {:a 1, :c 3}
```

Macros let you extend the language itself.

## Java Interop

Clojure has seamless Java integration:

```clojure
;; Calling Java methods
(.toUpperCase "hello")  ; "HELLO"
(.length "hello")       ; 5

;; Static methods
(Math/pow 2 10)  ; 1024.0

;; Creating Java objects
(new java.util.Date)
(java.util.Date.)  ; Shorthand

;; Importing Java classes
(import '(java.util Date ArrayList))

(def list (ArrayList.))
(.add list "item")

;; Implementing Java interfaces
(defn comparator-fn [x y]
  (compare x y))

(def my-comparator
  (reify java.util.Comparator
    (compare [this x y]
      (compare x y))))

;; Using Java collections
(doto (ArrayList.)
  (.add "a")
  (.add "b")
  (.add "c"))
```

Access the entire Java ecosystem from Clojure.

## What Clojure Is Best For

**Data transformation**: Immutable data and sequence operations excel here.

**Web services**: Ring, Compojure, and Pedestal for backends.

**Data analysis**: Rich data structures and REPL exploration.

**Financial systems**: Immutability and correctness matter.

**Interactive development**: REPL-driven workflow is productive.

**Complex domains**: Simplicity and composability handle complexity.

## What Clojure Is Worst For

**Performance-critical systems**: JVM overhead. Use Rust, C++, or Go.

**Mobile apps**: Not designed for mobile. Use Swift, Kotlin, or Flutter.

**Systems programming**: Too high-level. Use C or Rust.

**Beginners**: Syntax is unfamiliar. Start with Python or JavaScript.

**Teams unfamiliar with Lisp**: Steep learning curve for Lisp syntax.

## The Ecosystem

**Build tool**: Leiningen or tools.deps/CLI

**Package repository**: Clojars (Clojure libraries), Maven Central (Java libraries)

**Web**:
- **Ring**: HTTP server abstraction
- **Compojure**: Routing
- **Pedestal**: Async web services
- **Re-frame**: Frontend (ClojureScript)

**Data**: **Datomic**: Immutable database created by Rich Hickey

**Testing**: **clojure.test**, **Midje**

**Tooling**: **CIDER** (Emacs), **Cursive** (IntelliJ), **Calva** (VS Code)

**ClojureScript**: Clojure that compiles to JavaScript for frontend

The ecosystem is smaller than Java or JavaScript but high quality.

## ClojureScript: Clojure in the Browser

ClojureScript compiles to JavaScript:

```clojure
;; Same syntax as Clojure
(defn greet [name]
  (str "Hello, " name "!"))

;; React integration (Reagent)
(defn counter []
  (let [count (reagent/atom 0)]
    (fn []
      [:div
       [:h1 "Count: " @count]
       [:button {:on-click #(swap! count inc)} "+"]])))

;; Re-frame for state management (like Redux)
(re-frame/reg-event-db
  :increment
  (fn [db _]
    (update db :count inc)))

(re-frame/reg-sub
  :count
  (fn [db _]
    (:count db)))
```

ClojureScript brings Clojure's benefits to frontend development.

## The Community

**The Lisp Lovers**: Appreciate Clojure's elegance and power.

**The Functional Programmers**: Use Clojure for immutability and simplicity.

**The REPL-Driven Developers**: Love interactive development workflow.

**The Data Engineers**: Use Clojure for data transformation pipelines.

**The Parentheses Skeptics**: "So many parentheses!" (Eventually get over it.)

### Common Phrases
- "Code is data, data is code"
- "Just use a map"
- "REPL-driven development"
- "Simple made easy" (famous Rich Hickey talk)
- "It's just data"
- "Embrace immutability"
- "Paren-phobia is temporary"

## REPL-Driven Development

The REPL (Read-Eval-Print Loop) is central to Clojure:

```clojure
;; Start REPL, define functions interactively
user=> (defn greet [name]
         (str "Hello, " name "!"))

user=> (greet "Alice")
"Hello, Alice!"

;; Modify function, test immediately
user=> (defn greet [name]
         (str "Hi, " name "!"))

user=> (greet "Alice")
"Hi, Alice!"

;; Test queries against live database
;; Explore data structures interactively
;; Build systems incrementally
```

The REPL enables a highly interactive workflow.

## Should You Learn Clojure?

**Yes, if**:
- You want to deeply understand functional programming
- You're interested in Lisp and macros
- You value immutability and simplicity
- You work with data transformation
- You want REPL-driven development

**No, if**:
- You need maximum performance (use Rust or C++)
- You're building mobile apps
- You or your team aren't ready for Lisp syntax
- You need the largest ecosystem

**Maybe, if**:
- You're curious about Lisp
- You want to understand immutability
- You're open to different approaches to programming

## The Verdict

Clojure is Lisp for the 21st century. It brings Lisp's power—code as data, macros, functional programming—to the JVM, making it practical for modern development.

Is Clojure for everyone? No. The syntax is unfamiliar. The functional style requires a shift in thinking. The ecosystem is smaller than mainstream languages.

But Clojure offers something unique: radical simplicity. Immutable data. Powerful abstractions. Interactive development. Code that's easy to reason about.

Rich Hickey's "Simple Made Easy" talk argues that we confuse familiarity (easy) with simplicity. Clojure is not easy—it's unfamiliar. But it is simple—it has fewer concepts, less incidental complexity.

Companies use Clojure in production for data pipelines, web services, and financial systems. They value correctness, simplicity, and developer productivity over familiarity.

Clojure won't replace JavaScript or Python. But for developers who embrace functional programming, value immutability, and appreciate Lisp's elegance, Clojure is a revelation.

Learn Clojure not because it's popular, but because it will change how you think about programming. The parentheses become invisible. The immutability becomes liberating. The simplicity becomes obvious.

Code is data. Data is code. Once you see it, you can't unsee it.

---

**Next**: [Chapter 21: F# - Functional .NET Done Right](21-fsharp.md)
