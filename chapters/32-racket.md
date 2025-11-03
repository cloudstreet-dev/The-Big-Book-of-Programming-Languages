# Chapter 32: Racket - The Language for Making Languages

## The Programmable Programming Language

Racket began life in 1995 as PLT Scheme, created by Matthias Felleisen and a team at Rice University (later at Northeastern). It started as a Scheme implementation for teaching but evolved into something more ambitious: a platform for building languages.

By 2010, it had diverged enough from Scheme to warrant a new name: Racket. The rebrand reflected its unique position—not just another Lisp dialect but a language-oriented programming system.

Racket's motto: "The Language for Making Languages."

This isn't marketing. Racket provides a complete platform for designing, implementing, and using domain-specific languages. You don't just write programs in Racket; you extend Racket, creating custom languages tailored to your problem domain.

By 2025, Racket remains primarily academic and educational, but it's influenced mainstream language design. Its macro system inspired others, its teaching materials educated thousands, and its approach to language-oriented programming shaped modern metaprogramming.

## The Philosophy

**Language-oriented programming**: Don't adapt your problem to the language—adapt the language to the problem. Create domain-specific languages for specific domains.

**Extensibility at the core**: Macros aren't bolted on; they're fundamental. The language itself is extensible, mutable, and programmable.

**Educational focus**: Designed for teaching programming concepts. DrRacket IDE is built for learners. The HtDP (How to Design Programs) textbook uses Racket.

**Batteries included**: Comprehensive standard library, GUI toolkit, web server, documentation system, package manager. Everything needed for serious development.

**Contracts as specifications**: Design by contract built into the language. Specify function contracts, enforce them at runtime.

**Strong tooling**: DrRacket IDE, debugger, profiler, documentation generator. Professional-grade tools.

The result: A language ecosystem where creating new languages is a normal, encouraged activity.

## What It Looks Like

### Basic Syntax (Classic Lisp Parentheses)

```racket
#lang racket

; Simple expressions
(+ 1 2 3)           ; 6
(* 4 5)             ; 20
(- 10 3)            ; 7
(/ 15 3)            ; 5

; Variables
(define x 42)
(define name "Alice")

; Functions
(define (greet name)
  (string-append "Hello, " name "!"))

(greet "World")     ; "Hello, World!"

; Lambda (anonymous functions)
(define square
  (lambda (x) (* x x)))

; Or shorter syntax
(define (square x)
  (* x x))

; Multiple arguments
(define (add x y z)
  (+ x y z))

; Conditionals
(define (abs x)
  (if (< x 0)
      (- x)
      x))

; Cond (like switch/case)
(define (describe-number n)
  (cond
    [(< n 0) "negative"]
    [(= n 0) "zero"]
    [(> n 0) "positive"]))

; Let bindings
(define (quadratic a b c x)
  (let ([a-squared (* a x x)]
        [b-term (* b x)])
    (+ a-squared b-term c)))
```

### Lists and Data Structures

```racket
; Lists
(define numbers (list 1 2 3 4 5))
(define colors '(red green blue))  ; Quote syntax

; List operations
(first numbers)      ; 1
(rest numbers)       ; '(2 3 4 5)
(length numbers)     ; 5

; Cons (construct)
(cons 0 numbers)     ; '(0 1 2 3 4 5)

; Map, filter, fold
(map square numbers)                 ; '(1 4 9 16 25)
(filter even? numbers)               ; '(2 4)
(foldl + 0 numbers)                  ; 15

; Structures
(struct point (x y) #:transparent)

(define p (point 10 20))
(point-x p)          ; 10
(point-y p)          ; 20

; Structures with methods
(struct circle (radius) #:transparent
  #:methods gen:custom-write
  [(define (write-proc c port mode)
     (fprintf port "<circle ~a>" (circle-radius c)))])
```

### Pattern Matching

```racket
(require racket/match)

(define (describe-list lst)
  (match lst
    ['() "empty list"]
    [(list x) (format "single element: ~a" x)]
    [(list x y) (format "two elements: ~a and ~a" x y)]
    [(cons first rest) (format "starts with ~a" first)]))

(describe-list '())          ; "empty list"
(describe-list '(42))        ; "single element: 42"
(describe-list '(1 2 3))     ; "starts with 1"

; Match with structures
(define (point-describe p)
  (match p
    [(point 0 0) "origin"]
    [(point 0 y) (format "on y-axis at ~a" y)]
    [(point x 0) (format "on x-axis at ~a" x)]
    [(point x y) (format "point at (~a, ~a)" x y)]))
```

### Macros: The Superpower

```racket
; Simple macro
(define-syntax-rule (swap! x y)
  (let ([tmp x])
    (set! x y)
    (set! y tmp)))

(define a 1)
(define b 2)
(swap! a b)
; Now a=2, b=1

; More complex macro
(define-syntax when
  (syntax-rules ()
    [(_ condition body ...)
     (if condition
         (begin body ...)
         (void))]))

(when (> 5 3)
  (displayln "5 is greater than 3")
  (displayln "This also runs"))

; Loop macro
(define-syntax-rule (repeat n body ...)
  (for ([i n])
    body ...))

(repeat 3
  (displayln "Hello"))

; DSL for state machines
(define-syntax define-state-machine
  (syntax-rules (state transition on)
    [(_ name
        (state initial-state)
        (transition from-state on event to-state) ...)
     (define name
       (let ([current initial-state])
         (lambda (event)
           (cond
             [(and (eq? current 'from-state) (eq? event 'event))
              (set! current 'to-state)] ...))))]))

; Use the DSL
(define-state-machine door
  (state closed)
  (transition closed on open opened)
  (transition opened on close closed))
```

### Contracts: Design by Contract

```racket
; Simple contracts
(define/contract (divide x y)
  (-> number? (and/c number? (not/c zero?)) number?)
  (/ x y))

; (divide 10 0)  ; Contract violation!

; More complex contracts
(define/contract (sorted-list lst)
  (-> (listof number?) (listof number?))
  (sort lst <))

; Function contracts
(define/contract (process-user user)
  (-> (hash/c 'name string? 'age exact-positive-integer?) string?)
  (format "User: ~a, Age: ~a"
          (hash-ref user 'name)
          (hash-ref user 'age)))

; Dependent contracts
(define/contract (make-vector size init)
  (->i ([size exact-nonnegative-integer?]
        [init any/c])
       [result (size) (and/c vector?
                             (λ (v) (= (vector-length v) size)))])
  (make-vector size init))
```

### Modules and Packages

```racket
; Define a module
(module shapes racket
  (provide circle-area rectangle-area)

  (define pi 3.14159)

  (define (circle-area radius)
    (* pi radius radius))

  (define (rectangle-area width height)
    (* width height)))

; Use a module
(require 'shapes)
(circle-area 5)

; File-based modules (geometry.rkt)
#lang racket
(provide (all-defined-out))

(define (distance x1 y1 x2 y2)
  (sqrt (+ (sqr (- x2 x1))
           (sqr (- y2 y1)))))

; Import from file
(require "geometry.rkt")
(distance 0 0 3 4)  ; 5
```

### Language-Oriented Programming: The Big Idea

```racket
; Create a simple language for arithmetic
#lang racket

(provide (rename-out [my-+ +]
                     [my-* *])
         #%module-begin
         #%app #%datum)

(define (my-+ . args)
  (displayln (format "Adding: ~a" args))
  (apply + args))

(define (my-* . args)
  (displayln (format "Multiplying: ~a" args))
  (apply * args))

; Use your custom language
#lang s-exp "my-arithmetic.rkt"
(+ 1 2 3)     ; Prints: "Adding: (1 2 3)", returns 6
(* 4 5)       ; Prints: "Multiplying: (4 5)", returns 20
```

Built-in language-oriented examples:

```racket
; Scribble: Documentation language
#lang scribble/manual
@title{My Documentation}
@section{Introduction}
This is @bold{important}.

; Datalog: Logic programming
#lang datalog
parent(john, mary).
parent(john, tom).
ancestor(A, B) :- parent(A, B).
ancestor(A, B) :- parent(A, C), ancestor(C, B).
ancestor(john, X)?

; Typed Racket: Statically typed
#lang typed/racket
(: add (-> Number Number Number))
(define (add x y)
  (+ x y))
```

## Real-World Example: Web Server

```racket
#lang racket
(require web-server/servlet
         web-server/servlet-env)

; Handler function
(define (start request)
  (response/xexpr
   '(html
     (head (title "Racket Web Server"))
     (body
      (h1 "Hello from Racket!")
      (p "This is a web server written in Racket.")
      (form ((action "/greet") (method "post"))
            (input ((type "text") (name "name")))
            (input ((type "submit") (value "Greet"))))))))

; Greet handler
(define (greet-handler request)
  (define bindings (request-bindings request))
  (define name (extract-binding/single 'name bindings))
  (response/xexpr
   `(html
     (body
      (h1 ,(format "Hello, ~a!" name))))))

; Start server
(serve/servlet start
               #:servlet-path "/"
               #:port 8080
               #:listen-ip #f)
```

## Classes and Objects (Yes, Really)

```racket
; Racket has OOP too
(define fish%
  (class object%
    (init size)
    (super-new)

    (define current-size size)

    (define/public (get-size)
      current-size)

    (define/public (grow amt)
      (set! current-size (+ current-size amt)))

    (define/public (eat other-fish)
      (grow (send other-fish get-size)))))

(define charlie (new fish% [size 10]))
(define wanda (new fish% [size 5]))

(send charlie eat wanda)
(send charlie get-size)  ; 15

; Inheritance
(define shark%
  (class fish%
    (init size teeth)
    (super-new [size size])

    (define num-teeth teeth)

    (define/public (get-teeth)
      num-teeth)

    (define/override (eat other-fish)
      (super eat other-fish)
      (displayln "CHOMP!"))))
```

## The Good Parts

**Unparalleled metaprogramming**: Macro system is the best in any language. Transform syntax, create DSLs, extend the language at will.

**Language-oriented programming**: Building custom languages for specific problems is normal, not exceptional. True linguistic abstraction.

**Excellent for education**: DrRacket IDE is perfect for learning. How to Design Programs textbook is influential. Teaching languages (Beginning Student, Intermediate Student) ease learners in.

**Strong tooling**: DrRacket IDE, debugger, profiler, comprehensive documentation. Not typical for Lisp dialects.

**Contracts**: Design by contract built into the language. Specify and enforce function contracts.

**Batteries included**: GUI toolkit, web server, graphics, testing, package manager. Complete ecosystem.

**Multiple paradigms**: Functional, OOP, logic programming (Datalog), all available. Use what fits.

**Static typing available**: Typed Racket provides gradual typing. Add types where useful, skip where not.

## The Pain Points

**Parentheses overload**: Lisp syntax is intimidating. Lots of parentheses. Not familiar to most programmers.

**Small job market**: Academic and educational focus. Few industry jobs specifically for Racket.

**Performance**: Slower than compiled languages. Fast for a dynamic language, but not competitive with C, Rust, or Go.

**Niche ecosystem**: Package ecosystem is small. Many libraries are academic projects, not production-ready.

**Learning curve**: Functional programming, macros, and Lisp syntax all at once. Steep for beginners.

**Macros are powerful and dangerous**: With great power comes great potential for incomprehensible code. Macros can make code unreadable.

**Limited industry adoption**: Primarily used in academia and education. Rare in production systems.

## Use Cases

**Teaching programming**: HtDP book, DrRacket IDE, teaching languages. Perfect for CS education.

**Language design research**: Academia uses Racket to experiment with language features and paradigms.

**Domain-specific languages**: When you need a custom language for a specific problem, Racket makes it tractable.

**Rapid prototyping**: High-level abstractions, quick iteration. Good for exploring ideas.

**Academic research**: Type systems, program analysis, language semantics. Racket is common in PL research.

**Scripting and automation**: High-level, powerful. Like Python but with better metaprogramming.

## The Ecosystem

**Package manager**: `raco pkg` handles packages well. Racket Package Catalog has thousands of packages.

**Libraries**: Web frameworks, GUI (racket/gui), graphics, data structures, testing frameworks. More complete than you'd expect for a niche language.

**DrRacket IDE**: Integrated development environment built specifically for Racket. Excellent for learning and development.

**Documentation**: Fantastic. Comprehensive, searchable, with examples. Sets the standard for language documentation.

**Community**: Small but welcoming. Academic focus. Helpful mailing list and Slack.

## Common Phrases You'll Hear

**"Language-oriented programming"**: The core philosophy. Create languages for problems, not just programs.

**"Syntax is just data"**: Homoiconicity. Code is lists, lists are data. Macros manipulate syntax as data.

**"Design by contract"**: Contracts specify and enforce function behavior. Racket takes this seriously.

**"A programmable programming language"**: You don't just use Racket; you extend and modify it.

**"How to Design Programs"**: The famous textbook. HtDP teaches programming using Racket's teaching languages.

**"Macros all the way down"**: Many language features are implemented as macros. Extensibility isn't theoretical.

## The Verdict

Racket is the language for people who want to make languages. It's unmatched for metaprogramming, language-oriented programming, and teaching programming concepts.

**Choose Racket if**:
- You're teaching or learning programming
- You need to create domain-specific languages
- You value extensibility and metaprogramming power
- You're doing programming language research
- You want to explore functional programming and Lisp
- You need a complete, well-documented ecosystem

**Avoid Racket if**:
- You need industry jobs and mainstream adoption
- Performance is critical
- You're working with non-technical stakeholders (Lisp syntax scares people)
- You want a large package ecosystem
- You dislike parentheses
- You need a language your team already knows

Racket won't power your startup. It won't land you a high-paying job at a tech giant (unless you're in research). It won't compete with Python for data science or JavaScript for web development.

But if you want to understand how programming languages work, if you need to create custom languages for specific problems, or if you're teaching programming, Racket is exceptional.

It's the language for language enthusiasts. The meta-language. The toolkit for building tools.

And if you've ever thought "I wish this language did X," Racket lets you make it do X. Just write a macro.

Racket is Lisp taken to its logical conclusion: a language that's really a language construction kit. Most programmers will never need that level of power. But for those who do, nothing else comes close.

---

**Next**: [Chapter 33: Prolog - When You Want to Confuse Everyone](33-prolog.md)
