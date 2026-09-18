# WasmEdge: SIMD memory representation and a store-to-load performance cliff

**Contribution:** LLVM lowering changes, endian-path preparation, and validation in [PR #5329](https://github.com/WasmEdge/WasmEdge/pull/5329), addressing [issue #4841](https://github.com/WasmEdge/WasmEdge/issues/4841).

**Status:** merged on 2026-09-14. Status checked 2026-09-18. [Merge commit](https://github.com/WasmEdge/WasmEdge/commit/38fbd79641fdbadef99a8cd3c6d7fc0018f675b6).

## Problem

The reported SIMD loop suffered a large slowdown around a vector store followed by a vector reload. The `<1 x i128>` store representation led the tested x86-64 backend to emit two narrow general-purpose-register stores. A subsequent vector load then encountered an unfavorable store-to-load forwarding pattern.

## Change

I changed the store representation to `<2 x i64>`, enabling a single 16-byte vector store in the tested environment. The merged patch also uses the canonical `Int64x2Ty` representation for full-width `v128.load`.

The endian helper now chooses its intermediate integer from the total vector width. That preparation matters because a 128-bit vector with 64-bit lanes cannot be bitcast through an integer selected only from its lane width. Existing supported shapes were not already broken on the baseline; the refactor makes the new representation valid too.

The patch preserves the existing addressing, alignment, and volatile parameters. The final scope is visible in the [merged diff](https://github.com/WasmEdge/WasmEdge/pull/5329/files).

## Measurements and tradeoff

The PR records these JIT measurements on an Intel Core Ultra 9 275HX with LLVM 18.1.3 on Ubuntu, using the issue's reproducers, 2^30 iterations, and the median of five runs:

| Reproducer | Before | After |
| --- | ---: | ---: |
| Fixed address, lane extraction | 4.89 s | 0.23 s |
| Fixed address, drop control | 0.37 s | 0.35 s |
| Fixed address, scalar-pair control | 0.42 s | 0.50 s |

The lane-extraction case improves by approximately 21.3×. The scalar-pair control regresses by approximately 19%; this is a measured tradeoff. These figures describe the specified experiments in the PR, not a general WasmEdge speedup or a fresh benchmark of every subsequent revision.

## Correctness evidence and limits

The PR records 2004/2004 LLVM core tests passing across AOT, universal AOT, JIT, and lazy JIT, with memory and SIMD coverage. Additional checks include fully out-of-bounds and page-straddling accesses, value preservation, shared memory, and memory64 variants.

Big-endian validation forced the relevant path on x86-64 and checked LLVM IR legality, including the previously invalid new-width bitcast. It did not run the change on physical big-endian hardware.

## Attribution

The issue and the `drop` / `scalar_pair` controls supplied by @gaaraw informed the investigation. My contribution covers the representation change, endian handling, and the validation reported in the PR. The upstream record also documents AI assistance from Claude and ChatGPT.

[Back to the index](../README.md)
