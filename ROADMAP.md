# 🗺️ Terra (Tesseract Compiler) Roadmap

> **Terra** is a compiler for the Tesseract language — a prototype modern compilation system written in C23.  
> *This roadmap outlines the development path from version 0.1 to 1.0 and beyond.*

---

## 📌 Executive Summary

### Core Principles

Every release adheres to these principles:

1. **Nothing gets slower** — performance must match or exceed previous releases
2. **Measure against rivals** — benchmarked against GCC, Clang on identical hardware
3. **Backward compatibility** — old programs continue working without modifications
4. **Production-ready quality** — stability, documentation, and testing at every stage

---

## 🔍 Work Priorities

| Priority | Area | Description |
|----------|------|-------------|
| 🔥 **Critical** | Compilation Speed | Fast compilation for projects from 1K to 100K LOC |
| ⚡ **High** | Code Generation | Multiple backends: C99, LLVM IR, WebAssembly |
| 🌐 **High** | Cross-Platform Support | x86_64, ARM64, RISC-V with automated CI |
| 🛠️ **Medium** | IDE Integration | LSP server, debugger, profiler, package manager |

---

## 🗂️ Small Tasks Queue (`road.txt`)

`road.txt` is the backlog of small, already-scoped tasks: targeted fixes and
low-risk additions that do not need an RFC or a milestone of their own. Every
entry is a single actionable line written as `[] - <task>`, and each should be
small enough for one commit.

Rules of the queue:

- An item is added only when the problem is already understood and the fix is
  local (one file, or a few closely related ones).
- Items are taken from the top; mark a finished item by changing `[]` to `[x]`.
- The file is a queue, not a roadmap: anything that changes semantics, public
  syntax or the architecture still goes through the phases below.
- Keep the list short (roughly a dozen open items); promote larger entries into
  the appropriate phase instead of letting them pile up here.

---

## ✅ Completed Releases

| Version | Highlights | Status |
|---------|------------|--------|
| **0.0.1** | Lexer + Parser + Sema, CLI, demo examples | 🟢 Released |
| **0.0.2** | Error diagnostics with location info, test suite foundation | 🟢 Released |
| **0.1** | AST optimization passes, initial C99 codegen | 🟢 Released |
| **0.3.0** | C99 code generation, modular backend architecture, language syntax updates | ✅ Complete |
| **0.4.0** | Native C headers, clearer declarations, application-oriented control flow and executable builds | ✅ Complete |
| **0.5.0 (Current)** | Standard library: Python-style string methods, validated math module, `--safe` runtime checking | 🏗️ In Progress |

---

## 📅 Development Phases (0.4 → 1.0)

Each phase builds upon the previous one while maintaining API/ABI stability.

### Standard library and safe runtime (active direction, 0.5)

- [x] Builtin string methods on string receivers, validated in sema and
  dispatched to selective `terra_str_*` runtime helpers: `len at find count
  is_empty contains starts_with ends_with upper lower trim replace reverse
  repeat substr`, including method chaining (`s.trim().upper()`).
- [x] Validated native `math` module surface (sqrt, pow, floor, ceil, round,
  fabs/abs, fmin/fmax, exp, log) with compile-time rejection of unknown
  members instead of link errors.
- [x] `--safe` mode: runtime-checked integer arithmetic (overflow, division
  by zero) and bounds-checked array/slice indexing via `terra_panic`; always
  on for `str.at()`.
- [x] Length-aware `put_len` for string literals, preserving embedded NULs.
- [x] Constant division/modulo-by-zero rejected at compile time.
- [ ] Typed dynamic collections (`terra_list` with element types, push/pop/
  get/set/insert/remove/find/sort) built on the existing slice machinery.
- [ ] String formatting (`format`, `join`, `split`) and number parsing with
  error reporting instead of silent zeros.
- [ ] Safe-mode coverage for more expression forms (compound assignment,
  method call arguments) and a benchmark comparing safe vs. unchecked code.

### Native C and application support (active direction)

- [x] Import validated system headers with `import "header.h" as alias`.
- [x] Declare typed C functions with `extern fn`, including C-style variadic
  signatures and opaque C typedefs, then emit prototypes for the C compiler to
  validate.
- [x] Add explicit `let name = value;` declarations while retaining assignment
  based declarations for existing programs.
- [x] Add a typed Tesseract `fn main()` application entry point while retaining
  top-level script execution for examples and quick programs.
- [x] Add scoped C-style `for (init; condition; step)` loops.
- [x] Add loop-scoped `break;` and `continue;` with semantic validation.
- [x] Add top-level `link "library";` metadata and a shell-free executable
  build mode that passes library names to the C linker.
- [ ] Add typed `extern` declarations for C APIs that do not provide headers or
  need narrower language-level types.
- [ ] Expand application primitives (collections, iteration, modules, and a
  stable runtime API) based on small real programs and examples.

### Freestanding and operating-system work (required before a Linux 1.0 port)

The compiler now has a source-to-object freestanding path for runtime-free
programs. It still does not link a kernel or produce a bootable image.

- [x] Fixed-size arrays, typed slices, indexing, pointer arithmetic, and
  explicit address/dereference operations.
- [x] C ABI `struct`/`union` layout and `volatile` qualified types.
- [x] Freestanding object mode without hosted runtime includes or link steps;
  programs that require Terra's hosted helpers are rejected in this mode.
- [x] Target triple forwarding and GNU-style inline assembly templates.
- [ ] Complete function-pointer types, calling-convention attributes,
  assembly constraints/intrinsics, linker scripts, and reproducible boot-image
  builds.
- [ ] Architecture-specific integration tests and boot tests in an emulator.

---

## 🛡️ Zero Trust C99 Backend & Compiler Hardening (cross-cutting track)

C99 is not a portable assembler. Signed overflow, invalid shifts, signed
division (`INT_MIN / -1`), float-to-int casts, uninitialized reads, unread
padding bytes, strict-aliasing and data races are Undefined Behavior, and an
optimizing C compiler is allowed to delete the very checks meant to prevent
them. This track makes the generated C99 a mathematically UB-free subset, and
makes the compiler's own pipeline distrustful: no pass trusts a previous pass.

Small, local items from this track are also queued in `road.txt`; the bulk is
planned here because it changes codegen, IR and language semantics.

### 1. Defensive C99 code generation

- [ ] Lower every expression to ANF/TAC: one side effect per generated C
  statement, explicit temporaries, and no reliance on C's unspecified argument
  and operand evaluation order (`f(g(), h())`, `a[i] = i++`).
- [ ] Route all signed `+ - * / % << >>` through checked runtime helpers
  (`zl_add_i32`, `zl_div_i32`, `zl_shl_u32`, ...) covering overflow,
  division-by-zero, `INT_MIN / -1` and shift-amount bounds; use
  `__builtin_*_overflow` on GCC/Clang with a pure-C99 unsigned fallback.
- [ ] Check every `double -> int` conversion for NaN and out-of-range values
  per ISO C99 §6.3.1.4 (`zl_f64_to_i32`) instead of a raw cast.
- [ ] Represent every array/buffer/string crossing generated code as a
  bounds-carrying descriptor (`{ptr, len, cap}`), never a bare `T*`; add
  `restrict` where the language proves there is no mutable aliasing.
- [ ] Zero-initialize every generated local (`= {0}`) and turn hidden struct
  padding into explicit zero-filled `uint8_t _padN[...]` fields so padding
  bytes never leak through `memcmp` or I/O.
- [ ] Add `zl_secure_zero` backed by a `volatile` loop and an asm memory
  barrier so Dead Store Elimination cannot remove secret wiping.
- [ ] Emit C99 static assertions (negative-array-size trick) for
  `sizeof(void*)` and every generated struct/record against the compiler's
  computed layout, to catch ABI drift on other targets.
- [x] Never emit a recoverable panic: `terra_panic` is `noreturn` and
  terminates via `abort()`; no `setjmp`/`longjmp`-based recovery.
- [ ] Emit `default: zl_panic("unreachable enum state")` in every generated
  `switch` over enums/unions.
- [ ] Add a forced, non-overridable compiler-flag profile for `--compile`:
  `-std=c99 -fwrapv -fno-strict-aliasing -fno-delete-null-pointer-checks
  -fstack-protector-strong -fstack-clash-protection -D_FORTIFY_SOURCE=3
  -fPIE -pie`, plus an opt-in debug profile with
  `-fsanitize=undefined,address,leak -fno-omit-frame-pointer`.
- [ ] Define the concurrency story: C99 has no memory model, so shared
  variables must use `__atomic_load_n`/`__atomic_store_n`/
  `__atomic_compare_exchange_n` with `__ATOMIC_SEQ_CST`; `volatile` is not a
  synchronization primitive.

### 2. FFI isolation (Zero Trust boundary with native C)

- [ ] Introduce FFI-only types: `c_ptr<T>`, `c_str`, and an `@c_layout`
  attribute; language structs and slices are never passed to C directly.
- [ ] Deep-copy or pin/freeze slices and buffers across the FFI boundary
  instead of exposing live language-managed memory.
- [ ] Mark every value returned from `extern "C"` as `tainted<T>`; forbid
  using it as an index, allocation size, or dereference target until sanitized.
- [ ] Require an explicit `sanitize(value) with (predicate) else { ... }`
  gate (Sanitization Contract) to strip `tainted`.
- [ ] Generate a trampoline around each FFI call: pre-call invariant checks,
  stack-canary/metadata snapshot, and a post-call corruption check that panics.
- [ ] Capability-based FFI: allow-list native symbols via a project manifest
  (`allow_ffi = [...]`); a transitive dependency without a granted token is a
  compile error.

### 3. Zero Trust compiler internals (IR and analyses)

- [ ] Introduce an explicit HIR → typed-SSA MIR → C-AST pipeline; no C99
  emission through string concatenation or `sprintf`.
- [ ] Make safety operations first-class IR nodes: `check_bounds` produces a
  proof token consumed by `get_element_unchecked`; the unchecked node must not
  verify without a dominating proof. Optimizations may only replace a check
  with a `static_proof_interval`, never delete it.
- [ ] Add an independent IR verifier after every optimization pass that fails
  with an Internal Compiler Error instead of emitting vulnerable C.
- [ ] Definite-assignment dataflow: prove no CFG path reads a variable in the
  `Uninit` or `Moved` state.
- [ ] Interval/value-range analysis feeding safety proofs and turning provably
  safe checks into tokens; make provable overflow and division-by-zero
  compile-time errors.
- [ ] Escape/lifetime analysis: no reference to a stack local may escape via
  return, a heap/global store, or a thread.
- [ ] Totality/exhaustiveness checking for `match`/`switch` over enums and
  tagged unions.
- [ ] Call-graph-based stack-frame bounding with a recursion/indirect-call
  depth guard (`++stack_depth > MAX_DEPTH -> panic`).
- [ ] Deterministic name mangling for all emitted C identifiers to avoid C
  keywords, reserved `_`/`__` names and standard-header macros.
- [ ] Typed C-AST emitter in which a variable node cannot exist without an
  initializer.

### 4. Design by Contract (DbC)

- [ ] Add `requires`, `ensures`, `invariant`, `old(expr)` and `result` syntax,
  parsed into the AST.
- [ ] Enforce a Pure Expression Check on all contract expressions: no
  mutation, no non-pure calls, no I/O or FFI.
- [ ] Feed `requires`/`invariant` facts into interval analysis so provably
  safe operations in the body drop their runtime checks (static promotion);
  insert call-site checks only when the caller cannot prove them.
- [ ] Lower contracts in C99 with a single epilogue: replace every `return`
  with `_zl_ret_val = ...; goto _zl_epilogue;`, snapshot `old(...)` in the
  prologue after `requires`, and check invariants plus `ensures` at the
  epilogue.
- [ ] Check struct invariants on creation and after every mutating method.

### 5. Remaining blind spots

- [ ] Temporal memory safety: choose and document a model (generational
  references with `{ptr, expected_gen}`, linear/affine ownership, or ARC
  without cycles), since bounds-checked fat pointers only provide spatial
  safety.
- [ ] Define the safe-error story: `Result<T, E>` as a tagged union for
  recoverable failures, while panics remain fatal and non-recoverable.
- [ ] Treat FFI that can `longjmp` or throw across the C ABI as unsupported
  and document the quarantine requirement.
- [ ] Add sanitizer/static-analysis CI jobs (UBSan, ASan, LSan, `-fanalyzer`)
  over all generated C and examples.

---

### Phase X: Version 0.4 — Runtime Library & Executable Programs

#### 🎯 Objectives

Create a complete runtime library and ensure Terra programs compile to fully executable C99 code.

#### ✨ Features

##### 1. C99 Runtime Library (`terra_runtime.h`)

```c
// Memory management
void* terra_malloc(size_t bytes);
void terra_free(void* ptr);

// Standard I/O
void put(int value);
void put_str(const char* str);
int input_int();
char* input_str();

// Type conversion
int to_int(double d);
double to_float(int i);
const char* to_string(int i);

// String manipulation (future)
const char* concat(const char* a, const char* b);
int len(const char* str);
```

##### 2. Object Lifecycle Management

**Rules:**
- Objects are allocated on heap at creation time
- No garbage collection during program execution  
- `let B = A` makes B reference the same object as A
- No manual deallocation in current version

**AST Representation:**
```tesseract
let obj = ClassName.init;      // Allocation happens in init()
let ref = obj;                  // Reference copy
obj.method(args);               // Method call via pointer
```

##### 3. Function Definitions

**New syntax for functions:**
```tesseract
fn main() {
    let x = 10;
    let y = 20;
    put(x + y);
}

fn multiply(a: int, b: int): int {
    return a * b;
}
```

Benefits:
- Clear distinction between function declarations and calls
- Consistent type notation (return type after parameters)
- Easier parsing and better readability

#### 📋 Deliverables

- [ ] `terra_runtime.h` - Core runtime functions
- [ ] Memory allocation/deallocation implementation
- [ ] Standard I/O integration
- [ ] Object creation semantics documentation
- [ ] Function syntax parser support
- [ ] Complete hello_world example compilation

#### 📊 Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| Hello world compilation | 100% | Compiles to executable binary |
| Basic arithmetic ops | 100% | All examples run correctly |
| Runtime performance | ≤ 2x | vs native GCC O2 |
| Binary size overhead | ≤ 3x | vs equivalent C code |
| Memory safety | 0 leaks | Valgrind clean |

---

### Phase 3: Version 0.5 — Optimizations & Multi-Backend Support

#### 🎯 Objectives

Apply optimization passes and enable multiple target backends.

#### ✨ Optimization Passes

| Optimization | Priority | Description | Status |
|--------------|----------|-------------|--------|
| Constant Folding | High | Evaluate constant expressions | 🔵 Planned |
| Dead Code Elimination | High | Remove unreachable code | 🔵 Planned |
| Inline Expansion | Medium | Embed small functions | 🟡 Next |
| Loop Unrolling | Medium | Expand loop iterations | 🔮 Future |

#### ✨ Multi-Backend Architecture

**Supported Backends:**
- **C99 Backend** (✅ Implemented) - Production-ready
- **LLVM IR Backend** (🔵 Planned) - Max performance
- **WebAssembly Backend** (🔮 Future) - Web/edge deployment

**Backend Interface:**
```c
backend_codegen* backend_create_c99(config);
backend_codegen* backend_create_llvm(config);
backend_codegen* backend_create_wasm(config);

bool backend_generate(backend, ast, output_buffer);
```

Usage:
```bash
terra build --emit c99 program.t -o program.c
clang -O2 program.c -o program

terra build --emit llvm program.t -o program.ll
opt -O3 program.ll -o program_opt.ll
clang program_opt.ll -o program

terra build --emit wasm program.t -o program.wasm
node program.wasm
```

#### 📋 Deliverables

- [ ] AST optimization pipeline
- [ ] LLVM IR backend (via llvm-capi)
- [ ] WASM binary format writer
- [ ] Cross-compilation toolchain support
- [ ] Backend abstraction layer tests

#### 📊 Success Criteria

| Metric | Target | Platform |
|--------|--------|----------|
| LLVM IR validity | Passes `opt -O3` | All LLVM versions |
| WASM execution | Runs in browser | Chrome, Firefox, Edge |
| WASM execution | Runs in Node.js | v18+ |
| Performance (LLVM) | ≥ 1.5x better | vs C99 backend |

---

### Diagnostics & Test Infrastructure (0.2)

#### 🎯 Objectives

Enhance error reporting quality and establish comprehensive testing infrastructure.

#### ✨ Features

##### 1. Enhanced Diagnostic System

Multi-line error messages with context and suggestions:

```text
error[E001]: type mismatch in assignment
  ┌─ file.t:10:5
  │
10  │   x = "hello"
  │       ─────── expected int, found str
  │
help: did you mean str(x)?
  │
10  │   x = str("hello")
  │       ~~~~~        ~
```

##### 2. Test Infrastructure

- Unit tests for lexer, parser, sema (≥90% coverage)
- Integration test suite (examples that must compile)
- Snapshot testing for regression detection
- Benchmark harness for compiler performance metrics

#### 📋 Deliverables

- [ ] Advanced diagnostic system v2 (based on `core_diagnostics`)
- [ ] Unit tests for all frontend components
- [ ] Integration test suite covering all examples
- [ ] Benchmarking framework with CI integration
- [ ] Contributing guide and testing best practices docs

#### 📊 Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| Time to first error | ≤ 10ms | Files up to 10K LOC |
| Code coverage | ≥ 85% | Critical paths only |
| Test execution time | ≤ 2min | Full suite on CI |
| Example pass rate | 100% | All examples compile |

---

### Phase 2: Version 0.3 — C99 Codegen Backend

#### 🎯 Objectives

Generate production-quality C99 code with runtime library support.

#### ✨ Features

##### 1. C99 Code Generation Engine

Translation from Tesseract to ANSI C99:

**Tesseract Source:**
```tesseract
Greeter {
    name: str;
    
    greet(target: str) {
        put("Hello, " + target + "!");
    }
}

g = Greeter.new;
g.name = "Alice";
g.greet("world");
```

**Generated C99:**
```c
typedef struct Greeter {
    char* name;
} Greeter;

void Greeter_greet(Greeter* self, char* target) {
    put("Hello, ");
    put(target);
    put("!");
}

int main(int argc, char** argv) {
    Greeter* g = (Greeter*)malloc(sizeof(Greeter));
    g->name = malloc(6);
    strcpy(g->name, "Alice");
    Greeter_greet(g, "world");
    return 0;
}
```

##### 2. Runtime Library

Memory management and standard functions:

- Arena allocator for efficient allocation
- Reference counting for objects
- Standard library: `std.put()`, `std.assert()`, `len()`, `input()`
- Linker scripts for executable creation

#### 📋 Deliverables

- [ ] C99 code generation engine
- [ ] Type representation in C (primitives, classes, arrays)
- [ ] Method dispatch implementation
- [ ] Memory management runtime (arena + refcount)
- [ ] Standard library `std.*` module
- [ ] Linker scripts for executable creation

#### 📊 Success Criteria

| Metric | Target | Notes |
|--------|--------|-------|
| Compilation success | 100% | All semantically valid programs |
| Performance penalty | ≤ 2x | Compared to native GCC O2 |
| Binary size overhead | ≤ 3x | vs equivalent manual C code |
| Memory safety | 0 leaks | Valgrind clean |

---

### Phase 3: Version 0.5 — Optimizations & Performance

#### 🎯 Objectives

Apply optimization passes to improve runtime performance and binary size.

#### ✨ Optimization Passes

| Optimization | Priority | Description | Status |
|--------------|----------|-------------|--------|
| Constant Folding | High | Evaluate constant expressions at compile time | 🔵 Planned |
| Dead Code Elimination | High | Remove unreachable code blocks | 🔵 Planned |
| Inline Expansion | Medium | Embed function bodies directly | 🟡 Next |
| Loop Unrolling | Medium | Expand loop iterations | 🔮 Future |
| Strength Reduction | Low | Replace expensive ops with cheaper ones | 🔮 Future |

#### ✨ Example: Automatic Optimizations

```tesseract
// Optimizations applied automatically
result = expensive_function(a, b) + 0;  // +0 eliminated
if (true) { /* dead branch removed */ }
return compute(x);  // inlined if small enough
```

#### 📋 Deliverables

- [ ] AST optimization pipeline
- [ ] Simple IR for cross-pass analysis
- [ ] Constant propagation engine
- [ ] Unused code detection and removal
- [ ] Function inlining heuristics
- [ ] Benchmark comparison vs unoptimized build

#### 📊 Success Criteria

| Metric | Target | Impact |
|--------|--------|--------|
| Runtime speedup | ≥ 1.2x | Optimized vs non-optimized |
| Binary size reduction | ≥ 15% | After optimization passes |
| Compilation overhead | ≤ 20% | Extra time for optimizations |
| Optimization coverage | ≥ 80% | Identifiable opportunities |

---

### Phase 4: Version 0.5 — Multi-Target Support

#### 🎯 Objectives

Support multiple compilation targets for different use cases.

#### 🎯 Target Backends Comparison

| Backend | Use Case | Priority | Timeline |
|---------|----------|----------|----------|
| C99 | General purpose | High | ✅ Done |
| LLVM IR | Max performance | High | 🟡 Next |
| WebAssembly | Web/edge | Medium | 🏗️ Later |
| Native (x86) | Raw speed | Low | 🔮 Future |

#### ✨ Features

**LLVM IR Backend:**
```bash
terra compile --emit llvm-ir program.t -o program.ll
clang --target=x86_64-linux-gnu -O3 program.ll -o program
```

**WebAssembly Backend:**
```bash
terra compile --emit wasm program.t -o program.wasm
node --wasm program.wasm
```

**Cross-Compilation:**
```bash
# Linux → Windows
terra compile --target x86_64-pc-windows-gnu program.t

# Linux → macOS
terra compile --target x86_64-apple-darwin program.t

# Linux → ARM
terra compile --target aarch64-linux-gnu program.t
```

#### 📋 Deliverables

- [ ] LLVM IR code generation (via llvm-capi)
- [ ] WebAssembly binary format writer
- [ ] Cross-compilation toolchain support
- [ ] Runtime JIT for dynamic compilation
- [ ] Backend abstraction layer (common interface)

#### 📊 Success Criteria

| Metric | Target | Platform |
|--------|--------|----------|
| LLVM IR validity | Passes `opt -O3` | All LLVM versions |
| WASM execution | Runs in browser | Chrome, Firefox, Edge |
| WASM execution | Runs in Node.js | v18+ |
| Cross-compilation | Successful binaries | Linux→Win, Mac, ARM |
| Performance (LLVM) | ≥ 1.5x better | vs C99 backend |

---

### Phase 5: Version 0.6 — Standard Library & Package Manager

#### 🎯 Objectives

Provide comprehensive standard library and dependency management.

#### 📦 Standard Library Modules

| Module | Description | Status |
|--------|-------------|--------|
| `std.io` | Input/output, file operations | 🟡 Planned |
| `std.string` | String manipulation | 🟡 Planned |
| `std.collections` | Arrays, lists, maps | 🟡 Planned |
| `std.math` | Math functions | 🟡 Planned |
| `std.datetime` | Date/time handling | 🔮 Future |
| `std.net` | Network protocols | 🔮 Future |

#### 💼 Package Manager (`terrapkg`)

**Declaration (`terrapkg.toml`):**
```toml
[package]
name = "my-project"
version = "0.1.0"

[dependencies]
stdlib = ">=0.6.0"
http-client = "1.2.3"
json-parser = "^2.0"

[dev-dependencies]
test-framework = "latest"
```

**Module System:**
```tesseract
import std.io
import std.string
from std.math import sqrt, abs

use std.logging { level: Debug }

fn main() {
    let path = std.io.read_line();
    put("Hello, " + path + "!");
}
```

#### 📋 Deliverables

- [ ] `std.io` module (read, write, file I/O)
- [ ] `std.string` module (concat, split, replace)
- [ ] `std.collections` module (Array, HashMap, LinkedList)
- [ ] Module loader and resolver
- [ ] Package manager CLI (`terrapkg`)
- [ ] Public package registry MVP

#### 📊 Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| Std lib coverage | 100% | Core functionality |
| Dependency resolution | ≤ 1s | Projects with 10 packages |
| Build time improvement | ≥ 30% | With caching enabled |
| Offline mode | Functional | Local cache only |

---

### Phase 6: Version 0.7 — Concurrency & Parallelism

#### 🎯 Objectives

Enable parallel compilation and concurrent program execution.

#### ⚡ Concurrency Model

**Parallel Execution:**
```tesseract
let results = async_map([1, 2, 3, 4], |x| {
    expensive_computation(x)
}, threads: 4);
```

**Async/Await Pattern:**
```tesseract
async fn fetch_data(url: str) -> str {
    let response = http_get(url).await;
    return parse(response);
}
```

**Channels for Communication:**
```tesseract
channel = make_channel[int](buffer: 10);
spawn send_to(channel);
let value = channel.recv().await;
```

#### 📋 Deliverables

- [ ] Thread pool implementation
- [ ] Task scheduler for work-stealing
- [ ] Async/await compiler support
- [ ] Channel-based communication primitives
- [ ] Concurrent data structures (ConcurrentQueue, AtomicRef)
- [ ] Race condition detector (thread sanitizer)

#### 📊 Success Criteria

| Metric | Target | Scaling |
|--------|--------|---------|
| Compile speedup | ≥ 2x | -jN flag for N cores |
| Runtime scaling | Linear | Up to 8 cores |
| Data races | 0 detected | Thread sanitizer |
| Async overhead | ≤ 5% | vs sync baseline |

---

### Phase 7: Version 0.8 — Advanced Type System

#### 🎯 Objectives

Add advanced type features for expressive and safe code.

#### 🔷 Type System Features

**Generics with Constraints:**
```tesseract
class List[T] {
    items: Array[T];
    
    fn append(item: T) {
        items.push(item);
    }
    
    fn get(index: int) -> T {
        return items[index];
    }
}

// Usage
let int_list = List[int].new();
let str_list = List[str].new();
```

**Type Inference:**
```tesseract
let inferred = 42;              // inferred as int
let string_val = "hello";       // inferred as str
let array = [1, 2, 3];          // inferred as Array[int]
```

**Union Types:**
```tesseract
type Result = success[str] | error[std.error.Error];

fn parse(text: str) -> Result {
    if text == "" {
        return error(ParseError::Empty);
    }
    return success(text.trim());
}
```

**Traits (Interfaces):**
```tesseract
trait Printable {
    fn print() -> str;
}

impl Printable for Int {
    fn print() -> str {
        return str(self.value);
    }
}
```

#### 📋 Deliverables

- [ ] Generic type parameter system
- [ ] Type inference engine (Hindley-Milner style)
- [ ] Union type representation and pattern matching
- [ ] Trait system with automatic impl resolution
- [ ] Higher-kinded types (optional)
- [ ] Type-level programming basics

#### 📊 Success Criteria

| Metric | Target | Notes |
|--------|--------|-------|
| Inference coverage | ≥ 80% | Code without annotations |
| Binary size increase | ≤ 20% | With generics |
| Trait resolution time | ≤ 100ms | For large projects |
| Type safety | 100% | No runtime type errors |

---

### Phase 8: Version 0.9 — Tooling & Developer Experience

#### 🎯 Objectives

Provide excellent developer experience with integrated tools.

#### 🛠️ Tooling Ecosystem

| Tool | Purpose | Status |
|------|---------|--------|
| `terra-lsp` | Language Server Protocol | 🟡 Planned |
| `terra-dbgr` | Debugger (GDB integration) | 🔮 Future |
| `terra-profil` | Runtime profiler | 🔮 Future |
| `terra-fmt` | Code formatter | 🟡 Planned |
| `terra-lint` | Static analysis | 🟡 Planned |

#### ✨ LSP Features

- Go to definition/references
- Auto-completion (IntelliSense)
- Hover documentation
- Find symbols (workspace-wide search)
- Rename symbol across files
- Diagnostics streaming on-save
- Workspace symbols

#### ✨ Formatter Rules

```tesseract
// terra fmt applies automatically
Class Example {
    field: int;  // single space after colon

    init(value: int) {
        self.field = value;
    }

    compute(other: int) -> int {
        return self.field + other;
    }
}
```

#### 📋 Deliverables

- [ ] LSP server implementation (language-server.org spec)
- [ ] Editor plugins (VS Code extension, Neovim plugin)
- [ ] Code formatter (deterministic, idempotent)
- [ ] Linter with style and security rules
- [ ] Debugger protocol (DAP integration)
- [ ] Profiler UI (flame graphs, hot spots)

#### 📊 Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| LSP conformance | ≥ 90% | Official LSP test suite |
| Formatting speed | ≤ 500ms | File with 1K lines |
| Debugger features | Complete | Breakpoints, step, watch |
| Lint accuracy | ≥ 80% | Common errors caught |

---

### Phase 9: Version 1.0 — Stability & Production Readiness

#### 🎯 Objectives

Achieve production-ready status with stable API/ABI.

#### 🔒 Stability Commitments

1. **API Stability**: All public symbols have ABI-version scripts
2. **ABI Compatibility**: Old binaries work with new compiler versions
3. **Deprecation Policy**: 2 major release warning before removal
4. **Changelog**: Every change documented with migration guides

#### ✅ Production Checklist

- [ ] API surface frozen and fully documented
- [ ] All benchmarks meet targets vs competitors
- [ ] Coverage tests ≥ 95% for critical paths
- [ ] Security audit completed
- [ ] Performance regression tests in CI
- [ ] Migration guides for every breaking change
- [ ] Production-ready documentation website

#### 📊 Benchmarks Against Competitors

| Metric | Terra | GCC | Clang | Notes |
|--------|-------|-----|-------|-------|
| Compile speed | Baseline | +X% | +Y% | Measured on 100K LOC |
| Binary size | Baseline | ±Z% | ±W% | Optimized (-O3) |
| Runtime perf | Baseline | ±A% | ±B% | Same optimization level |

#### 📋 Deliverables

- [ ] Version 1.0 release with stable API/ABI
- [ ] Comprehensive user guide (100+ pages)
- [ ] API reference documentation
- [ ] Tutorial series (beginner to advanced)
- [ ] Cookbook of common patterns
- [ ] Community contribution guidelines
- [ ] Official website with download mirrors

#### 📊 Success Criteria

| Criterion | Target | Verification |
|-----------|--------|--------------|
| Semantic versioning | Upheld | All 1.x releases |
| Performance parity | Within 10% | Of best alternative |
| Documentation | Complete | All public APIs |
| Community adoption | ≥ 100 stars | GitHub metrics |
| Active contributors | ≥ 10 | PRs/issues monthly |

---

## 🚀 Beyond 1.0: Long-term Vision

### 1.1 — Cloud & Distributed Compilation
- Cloud-based compilation service
- Global distributed caching
- Continuous integration platform

### 1.2 — Machine Learning Integration
- ML-powered code optimization suggestions
- Auto-tuning based on workload profiles
- Neural code generation assistance

### 1.3 — Edge Computing
- Microcontroller support (Cortex-M, ESP32)
- Embedded systems toolchain
- Real-time OS targets

### 1.4 — Quantum Computing
- Quantum circuit compilation
- QASM backend generation
- Hybrid classical-quantum programs

---

## 📈 Key Performance Indicators

Tracked per release:

### 1. Compiler Performance
- ✓ Compilation speed (LOC/sec)
- ✓ Memory usage during compile
- ✓ Binary size ratio vs manual C

### 2. Runtime Performance
- ✓ Execution speed vs reference implementations
- ✓ Memory efficiency (heap usage)
- ✓ Startup time

### 3. Developer Experience
- ✓ Time to first compile (new user)
- ✓ Error message clarity score
- ✓ Documentation coverage

### 4. Ecosystem Health
- ✓ Number of published packages
- ✓ Active contributors
- ✓ Community engagement (issues, PRs)

---

## 🔄 Change Process

Every version follows this workflow:

```
Proposal RFC → Implementation → Testing Beta → Community Feedback → Release → Maintenance
```

### Phases

1. **Proposal Phase**: RFC (Request for Comments) for major changes
2. **Implementation Phase**: Develop in feature branches
3. **Testing Phase**: Beta releases, community feedback
4. **Release Phase**: Tagged release with changelog
5. **Maintenance Phase**: Bug fixes for minor versions

### Changelog Format

```markdown
## Version 0.2.0

### Added
- Enhanced diagnostics with context-aware suggestions
- Test suite framework with snapshot testing

### Changed
- Error messages now include hint suggestions
- Lexer uses single-pass tokenization

### Fixed
- Parser crash on malformed class definitions
- Memory leak in arena allocator edge case
```

---

## 🤝 Contribution Guide

### How You Can Help

1. **Report Issues**: Bugs, feature requests, documentation gaps
2. **Submit PRs**: Fixes, new features, improvements
3. **Write Tests**: Increase coverage, add edge cases
4. **Improve Docs**: Tutorials, API reference, examples
5. **Spread the Word**: Blog posts, talks, workshops
6. **Pick a Small Task**: Take the next `[]` item from [`road.txt`](road.txt)

### Getting Started

```bash
git clone https://github.com/nazar-ponomarenko-dots/Terra
cd Terra
cmake -S . -B build -G Ninja
cmake --build build
ctest --test-dir build
```

---

## 📝 License & Attribution

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

**Copyright © 2026 Terra Contributors**
