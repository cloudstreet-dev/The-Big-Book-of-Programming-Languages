# Chapter 36: WebAssembly - The Browser's New Universal Runtime

## The Compilation Target for the Web

WebAssembly (abbreviated Wasm) was released in 2017 as a collaboration between Mozilla, Google, Microsoft, and Apple—a rare moment of agreement among browser vendors. It's a binary instruction format designed to run in web browsers alongside JavaScript at near-native speed.

The problem: JavaScript was the only language browsers could run. Developers wanted to use C++, Rust, Go, and others on the web. Previous attempts (Java applets, Flash, ActiveX) failed due to security issues, vendor lock-in, or poor performance.

The insight: Don't create a new high-level language. Create a low-level compilation target—a virtual instruction set that any language can compile to. Make it fast, safe, and portable.

By 2025, WebAssembly has exceeded its original scope. It runs in browsers, on servers, in edge computing, embedded systems, and blockchain smart contracts. It's become the universal binary format many hoped for.

## The Philosophy

**Fast execution**: Near-native performance. Compiled to machine code, no interpretation overhead.

**Safe execution**: Sandboxed, memory-safe. Can't escape the sandbox, can't access system resources without permission.

**Portable**: Runs on any platform with a Wasm runtime. Write once, run anywhere (for real this time).

**Compact binary format**: Smaller than JavaScript. Downloads faster, parses faster.

**Not a programming language**: WebAssembly is a compilation target. You compile C, Rust, Go, etc. to Wasm. You don't write Wasm by hand.

**Language-agnostic**: Any language can compile to Wasm. C/C++, Rust, Go, C#, Python, Ruby—all work.

The result: A universal runtime that brings decades of existing code to the web and beyond.

## What It Looks Like (WAT - WebAssembly Text Format)

WebAssembly is binary, but there's a human-readable text format (WAT) for debugging:

```wasm
(module
  (func $add (param $a i32) (param $b i32) (result i32)
    local.get $a
    local.get $b
    i32.add)
  (export "add" (func $add)))
```

But you don't write this. You compile from other languages:

### Compiling Rust to WebAssembly

```rust
// src/lib.rs
#[no_mangle]
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[no_mangle]
pub fn fibonacci(n: i32) -> i32 {
    if n <= 1 {
        n
    } else {
        fibonacci(n - 1) + fibonacci(n - 2)
    }
}
```

Compile:
```bash
cargo build --target wasm32-unknown-unknown --release
```

### Using WebAssembly in JavaScript

```javascript
// Load and instantiate WebAssembly module
WebAssembly.instantiateStreaming(fetch('module.wasm'))
  .then(obj => {
    const { add, fibonacci } = obj.instance.exports;

    console.log(add(5, 3));        // 8
    console.log(fibonacci(10));    // 55
  });

// Or with async/await
async function loadWasm() {
  const response = await fetch('module.wasm');
  const buffer = await response.arrayBuffer();
  const module = await WebAssembly.compile(buffer);
  const instance = await WebAssembly.instantiate(module);

  return instance.exports;
}

const wasm = await loadWasm();
console.log(wasm.add(10, 20));
```

### Compiling C to WebAssembly (Emscripten)

```c
// hello.c
#include <stdio.h>
#include <emscripten.h>

EMSCRIPTEN_KEEPALIVE
int calculate(int x, int y) {
    return x * y + x + y;
}

EMSCRIPTEN_KEEPALIVE
void greet(const char* name) {
    printf("Hello, %s!\n", name);
}
```

Compile with Emscripten:
```bash
emcc hello.c -o hello.html -s EXPORTED_FUNCTIONS='["_calculate","_greet"]' -s EXPORTED_RUNTIME_METHODS='["ccall","cwrap"]'
```

Use in JavaScript:
```javascript
// Emscripten provides Module object
const result = Module.ccall('calculate', 'number', ['number', 'number'], [5, 3]);
console.log(result);  // 5*3 + 5 + 3 = 23
```

## Passing Data Between JavaScript and Wasm

### Simple Types (Numbers)

```javascript
// Numbers pass directly
const result = wasmInstance.exports.add(5, 3);
```

### Strings (More Complex)

```rust
// Rust side
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}
```

```javascript
// JavaScript side (with wasm-bindgen)
import init, { greet } from './pkg/module.js';

await init();
const greeting = greet("Alice");
console.log(greeting);  // "Hello, Alice!"
```

### Arrays and Memory

```rust
// Rust: expose memory
#[no_mangle]
pub fn get_array_ptr() -> *const u8 {
    static ARRAY: [u8; 5] = [1, 2, 3, 4, 5];
    ARRAY.as_ptr()
}

#[no_mangle]
pub fn process_array(ptr: *mut i32, len: usize) {
    unsafe {
        let slice = std::slice::from_raw_parts_mut(ptr, len);
        for item in slice {
            *item *= 2;
        }
    }
}
```

```javascript
// JavaScript: access Wasm memory
const wasmMemory = new WebAssembly.Memory({ initial: 1 });
const instance = await WebAssembly.instantiate(module, {
  env: { memory: wasmMemory }
});

// Write array to Wasm memory
const array = new Int32Array(wasmMemory.buffer, 0, 5);
array.set([1, 2, 3, 4, 5]);

// Call Wasm function
instance.exports.process_array(array.byteOffset, array.length);

// Read result
console.log(array);  // [2, 4, 6, 8, 10]
```

## Real-World Example: Image Processing

```rust
// Rust: grayscale image processing
#[no_mangle]
pub fn grayscale(ptr: *mut u8, len: usize) {
    unsafe {
        let pixels = std::slice::from_raw_parts_mut(ptr, len);

        // Process RGBA pixels (4 bytes each)
        for chunk in pixels.chunks_exact_mut(4) {
            let r = chunk[0] as f32;
            let g = chunk[1] as f32;
            let b = chunk[2] as f32;

            // Grayscale formula
            let gray = (0.299 * r + 0.587 * g + 0.114 * b) as u8;

            chunk[0] = gray;
            chunk[1] = gray;
            chunk[2] = gray;
            // chunk[3] = alpha (unchanged)
        }
    }
}
```

```javascript
// JavaScript: use in canvas
const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

// Copy image data to Wasm memory
const wasmMemory = new Uint8Array(instance.exports.memory.buffer);
const imageOffset = instance.exports.allocate(imageData.data.length);
wasmMemory.set(imageData.data, imageOffset);

// Process with Wasm
instance.exports.grayscale(imageOffset, imageData.data.length);

// Copy back to canvas
imageData.data.set(wasmMemory.subarray(imageOffset, imageOffset + imageData.data.length));
ctx.putImageData(imageData, 0, 0);
```

## WebAssembly System Interface (WASI)

WASI extends WebAssembly beyond browsers to run on servers, CLIs, and embedded systems:

```rust
// Rust program using WASI
use std::fs;

fn main() {
    let contents = fs::read_to_string("input.txt")
        .expect("Failed to read file");
    println!("File contents: {}", contents);
}
```

Compile and run with Wasmtime:
```bash
cargo build --target wasm32-wasi
wasmtime target/wasm32-wasi/release/myapp.wasm
```

## The Good Parts

**Near-native performance**: 1-2x slower than native code. Much faster than JavaScript for compute-heavy tasks.

**Bring existing code to web**: Decades of C/C++ libraries can run in browsers. Games, image processors, scientific simulations.

**Multiple languages**: Write in C, C++, Rust, Go, C#, Python (via Pyodide), Ruby. Not limited to JavaScript.

**Safe sandboxing**: Memory-safe by design. Can't escape sandbox. Better than native plugins (Flash, Java applets).

**Fast load times**: Binary format is compact. Compiles to machine code quickly.

**Beyond browsers**: WASI enables server-side, edge computing, embedded systems. Universal runtime.

**Predictable performance**: No garbage collection pauses (unless your source language has GC). Deterministic.

**Growing ecosystem**: Libraries, toolchains, runtimes improving rapidly.

## The Pain Points

**Debugging is harder**: Browser devtools improving, but debugging Wasm isn't as smooth as JavaScript.

**Interop overhead**: Calling between JavaScript and Wasm has cost. Passing complex data structures is painful.

**No DOM access**: Wasm can't directly manipulate DOM. Must call JavaScript. Limits use cases.

**Binary size**: Compiled Wasm can be large. Needs careful optimization and tree-shaking.

**Still evolving**: Spec is stable, but features like threads, SIMD, GC are recent or upcoming additions.

**Toolchain complexity**: Compiling to Wasm adds build complexity. Cross-compilation quirks.

**Not always faster**: For DOM-heavy tasks, JavaScript is fine. Wasm shines in compute-heavy workloads.

## Use Cases

**High-performance web apps**: Games (Unity, Unreal), CAD software, video editing, music production.

**Image and video processing**: Filters, compression, encoding. Faster than JavaScript.

**Scientific computing**: Simulations, data analysis, visualization. Port existing tools.

**Cryptography**: Performance-critical hashing, encryption. Native speed in browsers.

**Code editors and IDEs**: VS Code uses Wasm for language servers, syntax highlighting.

**Emulation**: Run retro games, operating systems in the browser. DOSBOX, QEMU compiled to Wasm.

**Blockchain smart contracts**: Ethereum 2.0, Polkadot, others use Wasm for contracts.

**Server-side computing**: Edge computing (Cloudflare Workers), serverless functions, plugin systems.

**Portable CLI tools**: Compile once, run on any OS via Wasmtime or Wasmer.

## The Ecosystem

**Compilers**:
- Emscripten: C/C++ to Wasm, mature toolchain
- wasm-pack: Rust to Wasm, great developer experience
- TinyGo: Go to Wasm (standard Go too large)
- AssemblyScript: TypeScript-like language for Wasm

**Runtimes**:
- Browsers: Chrome, Firefox, Safari, Edge (all support Wasm)
- Wasmtime: Standalone runtime, fast and secure
- Wasmer: Universal runtime, supports WASI
- WasmEdge: Edge computing runtime

**Tools**:
- wasm-bindgen: Rust/JavaScript interop
- wabt: WebAssembly Binary Toolkit (wat2wasm, wasm2wat)
- wasm-opt: Optimize Wasm binaries (part of Binaryen)

**Languages targeting Wasm**:
- C/C++, Rust, Go, C#/.NET, Python (Pyodide), Ruby (ruby.wasm), Swift, Kotlin

## Common Phrases You'll Hear

**"Compile to Wasm"**: WebAssembly is a compilation target, not a source language.

**"Near-native speed"**: Usually 1-2x slower than native. Way faster than JavaScript for compute.

**"The fourth language of the web"**: HTML, CSS, JavaScript, and now WebAssembly.

**"WASI extends beyond browsers"**: WebAssembly System Interface brings Wasm to servers and CLIs.

**"Interop is the hard part"**: Calling between JavaScript and Wasm, passing data—that's where complexity lives.

**"It's not replacing JavaScript"**: Wasm complements JavaScript for performance-critical code, doesn't replace it.

## The Verdict

WebAssembly is the most important development in web technology since JavaScript. It brings performance, portability, and language choice to the browser—and increasingly, beyond.

**Use WebAssembly when**:
- You have performance-critical code (image processing, games, simulations)
- You want to reuse existing C/C++/Rust libraries in the browser
- You need predictable, near-native performance
- You're building edge computing or serverless functions
- You want a universal binary format

**Don't use WebAssembly when**:
- Your code is DOM-heavy (JavaScript is fine)
- You don't need performance (JavaScript is easier)
- You're prototyping quickly
- Interop overhead outweighs performance gains

WebAssembly isn't replacing JavaScript. It's complementing it. Use JavaScript for UI, business logic, and DOM manipulation. Use WebAssembly for compute-intensive tasks.

The vision: A universal runtime where any language can run anywhere—browsers, servers, edge, embedded. By 2025, that vision is becoming reality.

WebAssembly proved that browsers don't have to be JavaScript-only. Performance doesn't require native plugins. Safety and speed can coexist.

For web developers, Wasm opens doors. That C++ game engine? Port it. That Rust image library? Use it. That Python data science tool? Run it in the browser.

Beyond the web, WASI is turning WebAssembly into a universal binary format for cloud computing, edge functions, and plugin systems. Write once in any language, run securely anywhere.

WebAssembly is the compilation target that makes the "write once, run anywhere" promise actually work.

---

**Next**: [Chapter 37: Choosing Your Weapon](37-choosing.md)
