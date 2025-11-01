# Chapter 32: Racket - The Language for Making Languages

## The Programmable Programming Language

Racket (formerly PLT Scheme) is a descendant of Scheme and Lisp, created in the 1990s at Brown University. It's designed to create other programming languages—a meta-language for language designers, educators, and researchers.

Racket's motto: "The language for making languages."

## The Philosophy

**Language-oriented programming**: Create domain-specific languages.

**Extensibility**: Macros that transform the language itself.

**Education**: Designed for teaching programming concepts.

**Powerful tooling**: IDE, debugger, documentation built-in.

## What It Looks Like

```racket
#lang racket

; Simple expressions
(+ 1 2 3)  ; 6
(* 4 5)    ; 20

; Functions
(define (greet name)
  (string-append "Hello, " name "!"))

; Macros (extend the language)
(define-syntax-rule (swap! x y)
  (let ([tmp x])
    (set! x y)
    (set! y tmp)))

; Create your own language
#lang slideshow
(circle 50)
```

## The Verdict

Racket is for educators, language designers, and programmers who want to create custom DSLs. It's powerful, well-tooled, and designed for metaprogramming.

Not for general application development, but invaluable for teaching programming and exploring language design.

---

**Next**: [Chapter 33: Prolog - When You Want to Confuse Everyone](33-prolog.md)
