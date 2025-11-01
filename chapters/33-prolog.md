# Chapter 33: Prolog - When You Want to Confuse Everyone

## The Logic Programming Language

Prolog (Programming in Logic) was created in 1972 by Alain Colmerauer and Philippe Roussel. Unlike imperative or functional languages, Prolog is declarative—you describe what you want, not how to get it.

Prolog is used in AI, natural language processing, and expert systems. It's also famous for making programmers' heads hurt.

## The Philosophy

**Logic programming**: Define facts and rules. Let Prolog figure out the solution.

**Declarative**: Describe relationships, not procedures.

**Backtracking**: Automatically tries alternatives when stuck.

**Pattern matching**: Unification is central.

## What It Looks Like

```prolog
% Facts
parent(tom, bob).
parent(tom, liz).
parent(bob, ann).

% Rules
grandparent(X, Z) :- parent(X, Y), parent(Y, Z).

% Queries
?- grandparent(tom, ann).
% Yes

% Lists
member(X, [X|_]).
member(X, [_|T]) :- member(X, T).

% Recursion
length([], 0).
length([_|T], N) :- length(T, N1), N is N1 + 1.
```

## The Verdict

Prolog is the language for logic problems, AI, and making your brain work differently. It's great for constraint solving, expert systems, and natural language processing.

Not for web apps or general programming. Learn it to expand your mind, not your job prospects.

---

**Next**: [Chapter 34: PowerShell - Windows Finally Gets a Real Shell](34-powershell.md)
