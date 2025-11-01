# Chapter 7: PHP - The Internet's Underdog

## The Language Everyone Loves to Hate

PHP (originally "Personal Home Page," later backronymed to "PHP: Hypertext Preprocessor") was created in 1994 by Rasmus Lerdorf. He just wanted to track visits to his online resume. He didn't set out to create a programming language. Yet somehow, PHP became the engine behind a significant portion of the internet.

WordPress? PHP. Facebook (originally)? PHP. Wikipedia? PHP. Drupal, Laravel applications, countless small business websites? All PHP.

PHP has a reputation problem. It's been called ugly, inconsistent, and insecure. Programming language elitists mock it. "Real programmers" claim they'd never touch it. Yet in 2025, PHP still powers roughly 75% of websites with a known server-side language. Someone is using it, and successfully.

The truth is more nuanced than the memes suggest. Modern PHP (8.x) is actually a competent language with good performance, strong typing, and a mature ecosystem. But it carries the baggage of its messy past.

## The Philosophy

PHP's original philosophy (if it had one): **Make it easy to mix code and HTML**.

That's it. Rasmus wanted a simple way to create dynamic web pages. No grand design, no theoretical purity, just pragmatic solutions to immediate problems.

This resulted in:
- **Low barrier to entry**: Embed PHP in HTML, upload to a server, it works.
- **Flexibility over strictness**: Many ways to do things, few restrictions.
- **Batteries included**: Built-in functions for common web tasks.
- **Rapid development**: Get a website running quickly.

Modern PHP has added:
- **Performance**: PHP 7+ is fast, PHP 8+ is faster.
- **Type safety**: Optional type hints and strict typing.
- **Modern features**: Namespaces, traits, anonymous functions.
- **Framework ecosystem**: Laravel, Symfony, and others bring structure.

## What It Looks Like

Old-school PHP (mixing code and HTML):
```php
<!DOCTYPE html>
<html>
<head><title>My Page</title></head>
<body>
    <h1>Welcome, <?php echo $username; ?>!</h1>

    <?php
    $items = ['Apple', 'Banana', 'Orange'];
    foreach ($items as $item) {
        echo "<li>$item</li>";
    }
    ?>
</body>
</html>
```

Modern PHP (clean separation):
```php
<?php

declare(strict_types=1);

class User {
    public function __construct(
        private string $name,
        private int $age
    ) {}

    public function greet(): string {
        return "Hello, {$this->name}!";
    }
}

// Type hints
function processUsers(array $users): int {
    return count($users);
}

// Arrow functions (PHP 7.4+)
$squared = array_map(fn($x) => $x * $x, [1, 2, 3, 4, 5]);

// Match expressions (PHP 8+)
$message = match($status) {
    'draft' => 'Not published',
    'published' => 'Live',
    'archived' => 'Old content',
    default => 'Unknown'
};
```

The language has evolved significantly. Modern PHP looks respectable.

## The Type System

PHP started dynamically typed, added optional types, and now supports strict typing:

```php
<?php

// Dynamic typing (PHP's default)
$x = 42;
$x = "hello";    // Fine, variables can change types

// Type hints (PHP 7+)
function add(int $a, int $b): int {
    return $a + $b;
}

add(5, 3);       // 8
add("5", "3");   // 8 (strings coerced to ints in non-strict mode)

// Strict types (PHP 7+)
declare(strict_types=1);

function strictAdd(int $a, int $b): int {
    return $a + $b;
}

strictAdd(5, 3);     // 8
strictAdd("5", "3"); // TypeError!

// Union types (PHP 8+)
function process(int|string $value): void {
    // ...
}

// Nullable types
function find(int $id): ?User {
    return $user ?? null;
}

// Property types (PHP 7.4+)
class Person {
    public string $name;
    public int $age;
    private ?string $email = null;
}
```

Modern PHP developers increasingly use strict types. It's a different language from the wild west of early PHP.

## The Infamous Inconsistencies

PHP's standard library is notoriously inconsistent:

```php
// Needle-haystack order varies
strpos($haystack, $needle);     // String position
in_array($needle, $haystack);   // Array search

// Naming conventions are chaotic
strlen($str);           // No underscore
str_replace();          // Underscore
htmlspecialchars();     // No underscore, no separation

// Some functions modify arrays in place, others return new arrays
sort($array);           // Modifies in place, returns bool
array_reverse($array);  // Returns new array

// Case sensitivity
class MyClass {}
$obj = new myclass();   // Works! Class names are case-insensitive
function MyFunc() {}
MyFunc();              // Works! Function names are case-insensitive
```

This inconsistency is PHP's original sin. The language grew organically, and it shows.

## Superglobals and Web Integration

PHP was built for the web, and it shows:

```php
// Superglobals (available everywhere)
$_GET      // Query parameters
$_POST     // POST data
$_SERVER   // Server info
$_SESSION  // Session data
$_COOKIE   // Cookies
$_FILES    // Uploaded files

// Simple form handling
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'] ?? '';
    $email = $_POST['email'] ?? '';
    // Process form...
}

// Sessions
session_start();
$_SESSION['user_id'] = 42;

// Cookies
setcookie('name', 'value', time() + 3600);
```

This tight integration with HTTP made PHP incredibly easy for web development. No framework needed for simple tasks.

## The Security Reputation

Early PHP made it easy to write insecure code:

```php
// DON'T DO THIS (but people did)
$id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = $id";
// SQL injection vulnerability!

// Or this
echo $_GET['name'];
// XSS vulnerability!

// Or this
$file = $_GET['file'];
include($file);
// Remote code execution!
```

Modern PHP has better defaults and tools:

```php
// Prepared statements (SQL injection prevention)
$stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
$stmt->execute([$id]);

// HTML escaping (XSS prevention)
echo htmlspecialchars($_GET['name'], ENT_QUOTES, 'UTF-8');

// Modern frameworks handle this automatically
{{ $name }}  // Blade (Laravel) auto-escapes
```

The language got safer, but the reputation stuck.

## Object-Oriented PHP

PHP added OOP in PHP 5 and improved it in later versions:

```php
<?php

class Animal {
    protected string $name;

    public function __construct(string $name) {
        $this->name = $name;
    }

    public function speak(): string {
        return "Some sound";
    }
}

class Dog extends Animal {
    public function speak(): string {
        return "Woof!";
    }
}

// Interfaces
interface Drawable {
    public function draw(): void;
}

// Traits (mixins)
trait Timestampable {
    public DateTime $createdAt;

    public function touch(): void {
        $this->createdAt = new DateTime();
    }
}

class Post {
    use Timestampable;
}

// Abstract classes
abstract class Shape {
    abstract public function area(): float;
}

// PHP 8: Constructor property promotion
class Point {
    public function __construct(
        public float $x,
        public float $y
    ) {}
}
```

Modern PHP OOP is quite capable, though it came late compared to Java or C++.

## The Framework Revolution

Raw PHP is messy. Frameworks brought order:

**Laravel** (The Popular One):
```php
// Routes
Route::get('/users/{id}', [UserController::class, 'show']);

// Eloquent ORM
$users = User::where('active', true)->get();

// Blade templates
@foreach($users as $user)
    <p>{{ $user->name }}</p>
@endforeach
```

**Symfony** (The Enterprise Choice):
```php
// Controllers
class UserController extends AbstractController {
    #[Route('/users/{id}')]
    public function show(int $id): Response {
        // ...
    }
}
```

**Other frameworks**: CodeIgniter, CakePHP, Yii, Slim

Laravel in particular has made PHP development pleasant. Good documentation, elegant syntax, and a strong ecosystem have revitalized PHP's image.

## What PHP Is Best For

**WordPress sites**: The WordPress ecosystem is massive. Plugins, themes, custom development.

**Web applications**: PHP was built for this. With Laravel, it's genuinely good.

**Content management**: WordPress, Drupal, Joomla. PHP dominates CMS.

**E-commerce**: Magento, WooCommerce, Shopify (originally PHP).

**Shared hosting**: PHP runs on cheap shared hosting. Easy deployment.

**Rapid web development**: Get a site up quickly with minimal infrastructure.

## What PHP Is Worst For

**Command-line tools**: Possible, but Python or Go are better suited.

**Systems programming**: No low-level memory access. Use C, Rust, or Go.

**Real-time applications**: Not designed for long-running processes or WebSockets (though it can be done).

**Mobile apps**: Use Swift, Kotlin, or React Native.

**Data science**: Python owns this space.

**Desktop GUIs**: PHP wasn't designed for this.

## The Ecosystem

**Composer**: Package manager (like npm for Node.js). Made dependency management civilized.

**Packagist**: Central repository for PHP packages.

**Testing**: PHPUnit, Pest (modern testing framework)

**Code quality**: PHPStan, Psalm (static analysis), PHP-CS-Fixer (formatting)

**Frameworks**: Laravel, Symfony, Slim, Lumen

**CMS**: WordPress, Drupal, Joomla, Concrete5

**Hosting**: Cheap and ubiquitous. Every hosting provider supports PHP.

## Performance Evolution

PHP's performance has improved dramatically:

- **PHP 5**: Slow by modern standards
- **PHP 7** (2015): 2x faster than PHP 5
- **PHP 8** (2020): JIT compiler, even faster
- **PHP 8.3** (2023): Continued improvements

Modern PHP is fast. Benchmarks show it's competitive with Node.js and faster than Python for web workloads.

## The Community

**The WordPress Developers**: Build themes and plugins. Might not know much PHP outside WordPress.

**The Laravel Developers**: Modern PHP developers. Love Taylor Otwell (Laravel's creator) and Eloquent ORM.

**The Legacy Maintainers**: Dealing with ancient PHP 5 codebases. Dreaming of refactoring.

**The PHP Apologists**: "PHP is actually good now!" (They're not wrong.)

**The Haters**: Will never forgive PHP for its past sins. Mock it relentlessly.

### Common Phrases
- "It's not your father's PHP anymore"
- "Just use Laravel"
- "WordPress pays the bills"
- "Have you tried PHP 8?"
- "The documentation is actually really good"
- "But it works..." (defending legacy code)
- "We're modernizing... eventually"

## PHP 8: The Modern Era

PHP 8 brought significant improvements:

**JIT Compiler**: Faster performance for CPU-intensive tasks.

**Named arguments**:
```php
function create(string $name, int $age, bool $active = true) {}

create(name: 'Alice', age: 30);
create(age: 25, name: 'Bob', active: false);
```

**Attributes** (annotations):
```php
#[Route('/api/users')]
class UserController {}
```

**Match expression**: Better switch statement.

**Union types**: `function process(int|string $value)`

**Nullsafe operator**:
```php
$country = $user?->getAddress()?->getCountry();
```

**Constructor property promotion**:
```php
class Point {
    public function __construct(
        public float $x,
        public float $y
    ) {}
}
```

Modern PHP is genuinely nice to work with.

## Should You Learn PHP?

**Yes, if**:
- You're building web applications
- You work with WordPress (unavoidable)
- You want to freelance (tons of PHP work)
- You want easy deployment (shared hosting)
- You're using Laravel or Symfony

**No, if**:
- You're doing data science, ML, or systems programming
- You want to build mobile or desktop apps
- You prefer compiled, statically-typed languages

**Maybe, if**:
- You want to understand what powers much of the web
- You're open to giving it a fair shake despite its reputation

## The Verdict

PHP is the underdog that won despite itself. It's not elegant. It's not theoretically pure. It wasn't designed by academics or language theorists. It was hacked together by someone who just wanted to solve a problem.

And yet, it works. It works at massive scale (Facebook), it works for small businesses, and it works for the majority of websites on the internet.

Modern PHP (8.x) with Laravel or Symfony is actually a pleasant development experience. The language learned from its mistakes. The community matured. The tooling improved.

Is PHP the best language? No. Is it the most beautiful? Definitely not. But is it practical, employable, and capable? Absolutely.

The internet runs on PHP. That's not changing anytime soon. You might not love it, but you should probably respect it. And if you actually try modern PHP with a good framework, you might be pleasantly surprised.

PHP: Constantly improving, eternally underestimated, and quietly powering more of the internet than you'd think.

---

**Next**: [Chapter 8: Ruby - Happiness and Magic](08-ruby.md)
