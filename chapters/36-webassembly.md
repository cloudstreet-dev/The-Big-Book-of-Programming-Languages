# Chapter 36: WebAssembly - The Browser's New Universal Runtime

## The Compilation Target for the Web

WebAssembly (Wasm) was released in 2017 as a collaboration between Mozilla, Google, Microsoft, and Apple. It's a binary instruction format that runs in web browsers alongside JavaScript at near-native speed.

The key insight: Instead of making browsers run only JavaScript, create a low-level target that any language can compile to. C, C++, Rust, Go, and others can now run in the browser.

## The Philosophy

**Fast execution**: Near-native performance in the browser.

**Safe**: Sandboxed execution, memory-safe.

**Portable**: Runs on any platform with a Wasm runtime.

**Compact**: Binary format, smaller than JavaScript.

**Compilation target**: Not written by hand, compiled from other languages.

## What It Looks Like (WAT - WebAssembly Text Format)

```wasm
(module
  (func $add (param $a i32) (param $b i32) (result i32)
    local.get $a
    local.get $b
    i32.add)
  (export "add" (func $add)))
```

But you don't write this. You compile C/Rust/etc. to Wasm:

```rust
// Rust code
#[no_mangle]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// Compile to WebAssembly
// cargo build --target wasm32-unknown-unknown
```

Use in JavaScript:
```javascript
WebAssembly.instantiateStreaming(fetch('module.wasm'))
  .then(obj => {
    const result = obj.instance.exports.add(5, 3);
    console.log(result);  // 8
  });
```

## The Verdict

WebAssembly isn't a language you write—it's a compilation target. It enables:
- High-performance web applications (games, video editing, CAD)
- Running existing C/C++ code in browsers
- Bringing Rust, Go, and other languages to the web
- Server-side use (WASI - WebAssembly System Interface)

By 2025, Wasm is running in browsers, on servers, in edge computing, and embedded systems. It's the universal runtime beyond just the web.

---

**Next**: [Chapter 37: Choosing Your Weapon](37-choosing.md)
