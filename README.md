![logo](logo.png)
# Terra - Tesseract Compiler
a fork based on https://github.com/TheTesseractProject/tract

> *Terra* is one of the implementations of the Tesseract language.
> At this time, this implementation may not comply with Tesseract standards.

> [!WARNING]
> This project is currently under active development. Features and documentation may change frequently. At the moment the language is not ready for full-fledged work, it is a prototype.

```tesseract
TesseractClass {
    a: int;
    b: str;
    hi {
         put("Hello there!");
    }
}

Class = TesseractClass.new;
Class.hi;
```

```txt
output:
Hello there!
```


## 🚀 Features

- **Fast Compilation**: Optimized for speed and efficiency
- **Cross-platform C99 output**: Linux, macOS, and Windows
- **Modern C**: Written in C23 with best practices
- **Error Handling**: Comprehensive error reporting

## 🗺️ Mini roadmap

> Terra is a prototype; the language and generated output may change. Current
> stage: **0.5 — standard library and safe runtime**.

| Version | Milestone | Status |
|---------|-----------|--------|
| 0.1 | Lexer, parser, sema, first C99 codegen | ✅ |
| 0.2 | Diagnostics and test foundation | ✅ |
| 0.3 | Production C99 backend, modular engine | ✅ |
| 0.4 | Native C interop (`import`, `extern fn`, `link`, `--compile`), `fn main`, `for` loops, systems types: arrays, slices, pointers, C-layout `struct`/`union`, `asm` | ✅ |
| 0.5 | Standard library: string methods, validated `math`, `--safe` arithmetic/indexing, binary/hex literals | 🏗️ current |
| 0.6 | Broader standard library and package manager | 🔵 planned |
| 0.7 | Concurrency and parallelism | 🔵 planned |
| 0.8 | Advanced type system | 🔵 planned |
| 0.9 | Tooling: LSP, formatter, debugger | 🔵 planned |
| 1.0 | Stability and production readiness | 🔵 planned |
| 2.0 | Native/LLVM backend, cross-platform, distributed compilation | 🔮 future |
| 3.0 | Ecosystem: ML, edge and quantum experiments | 🔮 future |

What works now: `put`/`print`/`len`/`str`/`int`/`float`/`assert`/`input`,
Python-style string methods, native interop via `import "header.h"` +
`extern fn` (see [c_import.t](examples/c_import.t) and
[c_file.t](examples/c_file.t)), and freestanding objects via `--freestanding`.
File-based Tesseract module imports remain unsupported; executable builds are
POSIX-only, while C source generation works on all supported platforms.

Small scoped tasks: [road.txt](road.txt) · full plan: [ROADMAP.md](ROADMAP.md).

## 📋 Requirements

### Build Dependencies
- **CMake** 3.16 or higher
- **C Compiler**: Clang 18 or higher (recommended) or GCC 15 or higher
- **Build System**: Ninja (recommended) or Make

### Runtime Dependencies
- **POSIX system** for `--compile` (C source generation works everywhere)
- **Math library** (linked automatically)

## 🛠️ Building

### Quick Start (Recommended)
```bash
# Clone the repository
git clone https://github.com/nazar-ponomarenko-dots/Terra
cd Terra
```

```bash
# Build with Clang and Ninja (recommended)
export CC=clang && export CXX=clang++
cmake -S . -B build -G Ninja
cmake --build build
```

```bash
# Build with GCC
export CC=gcc && export CXX=g++
cmake -S . -B build -G Ninja
cmake --build build
```

```bash
# Run the application
./bin/terra --help
```

### Clean Build
```bash
rm -rf bin/ build/
```

## 🎯 Usage

### Basic Commands

```bash
# Show help
./bin/terra --help

# Show version
./bin/terra --version

# Translate a file to C (writes ./output by default)
./bin/terra input.tt

# Translate with a custom output path
./bin/terra input.tt -o output.c

# Only parse and type-check
./bin/terra --check input.tt

# Write the generated C to stdout
./bin/terra input.tt --stdout

```

> Terra emits standard C99 source that builds with GCC or Clang, e.g.
> `cc output.c -o program`. Inline assembly (`asm(...)`) additionally requires
> GNU assembler syntax.

### Command Line Options

| Option | Long Option | Description |
|--------|-------------|-------------|
| `-h` | `--help` | Show help information |
| `-v`, `-V` | `--version` | Show version and build information |
| `-b` | `--build` | Compile the specified file (path may also be positional) |
| `-o` | `--output` | Output path; `-` means stdout (`--output=path` also works) |
| `-c` | `--check` | Parse and type-check only, do not generate C |
| | `--stdout` | Write generated C to standard output |
| | `--compile` | Compile generated C into an executable |
| | `--cc <path>` | Select compiler for `--compile` |
| | `--target <triple>` | Cross-compile with the selected C compiler (needs a sysroot) |
| | `--freestanding` | Emit a freestanding relocatable object (`.o`) |
| | `--safe` | Runtime-check arithmetic and indexing (panic instead of UB) |
| `-q` | `--quiet` | Suppress progress messages |
| | `--` | Treat all following arguments as paths |

### Supported File Extensions

- `.tt` - Tesseract source files
- `.tes` - Tesseract source files (alternative)
- `.t` - Tesseract source files (alternative)

## 🧪 Development

### Code Style

This project follows these coding standards:
- **C23 Standard** compliance
- **4-space indentation** (no tabs)
- **Snake_case** for functions and variables
- **UPPER_CASE** for constants and macros
- **Comprehensive error handling**
- **Memory safety** practices

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Guidelines

- Follow the existing code style
- Add error handling for all new functions
- Include appropriate documentation
- Test on multiple platforms if possible

## 📄 License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

## 🙏 Acknowledgments

- Made with ❤️ by Contributors
- Inspired by the mathematical beauty of 4D tesseracts
- Built with modern C development practices

## 📞 Support

If you encounter any issues or have questions:

1. Check the [Issues](https://github.com/nazar-ponomarenko-dots/Terra/issues) page
2. Create a new issue with detailed information
3. Include your system information and error messages

---

**Terra** - Where mathematics meets performance! 🚀
