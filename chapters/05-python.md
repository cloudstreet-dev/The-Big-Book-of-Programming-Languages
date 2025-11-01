# Chapter 5: Python - Executable Pseudocode

## The Language for Humans

Python was created in 1991 by Guido van Rossum, who wanted a language that was easy to read and fun to write. He succeeded so well that Python has become one of the most popular programming languages in the world, despite being "slow" by systems programming standards.

Python's secret? It makes the common cases easy and the hard cases possible. Whether you're writing a 10-line script or a million-line application, Python gets out of your way and lets you focus on solving problems.

It's been called "executable pseudocode" because well-written Python reads almost like English. This is not an accident. It's the entire point.

## The Philosophy

Python has an official philosophy, documented in "The Zen of Python" (type `import this` in a Python REPL). Key tenets:

- **Beautiful is better than ugly**
- **Explicit is better than implicit**
- **Simple is better than complex**
- **Readability counts**
- **There should be one—and preferably only one—obvious way to do it**

This last point is a dig at Perl, which famously has many ways to do everything. Python prefers one clear, obvious way.

The result is a language that values clarity, simplicity, and developer happiness. Where C++ asks "what features can we add?", Python asks "what features can we avoid adding?"

## What It Looks Like

Python is instantly recognizable by its use of indentation:

```python
def greet(name):
    if name:
        print(f"Hello, {name}!")
    else:
        print("Hello, stranger!")

numbers = [1, 2, 3, 4, 5]
squared = [x**2 for x in numbers]

for num in squared:
    print(num)
```

Key features:
- **No braces**: Indentation defines blocks
- **No semicolons**: Line breaks are statement delimiters
- **Dynamic typing**: Variables have no declared type
- **Everything is an object**: Even functions and classes
- **Batteries included**: Massive standard library

## The Type System

Python is dynamically typed, meaning types are checked at runtime:

```python
x = 42              # x is an int
x = "hello"         # now x is a string
x = [1, 2, 3]       # now x is a list

# This works:
def add(a, b):
    return a + b

add(1, 2)           # 3
add("Hello, ", "world")  # "Hello, world"
add([1], [2])       # [1, 2]
```

Modern Python has optional type hints:

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

numbers: list[int] = [1, 2, 3]
```

These are checked by tools like `mypy`, not by Python itself. They're documentation that can be verified, which is the best of both worlds... or a half-measure, depending on your perspective.

## The Good Parts

**List comprehensions**:
```python
squares = [x**2 for x in range(10)]
even_squares = [x**2 for x in range(10) if x % 2 == 0]
```

**Dictionary and set comprehensions too**:
```python
word_lengths = {word: len(word) for word in ["hello", "world"]}
unique_chars = {char for char in "hello world"}
```

**Context managers**:
```python
with open('file.txt') as f:
    contents = f.read()
# File automatically closed
```

**Destructuring**:
```python
x, y = (1, 2)
first, *rest = [1, 2, 3, 4]  # first=1, rest=[2,3,4]
```

**Everything is first-class**:
```python
functions = [len, sum, max]
result = [func([1, 2, 3]) for func in functions]
```

## The Standard Library

Python's standard library is legendary:

- **Data structures**: `collections`, `heapq`, `bisect`
- **File I/O**: `pathlib`, `shutil`, `tempfile`
- **Networking**: `socket`, `http`, `urllib`
- **Concurrency**: `threading`, `multiprocessing`, `asyncio`
- **Data formats**: `json`, `csv`, `xml`, `pickle`
- **Testing**: `unittest`, `doctest`
- **System**: `os`, `sys`, `subprocess`, `argparse`
- **And more**: `re` (regex), `datetime`, `random`, `itertools`, `functools`...

If you can think of it, there's probably a module for it. And if there isn't, there's definitely a PyPI package.

## The Package Ecosystem

**PyPI** (Python Package Index) hosts over 500,000 packages:

```python
pip install requests          # HTTP for humans
pip install numpy             # Numerical computing
pip install pandas            # Data analysis
pip install flask             # Web framework
pip install tensorflow        # Machine learning
```

Need something? Someone has probably already written it. The Python ecosystem is one of the richest in any language.

Key packages:
- **Web**: Django, Flask, FastAPI
- **Data science**: NumPy, Pandas, Matplotlib, Jupyter
- **Machine learning**: TensorFlow, PyTorch, scikit-learn
- **Testing**: pytest, tox
- **Automation**: requests, beautifulsoup, selenium
- **CLI**: click, typer

## Python 2 vs Python 3

For years, Python had a version split:

**Python 2**: The old guard. Ended support in 2020.
**Python 3**: The future. Not quite backward compatible.

The transition took over a decade and was painful. But it's over now. Python 2 is dead. Use Python 3. Specifically, use Python 3.9 or later.

Key changes:
- `print` is a function: `print("hello")` not `print "hello"`
- Strings are Unicode by default
- Integer division returns float: `5 / 2 == 2.5`
- Many standard library improvements

If you see Python 2 code in 2025, run away or prepare for archaeology.

## What Python Is Best For

**Data science and machine learning**: NumPy, Pandas, TensorFlow, PyTorch. Python owns this space.

**Scripting and automation**: Quick scripts to automate boring tasks.

**Web backends**: Django and Flask power millions of sites.

**APIs**: FastAPI makes building APIs delightful.

**Testing and DevOps**: Great scripting language for CI/CD pipelines.

**Education**: The best first programming language. Clear syntax, instant feedback.

**Prototyping**: Get ideas working quickly, optimize later if needed.

## What Python Is Worst For

**Performance-critical code**: Python is slow. Use C, C++, Rust, or Julia instead.

**Mobile apps**: Not impossible, but not common. Use Swift, Kotlin, or Flutter.

**Systems programming**: No direct memory access, GC pauses. Use C, C++, or Rust.

**Real-time applications**: GC pauses and interpretation overhead are dealbreakers.

**Desktop GUIs**: Possible (tkinter, PyQt, wxPython) but not ideal.

## The Performance Question

Yes, Python is slow compared to C or Rust. Typically 10-100x slower for CPU-bound tasks.

But:
1. **Developer time is expensive**: If Python lets you ship in 1/10th the time, the tradeoff is worth it.
2. **Most code isn't CPU-bound**: I/O, network, and database calls are usually the bottleneck.
3. **You can drop down to C**: NumPy, Pandas, and TensorFlow are Python wrappers around C/C++/Fortran code.
4. **JIT compilers exist**: PyPy can be much faster for certain workloads.
5. **"Fast enough" is fast enough**: If it meets your requirements, ship it.

The Python community has a saying: "Make it work, make it right, make it fast." In that order.

## The Indentation Wars

Python's use of whitespace is its most controversial feature.

**Proponents**: Forces consistent, readable code. You were going to indent anyway.

**Opponents**: Tabs vs spaces matters more. Mixing them breaks your code. Copying from the web is annoying.

The official style guide (PEP 8) says: use 4 spaces. Not tabs. Not 2 spaces. 4 spaces.

Everyone ignores this in their personal projects, then adopts it for work projects because the linter demands it.

## The Ecosystem

**Package management**: `pip` for installation, `venv` or `poetry` for virtual environments.

**Testing**: `pytest` is the de facto standard. Simple, powerful, extensible.

**Linting**: `pylint`, `flake8`, `ruff` (the new hotness).

**Formatting**: `black` (opinionated), `autopep8` (PEP 8 compliant).

**Type checking**: `mypy`, `pyright`.

**IDEs**: PyCharm, VS Code, Vim/Neovim with LSP.

## The Community

Python's community is famously welcoming. The language is popular across domains:

**The Data Scientists**: Live in Jupyter notebooks. Know pandas better than Python itself.

**The Web Developers**: Django for enterprises, Flask for startups, FastAPI for APIs.

**The DevOps Engineers**: Use Python for infrastructure automation and tooling.

**The Beginners**: Learning Python as their first language (smart choice).

**The "I Just Need a Script" Crowd**: Python solves their problem in 20 lines.

**The Academics**: Scientists and researchers who code to support their work, not as their work.

### Common Phrases
- "There should be one obvious way to do it"
- "We're all consenting adults here" (regarding lack of private variables)
- "It's not a bug, the GIL is a feature" (said defensively)
- "Just use NumPy"
- "Have you tried `import antigravity`?"
- "Tabs vs spaces? Spaces. Four of them."

## Asyncio: Python's Async Revolution

Python added async/await in 3.5:

```python
import asyncio

async def fetch_data(url):
    # Simulate network request
    await asyncio.sleep(1)
    return f"Data from {url}"

async def main():
    results = await asyncio.gather(
        fetch_data("url1"),
        fetch_data("url2"),
        fetch_data("url3")
    )
    print(results)

asyncio.run(main())
```

This brought Node.js-style concurrency to Python. It's great for I/O-bound tasks, but the ecosystem split between sync and async libraries has caused friction.

## The GIL Problem

Python has a Global Interpreter Lock (GIL) that prevents true multi-threading for CPU-bound tasks. Only one thread can execute Python bytecode at a time.

**Workarounds**:
- Use `multiprocessing` for CPU-bound parallelism
- Use `asyncio` for I/O-bound concurrency
- Use NumPy/Pandas which release the GIL
- Use PyPy or alternative implementations
- Wait for Python 3.13's experimental GIL-free mode

The GIL is controversial. Some see it as a critical flaw. Others argue it simplifies the runtime and most workloads don't hit it.

## Should You Learn Python?

**Yes, if**:
- You're learning to program (start here!)
- You work in data science, ML, or scientific computing
- You need to automate tasks or write scripts
- You want to build web backends or APIs
- You value productivity over raw performance

**No, if**:
- You're building performance-critical systems
- You're doing mobile app development
- You're working on real-time systems or embedded devices

**But really, yes**: Python is one of the most versatile and employable languages in 2025. Even if it's not your primary language, you'll use it for something.

## The Verdict

Python is the language for people who want to solve problems, not fight their tools. It's slow, but productive. It's dynamically typed, but increasingly type-hint friendly. It's interpreted, but has a massive ecosystem of compiled libraries.

In 2025, Python dominates data science, is strong in web development, excellent for scripting, and remains the best first language for beginners. It won't win a drag race against C++, but it'll get you to your destination while you're still reading the C++ manual.

Python proves that sometimes, being easy to write and read matters more than being fast to execute. When your developer costs more per hour than your servers, Python's value proposition is clear.

Code readability counts. Python counted it first.

---

**Next**: [Chapter 6: JavaScript - The Language That Ate the World](06-javascript.md)
