# CLAUDE.md - ArkCompiler ETS Runtime

## Project Overview

ArkCompiler ETS Runtime is the default ETS/TS/JS runtime for OpenHarmony. It provides:
- ECMAScript (ES2021, strict mode) standard library support
- Multi-tier execution: interpreter, JIT compiler, and AOT compiler
- High-performance garbage collection (concurrent marking/sweeping, parallel evacuation)
- Native C++ APIs (NAPI) for cross-language interoperability
- Efficient non-standard container libraries
- Debugger and profiling tools

**License**: Apache 2.0
**Component**: `ets_runtime` under the `arkcompiler` subsystem

## Repository Structure

```
/arkcompiler/ets_runtime/
├── ecmascript/           # Core runtime implementation
│   ├── base/             # Base helper classes and utilities
│   ├── builtins/         # ECMAScript standard library implementations
│   ├── compiler/         # Compiler infrastructure (Circuit IR, AOT, codegen, stubs)
│   ├── containers/       # Non-ECMAScript containers (ArrayList, HashMap, TreeMap, etc.)
│   ├── debugger/         # Debugger implementation
│   ├── deoptimizer/      # Compiler deoptimization support
│   ├── dfx/              # Diagnostics: CPU profiler, heap profiler, vmstat
│   ├── ic/               # Inline caching for property access optimization
│   ├── interpreter/      # Bytecode interpreter (C++ and assembly variants)
│   ├── intl/             # Internationalization (ICU-based)
│   ├── jit/              # Just-In-Time compilation engine
│   ├── jobs/             # Asynchronous job queue
│   ├── js_api/           # Non-standard JS object models
│   ├── js_vm/            # Command-line JSVM tool (ark_js_vm)
│   ├── jspandafile/      # ABC bytecode file management
│   ├── mem/              # Memory management and garbage collection
│   ├── module/           # ECMAScript module system (ESM + CommonJS)
│   ├── napi/             # C++ Native API (jsnapi.h, jsnapi_expo.h)
│   ├── patch/            # Hot patch and cold patch support
│   ├── pgo_profiler/     # Profile-Guided Optimization
│   ├── platform/         # Cross-platform abstractions (unix, ohos)
│   ├── regexp/           # Regular expression engine
│   ├── require/          # CommonJS require() implementation
│   ├── serializer/       # Object serialization
│   ├── shared_mm/        # Shared memory management
│   ├── shared_objects/   # Shared memory object implementations
│   ├── snapshot/         # VM snapshot serialization
│   ├── stubs/            # Runtime stub functions
│   ├── taskpool/         # Task pool
│   ├── ts_types/         # TypeScript type management
│   └── tests/            # Runtime unit tests
├── common_components/    # Shared infrastructure
│   ├── heap/             # GC framework (allocators, collectors, barriers, spaces)
│   ├── base/             # Base utilities
│   ├── common_runtime/   # Runtime base classes
│   ├── log/              # Logging framework
│   ├── mutator/          # Mutator lock
│   ├── thread/           # Threading utilities
│   ├── platform/         # Platform abstractions
│   ├── taskpool/         # Task pool infrastructure
│   └── tests/            # Component tests
├── compiler_service/     # AOT compiler service (SA-based)
├── test/                 # Integration and regression tests
│   ├── aottest/          # AOT compiler tests (~422 cases)
│   ├── moduletest/       # Module system tests (~332 cases)
│   ├── fuzztest/         # Fuzz tests (~391 cases)
│   ├── jittest/          # JIT compilation tests (~143 cases)
│   ├── sharedtest/       # Shared memory tests
│   ├── quickfix/         # Quick fix tests
│   ├── deopttest/        # Deoptimization tests
│   ├── regresstest/      # Regression tests
│   ├── jsperftest/       # JS performance benchmarks
│   ├── aotjsperftest/    # AOT performance benchmarks
│   ├── workload/         # Real-world workload tests
│   └── runtest.py        # Test runner script
├── tools/                # Utilities (AP file viewer, circuit viewer)
├── script/               # Build and execution scripts
├── docs/                 # Documentation
└── etc/                  # Configuration files
```

## Build System

The project uses **GN (Generate Ninja)** as its build system, integrated with the OpenHarmony build framework.

### Key Build Files

- `BUILD.gn` — Main build configuration (defines all targets)
- `js_runtime_config.gni` — Runtime feature flags and configuration
- `bundle.json` — OpenHarmony component definition and dependencies

### Primary Build Targets

| Target | Description |
|--------|-------------|
| `ark_js_packages` | Full runtime package |
| `ark_js_host_linux_tools_packages` | Linux host development tools |
| `libark_jsruntime` | Core runtime shared library |
| `libark_jsruntime_static` | Core runtime static library |
| `ark_aot_compiler` | Ahead-of-Time compiler |
| `ark_js_vm` | JavaScript VM executable |
| `libcompiler_service` | AOT compiler service library |

### Build Commands

```bash
# Full build (standard product)
./build.sh --product-name rk3568 --build-target ark_js_host_linux_tools_packages

# Alternative product
./build.sh --product-name hispark_taurus_standard --build-target ark_js_host_linux_tools_packages

# Debug build
./build.sh --product-name rk3568 --gn-args is_debug=true --build-target ark_js_host_linux_tools_packages

# ARM64 target
./build.sh --product-name rk3568 --gn-args use_musl=true --target-cpu arm64 --build-target ark_js_packages

# Specific component
./build.sh --build-target arkcompiler/ets_runtime:libark_jsruntime
```

### Build Output

```
out/<product>/clang_x64/arkcompiler/ets_runtime/   # Host x64 tools
out/<product>/arkcompiler/ets_runtime/              # Device binaries

Key binaries:
  ark_js_vm           — JavaScript VM interpreter
  ark_aot_compiler    — AOT compiler
  libark_jsruntime.so — Runtime shared library
  profdump            — PGO profiler dump tool
```

### Build Configuration Flags (js_runtime_config.gni)

| Flag | Default | Description |
|------|---------|-------------|
| `enable_ark_intl` | `true` | Use ICU for Intl APIs |
| `enable_fuzz_option` | `false` | Enable fuzz testing |
| `run_with_asan` | `false` | Address Sanitizer |
| `ets_runtime_feature_enable_pgo` | `false` | Profile-Guided Optimization |
| `ets_runtime_feature_enable_codemerge` | `false` | Code merging optimization |
| `ets_runtime_support_jit_code_sign` | `false` | JIT code signing |
| `enable_next_optimization` | `true` | Next-gen optimizations |
| `ets_runtime_enable_cmc_gc` | `false` | CMC garbage collector |

## Programming Languages

| Language | Usage |
|----------|-------|
| **C++** (primary) | Core runtime, compiler, VM, GC, builtins |
| **C++ Headers** | Public APIs, class definitions |
| **JavaScript** | Builtin implementations, test cases |
| **TypeScript** | Frontend tooling, test cases |
| **Python** | Test runners, build scripts |
| **GN** | Build configuration |
| **YAML** | ISA definitions (`ecma_isa.yaml`), system events |

## Testing

### Test Runner

```bash
# Run a specific test
python3 test/runtest.py <file|path>

# Run all tests in a directory
python3 test/runtest.py -a <path>

# Run with specific execution mode
python3 test/runtest.py <file> -s aot       # AOT compilation
python3 test/runtest.py <file> -s asmint    # Assembly interpreter (JIT)
python3 test/runtest.py <file> -s int       # C++ interpreter

# Debug mode
python3 test/runtest.py <file> -d

# Specify product
python3 test/runtest.py <file> -p rk3568

# With PGO
python3 test/runtest.py <file> --pgo --pgo-th 10

# With builtins
python3 test/runtest.py <file> --bt

# Set timeout (default: 300s)
python3 test/runtest.py <file> --timeout 600
```

### Test Categories

| Category | Directory | Description |
|----------|-----------|-------------|
| Unit tests | `ecmascript/*/tests/` | Per-component unit tests |
| AOT tests | `test/aottest/` | AOT compiler correctness |
| JIT tests | `test/jittest/` | JIT compilation correctness |
| Module tests | `test/moduletest/` | ES module system |
| Fuzz tests | `test/fuzztest/` | Fuzzing and stress tests |
| Shared tests | `test/sharedtest/` | Shared memory objects |
| Quick fix | `test/quickfix/` | Hot/cold patch |
| Deopt tests | `test/deopttest/` | Deoptimization |
| Regression | `test/regresstest/` | Regression suite |
| Performance | `test/jsperftest/`, `test/aotjsperftest/`, `test/workload/` | Benchmarks |

### ECMAScript Test262

```bash
python3 test/run_ts_test262.py
```

Supports ES5.1, ES2015, and ES2021 test suites in strict and default modes.

## Architecture

### Execution Pipeline

```
Source (TS/JS)
    │
    ▼
ArkCompiler Frontend (es2abc / ts2abc)
    │
    ▼
ABC Bytecode File (.abc)
    │
    ├──► Interpreter ─── C++ interpreter or assembly interpreter
    │
    ├──► JIT Compiler ── Hot method compilation at runtime
    │
    └──► AOT Compiler ── Ahead-of-time compilation for startup
```

### Compiler IR Pipeline

The compiler uses a multi-level Circuit IR:
1. **Bytecode** → **HCR** (High-level Circuit Representation) — bytecode-level ops
2. **HCR** → **MCR** (Mid-level Circuit Representation) — type-specialized ops
3. **MCR** → **LCR** (Low-level Circuit Representation) — machine-near ops
4. **LCR** → **Machine Code** (via LLVM or LiteCG backends)

Key optimization passes: constant folding, dead code elimination, escape analysis, loop peeling, type inference, range analysis, value numbering, inline caching specialization, array bounds check elimination.

### Memory Management

- **Tagged values**: All JS values use 64-bit tagged representation (`JSTaggedValue`)
- **Hidden classes**: Shape-based object optimization (`JSHClass`)
- **Heap spaces**: Young generation, old generation, large object, non-movable, machine code
- **GC strategies**: Concurrent marking, concurrent sweeping, parallel evacuation, incremental marking
- **Write barriers**: Required for cross-generation/cross-region references

### Key Runtime Classes

| Class | File | Purpose |
|-------|------|---------|
| `EcmaVM` | `ecmascript/ecma_vm.h` | Main VM instance |
| `JSThread` | `ecmascript/js_thread.h` | Execution thread |
| `JSTaggedValue` | `ecmascript/js_tagged_value.h` | Universal value representation |
| `JSObject` | `ecmascript/js_object.h` | Base object type |
| `JSHClass` | `ecmascript/js_hclass.h` | Hidden class / shape |
| `JSFunction` | `ecmascript/js_function.h` | Function objects |
| `Heap` | `ecmascript/mem/heap.h` | Heap and GC management |
| `Circuit` | `ecmascript/compiler/circuit.h` | Compiler IR graph |
| `StubBuilder` | `ecmascript/compiler/stub_builder.h` | Runtime stub code generation |

## Code Conventions

### File Organization

- Header files (`.h`) and implementation files (`.cpp`) are co-located in the same directory
- Inline implementations use `-inl.h` suffix (e.g., `heap-inl.h`, `stub_builder-inl.h`)
- Test files live in `tests/` subdirectories within each component
- Each BUILD.gn target explicitly lists its source files

### Naming Conventions

- **Files**: `snake_case.h`, `snake_case.cpp`
- **Classes**: `PascalCase` (e.g., `JSObject`, `EcmaVM`, `CircuitBuilder`)
- **Methods**: `PascalCase` (e.g., `GetProperty`, `SetPrototype`)
- **Member variables**: `camelCase_` with trailing underscore
- **Constants**: `UPPER_SNAKE_CASE` or prefixed enums
- **Macros**: `UPPER_SNAKE_CASE` (extensively used in `ecma_macros.h`)
- **Namespaces**: `panda::ecmascript` (primary), `panda::ecmascript::kungfu` (compiler)

### License Headers

All source files must include the Apache 2.0 license header:

```cpp
/*
 * Copyright (c) 2021-2025 Huawei Device Co., Ltd.
 * Licensed under the Apache License, Version 2.0 (the "License");
 * ...
 */
```

### Code Patterns

- **Handle pattern**: GC-safe references use `JSHandle<T>` to protect objects from GC movement
- **Tagged values**: All JS values are boxed in `JSTaggedValue` (64-bit NaN-boxed representation)
- **Macro-driven dispatch**: The interpreter uses macro-generated dispatch tables (`ecma_macros.h`)
- **Stub builders**: Runtime fast paths are implemented as compiler stubs using `StubBuilder`
- **Gate-based IR**: The compiler IR uses `Gate` nodes connected in a `Circuit` graph

### Important Constraints

- Only ABC bytecode files from the ArkCompiler frontend (es2abc/ts2abc) are supported
- Only ES2021 standard in strict mode is supported
- Dynamic function creation from strings is not supported (e.g., `new Function("...")`)

## Dependencies

### OpenHarmony Components

`runtime_core`, `hilog`, `hisysevent`, `hitrace`, `faultloggerd`, `icu`, `libuv`, `zlib`, `c_utils`, `code_signature`, `qos_manager`, `ffrt`, `cJSON`, `ipc`, `samgr`, `safwk`

### Related Repositories

- [arkcompiler_runtime_core](https://gitee.com/openharmony/arkcompiler_runtime_core) — Core runtime infrastructure
- [arkcompiler_ets_frontend](https://gitee.com/openharmony/arkcompiler_ets_frontend) — Frontend compiler (ts2abc, es2abc)

## Development Workflow

### PR Checklist (from project template)

When modifying this codebase, verify:

1. **Interpreter changes**: Ensure C++ interpreter and assembly interpreter remain consistent
2. **IR/compiler changes**: Verify IR logic matches equivalent C++ runtime implementations
3. **Array access**: Validate all array index bounds checking
4. **Slow paths**: Ensure all optimized paths have correct slowpath fallbacks
5. **Exception handling**: Verify exception propagation through all new code paths

### Performance Testing Requirements

For performance-sensitive changes, run:
- `jit-workload` — JIT compilation workload tests
- `interpreter-js_perf` — Interpreter JS performance
- `aot-js_perf` — AOT JS performance
- `interpreter-workload` — Interpreter workload tests
- `aot-workload` — AOT workload tests

### Code Ownership

- Overall: `@klooer`
- `common_components/`: `@weng-changcheng`, `@dmitriitr`
- `common_components/heap/`: `@weng-changcheng`, `@igelhaus`, `@dmitriitr`
- `compiler_service/`: `@dingding5`, `@zhangyukun8`

See `CODEOWNERS` and `REVIEWERS` files for full details.

## Quick Reference

### Running Bytecode

```bash
# Set library path and run
LD_LIBRARY_PATH=out/<product>/clang_x64/arkcompiler/ets_runtime:out/<product>/clang_x64/thirdparty/icu:prebuilts/clang/ohos/linux-x86_64/llvm/lib \
  ./out/<product>/clang_x64/arkcompiler/ets_runtime/ark_js_vm hello.abc
```

### Key Entry Points for Code Navigation

- **VM initialization**: `ecmascript/ecma_vm.cpp` — `EcmaVM::Initialize()`
- **Interpreter main loop**: `ecmascript/interpreter/interpreter-inl.cpp`
- **JIT compilation**: `ecmascript/jit/jit.cpp` — `Jit::Compile()`
- **AOT compilation**: `ecmascript/compiler/aot_compiler.cpp`
- **GC entry**: `ecmascript/mem/heap.cpp` — `Heap::CollectGarbage()`
- **Object allocation**: `ecmascript/mem/heap-inl.h` — `Heap::AllocateYoungOrHugeObject()`
- **Builtin functions**: `ecmascript/builtins/` (one file per ES builtin)
- **NAPI public API**: `ecmascript/napi/include/jsnapi.h`, `jsnapi_expo.h`
- **Runtime stubs**: `ecmascript/compiler/stub_builder.cpp`
- **Pass manager**: `ecmascript/compiler/pass_manager.cpp`
