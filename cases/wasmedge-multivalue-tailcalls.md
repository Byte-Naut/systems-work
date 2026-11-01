# WasmEdge: multi-value return ABI for compiled tail calls

**Contribution:** an internal calling-convention change and regression coverage in [PR #5440](https://github.com/WasmEdge/WasmEdge/pull/5440), addressing [issue #5420](https://github.com/WasmEdge/WasmEdge/issues/5420).

**Status:** open as of 2026-11-01. This page describes the submitted implementation and its recorded validation; it does not claim upstream merge.

## Problem

WasmEdge compiled functions with multiple results as direct LLVM structs. Same-type tail calls used `musttail`, but a sufficiently large result struct can be lowered by LLVM to an implicit `sret` parameter. On x86-64, the five-`anyref` reproducer then caused the backend to abort because it could not eliminate the `musttail` call.

Relaxing `musttail` to `tail` makes compilation succeed, but loses constant-stack execution: deep indirect or reference tail recursion eventually traps with `CallStackExhausted`. The fix therefore has to satisfy both backend ABI legality and WebAssembly tail-call behavior.

## Proposed change

Compiled functions with two or more results return `void` and receive a trailing caller-owned `ValVariant*` result buffer.

- Ordinary calls allocate temporary result storage in the caller's entry block and load values after the call. Direct calls check for a pending exception before reading the buffer.
- Same-type `musttail` calls forward the incoming buffer directly, so the tail callee writes into the outer caller's storage without growing the native stack.
- Direct, indirect, reference, import-thunk, and type-wrapper paths use the same convention. Heteromorphic tail calls retain their existing `tail` lowering.
- The AOT binary version changes from 3 to 4 so binaries using the previous return ABI are rejected explicitly.

This extends the earlier SIMD-lowering work from instruction representation to the compiled-function ABI: the common concern is choosing an LLVM representation that preserves Wasm semantics after target lowering.

## Validation recorded in the PR

The regression covers `return_call`, `return_call_indirect`, and `return_call_ref` with four `i64` results and five `f64`, `v128`, or `funcref` results. It includes ordinary-call consumers, compiled/interpreted cross-module calls, null-reference and table traps, and 100,000 recursive hops.

Recorded checks include:

- the original five-`anyref` reproducer compiles instead of aborting in LLVM;
- multi-value tail recursion completes under a 64 KiB stack limit in AOT, JIT, and LazyJIT modes;
- old v3 AOT artifacts fail with the expected version mismatch;
- 144/144 `wasmedgeLLVMCoreTests` cases and 23/23 `wasmedgeLazyJITTests` cases pass.

## Capability record

The change demonstrates work across WebAssembly tail-call semantics, LLVM ABI lowering, native stack behavior, AOT compatibility versioning, cross-module execution, and regression design across compiled execution modes.

Related: [SIMD memory representation and the store-to-load performance cliff](wasmedge-v128-store.md).

[Back to the index](../README.md)
