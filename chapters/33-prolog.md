# Chapter 33: Prolog - When You Want to Confuse Everyone

## The Logic Programming Alien

Prolog (Programming in Logic) was created in 1972 by Alain Colmerauer and Philippe Roussel in Marseille, France. It emerged from research in natural language processing and automated theorem proving.

Prolog is different. Radically different. It's not functional like Haskell, not object-oriented like Java, not procedural like C. It's *logical*—a paradigm so alien that most programmers find it mind-bending.

You don't write algorithms in Prolog. You state facts, define rules, and ask questions. Prolog figures out the answers through logical inference and backtracking. It's like having a conversation with a very logical (but sometimes frustratingly literal) reasoning engine.

In the 1980s, Prolog was the language of AI. Japan's Fifth Generation Computer Project bet heavily on Prolog. That project failed, and Prolog's mainstream relevance faded. But it survived in niches: expert systems, natural language processing, constraint solving, and academia.

By 2025, Prolog is obscure but not dead. It's the language you learn to expand your mind, not your job prospects.

## The Philosophy

**Logic programming paradigm**: Define what is true (facts) and what follows from truth (rules). Ask questions. Prolog computes answers through logical deduction.

**Declarative, not imperative**: You describe *relationships*, not *procedures*. No loops, no if-statements (in the traditional sense). Just logic.

**Backtracking search**: When Prolog hits a dead end, it automatically backtracks and tries alternatives. Built-in exhaustive search.

**Unification**: Pattern matching on steroids. Variables bind to values that make logical statements true.

**Horn clauses**: Prolog uses a subset of first-order logic. Specifically, Horn clauses—implications with at most one positive literal.

The result: A language that feels like doing math proofs, not writing programs.

## What It Looks Like

### Facts and Rules

```prolog
% Facts: statements that are true
parent(tom, bob).
parent(tom, liz).
parent(bob, ann).
parent(bob, pat).
parent(pat, jim).

male(tom).
male(bob).
male(jim).
female(liz).
female(ann).
female(pat).

% Rules: logical implications
% "X is the father of Y if X is parent of Y and X is male"
father(X, Y) :- parent(X, Y), male(X).

mother(X, Y) :- parent(X, Y), female(X).

% "X is grandparent of Z if X is parent of Y and Y is parent of Z"
grandparent(X, Z) :- parent(X, Y), parent(Y, Z).

% Sibling: same parent, different person
sibling(X, Y) :- parent(P, X), parent(P, Y), X \= Y.

% Ancestor (recursive rule)
ancestor(X, Y) :- parent(X, Y).
ancestor(X, Y) :- parent(X, Z), ancestor(Z, Y).
```

### Queries

```prolog
% Ask questions
?- father(tom, bob).
% Yes (true)

?- mother(liz, ann).
% No (false)

?- grandparent(tom, ann).
% Yes

% Variables (start with capital letter)
?- father(tom, X).
% X = bob ;
% X = liz ;
% No

?- ancestor(tom, Who).
% Who = bob ;
% Who = liz ;
% Who = ann ;
% Who = pat ;
% Who = jim ;
% No

% Multiple variables
?- parent(X, Y).
% X = tom, Y = bob ;
% X = tom, Y = liz ;
% X = bob, Y = ann ;
% ...
```

### Lists

```prolog
% List syntax: [Head|Tail]
% Head is first element, Tail is rest

% Member: X is member of list L
member(X, [X|_]).  % X is the head
member(X, [_|T]) :- member(X, T).  % Or X is in the tail

% Length of list
length([], 0).
length([_|T], N) :- length(T, N1), N is N1 + 1.

% Append two lists
append([], L, L).
append([H|T1], L2, [H|T3]) :- append(T1, L2, T3).

% Reverse a list
reverse([], []).
reverse([H|T], R) :- reverse(T, TR), append(TR, [H], R).

% Last element of list
last(X, [X]).
last(X, [_|T]) :- last(X, T).

% Sum of list
sum([], 0).
sum([H|T], S) :- sum(T, ST), S is H + ST.
```

### Pattern Matching and Unification

```prolog
% Unification: finding values that make statements true

% Simple unification
?- X = 5.
% X = 5

?- [H|T] = [1, 2, 3].
% H = 1, T = [2, 3]

?- [A, B | Rest] = [1, 2, 3, 4, 5].
% A = 1, B = 2, Rest = [3, 4, 5]

% Complex patterns
first_two([A, B | _], A, B).

?- first_two([10, 20, 30], X, Y).
% X = 10, Y = 20

% Pattern matching in function heads
process_list([]).
process_list([single]).
process_list([H|T]) :-
    write(H), nl,
    process_list(T).
```

### Arithmetic

```prolog
% Arithmetic evaluation with 'is'
calculate(X, Y, Sum, Product) :-
    Sum is X + Y,
    Product is X * Y.

?- calculate(5, 3, S, P).
% S = 8, P = 15

% Comparison
greater_than_ten(X) :- X > 10.

% Factorial
factorial(0, 1).
factorial(N, F) :-
    N > 0,
    N1 is N - 1,
    factorial(N1, F1),
    F is N * F1.

?- factorial(5, X).
% X = 120

% Fibonacci
fib(0, 0).
fib(1, 1).
fib(N, F) :-
    N > 1,
    N1 is N - 1,
    N2 is N - 2,
    fib(N1, F1),
    fib(N2, F2),
    F is F1 + F2.
```

### The Cut Operator (!)

```prolog
% Cut (!) prevents backtracking
% Use carefully—destroys Prolog's declarative nature

max(X, Y, X) :- X >= Y, !.
max(X, Y, Y).

% Without cut, both clauses could match
% Cut says "once first succeeds, don't try second"

% Deterministic member check
member_check(X, [X|_]) :- !.
member_check(X, [_|T]) :- member_check(X, T).

% First solution only
first_solution(X) :- solution(X), !.
```

### Negation as Failure

```prolog
% \+ is negation (not provable)
not_member(X, L) :- \+ member(X, L).

?- not_member(5, [1, 2, 3]).
% Yes

?- not_member(2, [1, 2, 3]).
% No

% Bachelor: male and not married
bachelor(X) :- male(X), \+ married(X).
```

## Real-World Example: Family Tree Queries

```prolog
% Complete family tree system

parent(george, elizabeth).
parent(george, margaret).
parent(elizabeth, charles).
parent(elizabeth, anne).
parent(philip, charles).
parent(philip, anne).
parent(charles, william).
parent(charles, harry).
parent(diana, william).
parent(diana, harry).

male(george).
male(philip).
male(charles).
male(william).
male(harry).

female(elizabeth).
female(margaret).
female(anne).
female(diana).

% Derived relationships
father(F, C) :- parent(F, C), male(F).
mother(M, C) :- parent(M, C), female(M).

% Grandparents
grandfather(GF, GC) :- father(GF, P), parent(P, GC).
grandmother(GM, GC) :- mother(GM, P), parent(P, GC).
grandparent(GP, GC) :- parent(GP, P), parent(P, GC).

% Siblings
full_sibling(X, Y) :-
    father(F, X), father(F, Y),
    mother(M, X), mother(M, Y),
    X \= Y.

half_sibling(X, Y) :-
    parent(P, X), parent(P, Y),
    X \= Y,
    \+ full_sibling(X, Y).

% Ancestors and descendants
ancestor(A, D) :- parent(A, D).
ancestor(A, D) :- parent(A, X), ancestor(X, D).

descendant(D, A) :- ancestor(A, D).

% Queries
?- grandfather(Who, william).
% Who = george ;
% Who = philip

?- full_sibling(william, Who).
% Who = harry

?- ancestor(elizabeth, Who).
% Who = charles ;
% Who = anne ;
% Who = william ;
% Who = harry
```

## Example: Solving Puzzles

```prolog
% Einstein's Zebra Puzzle (simplified)
% Who owns the zebra?

% Houses: [Color, Nationality, Pet, Drink, Smoke]
houses([
    house(red, english, _, _, _),
    house(_, spanish, dog, _, _),
    house(_, _, _, coffee, _),
    house(green, _, _, _, _),
    house(_, _, zebra, _, _)
]).

solution(Owner) :-
    houses(Houses),
    member(house(_, Owner, zebra, _, _), Houses).
```

## Constraint Logic Programming

```prolog
% Modern Prolog has constraint solving
:- use_module(library(clpfd)).

% Send More Money puzzle
% SEND + MORE = MONEY (each letter is unique digit)
sendmore(Digits) :-
    Digits = [S,E,N,D,M,O,R,Y],
    Digits ins 0..9,
    all_different(Digits),
    S #\= 0,
    M #\= 0,
                1000*S + 100*E + 10*N + D
    +           1000*M + 100*O + 10*R + E
    #= 10000*M + 1000*O + 100*N + 10*E + Y,
    label(Digits).

?- sendmore(D).
% D = [9,5,6,7,1,0,8,2]
% SEND = 9567
% MORE = 1085
% MONEY = 10652
```

## The Good Parts

**Declarative problem solving**: Describe the problem, not the solution. Prolog figures out the steps.

**Built-in backtracking**: Automatic search through solution space. No manual iteration or recursion control.

**Pattern matching**: Unification is elegant and powerful. Variables bind to make statements true.

**Logic puzzles and constraints**: Naturally express constraint satisfaction problems. Sudoku, scheduling, routing—all fit well.

**Natural language processing**: Original use case. Grammar rules map naturally to Prolog.

**Expert systems**: Rule-based reasoning is Prolog's forte. If-then rules are facts and implications.

**Database querying**: Prolog is essentially a logic database query language. Datalog is Prolog-inspired.

**Mind-expanding**: Learning Prolog changes how you think about problems. Pure declarative reasoning.

## The Pain Points

**Alien paradigm**: Logic programming is fundamentally different. Steep learning curve. Intuitions from other languages don't transfer.

**Performance unpredictable**: Backtracking can explode. Naive Prolog can be exponentially slow. Optimization requires understanding execution model.

**Debugging is hard**: When Prolog doesn't find a solution (or finds the wrong one), debugging is frustrating. No stack traces, just "No."

**Cut operator breaks declarativity**: The cut (!) is necessary for performance but destroys the logical purity. Code becomes order-dependent.

**Limited libraries**: Ecosystem is tiny. No modern web frameworks, no ML libraries, no mainstream tooling.

**String manipulation painful**: Lists of character codes. Not ergonomic for text processing despite NLP origins.

**Small community**: Niche academic language. Few resources, few jobs, few updates.

**I/O is awkward**: Side effects don't fit the logical paradigm. File I/O and user interaction feel bolted on.

## Use Cases

**Constraint satisfaction problems**: Scheduling, resource allocation, configuration problems. Prolog excels here.

**Expert systems**: Rule-based AI. Medical diagnosis, fault detection, recommendation systems.

**Natural language processing**: Grammar parsing, semantic analysis. Original design goal.

**Symbolic reasoning**: Theorem proving, logic puzzles, mathematical reasoning.

**Education**: Teaching logic programming, AI concepts, formal reasoning.

**Prototyping AI**: Quick exploration of logical approaches before implementing in faster languages.

**Knowledge representation**: Ontologies, semantic web, inference engines.

## The Ecosystem

**Implementations**:
- SWI-Prolog: Most popular modern implementation
- GNU Prolog: Free, fast
- SICStus Prolog: Commercial, well-supported
- YAP: Yet Another Prolog

**Libraries**: Constraint solving (CLP), web servers (minimal), database interfaces. Limited compared to mainstream languages.

**Tooling**: Text editors with syntax highlighting. Debuggers exist but aren't great. No modern IDE support.

**Community**: Small, academic. Helpful but tiny. Mostly researchers and educators.

**Documentation**: SWI-Prolog has good docs. Others vary. Academic papers are often the best resource.

## Common Phrases You'll Hear

**"It either says Yes or No"**: Prolog queries succeed or fail. No return values, just truth.

**"Let Prolog figure it out"**: The philosophy. State the problem, let the engine solve it.

**"Beware the cut"**: The cut operator (!) is powerful and dangerous. Use sparingly.

**"Unification is everything"**: Pattern matching via unification is the core operation.

**"Prolog is a mind-bender"**: Everyone says this. It's not hyperbole.

**"Great for the N-queens problem"**: Classic example. Prolog solves it elegantly (but not efficiently).

## The Verdict

Prolog is the most alien mainstream programming language. It's not functional, not imperative, not object-oriented. It's logical—a completely different way of thinking about computation.

**Choose Prolog if**:
- You're solving constraint satisfaction or logic problems
- You're building expert systems or rule-based AI
- You're teaching or studying logic programming
- You want to expand your mind and think differently
- You're working in natural language processing or symbolic AI
- You enjoy puzzles and logical reasoning

**Avoid Prolog if**:
- You're building web apps, mobile apps, or general software
- Performance is critical
- You need a large ecosystem and modern tooling
- Your team doesn't know logic programming
- You want job opportunities
- You value predictable execution time
- You need good string processing or I/O

Prolog won't land you a job. It won't power your startup. It won't compete with Python or JavaScript for mainstream use.

But Prolog will change how you think. Learning Prolog is like learning chess—it exercises different mental muscles. Even if you never use it professionally, understanding logic programming makes you a better programmer.

Prolog is the language equivalent of a thought experiment. It proves that computation doesn't have to be step-by-step instructions. You can describe what you want, and a computer can figure out how to get it.

For 99% of programming tasks, don't use Prolog. For the 1% where logic programming fits naturally—constraint solving, expert systems, symbolic reasoning—nothing else comes close.

It's the language you learn not to use, but to understand. And occasionally, when you encounter the right problem, you'll think "This would be elegant in Prolog" and smile at how weird and wonderful programming languages can be.

---

**Next**: [Chapter 34: PowerShell - Windows Finally Gets a Real Shell](34-powershell.md)
