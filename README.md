# Systems work — Byte-Naut

Selected work on language semantics, runtime resource lifetimes, and low-level execution. Each case links to its upstream discussion or benchmark and distinguishes the implementation, its validation, and its current status.

My current concentration is WebAssembly runtime correctness and resource management. The latest [Wasmtime delivery series](cases/wasmtime-resource-lifecycle.md) develops one subsystem across three merged PRs: concurrent destruction establishes an execution path and regression; synthetic-borrow conversion refines the shared cleanup model; the official embedding example then incorporates the ownership rules and reuses the earlier tests. The database, JavaScript-runtime, and CUDA work provides additional evidence across system boundaries.

## Selected cases

| Work | Contribution | Status / result |
| --- | --- | --- |
| [Wasmtime resource lifecycle](cases/wasmtime-resource-lifecycle.md) | Concurrent guest destruction → synthetic-borrow conversion → documented ownership transfer in the official embedding example | [#14362](https://github.com/bytecodealliance/wasmtime/pull/14362), [#14384](https://github.com/bytecodealliance/wasmtime/pull/14384), [#14434](https://github.com/bytecodealliance/wasmtime/pull/14434); all merged 2026-09-22–29 |
| [WasmEdge SIMD lowering](cases/wasmedge-v128-store.md) | LLVM memory representation and a width-safe endian conversion | [PR #5329](https://github.com/WasmEdge/WasmEdge/pull/5329), merged 2026-09-14 |
| [workerd socket garbage collection](cases/workerd-socket-close-gc.md) | Traceable continuation captures and retention regression tests | [PR #7258](https://github.com/cloudflare/workerd/pull/7258), open |
| [DuckDB BETWEEN semantics](cases/duckdb-between-null.md) | NULL handling and statistics propagation fixes | [PR #25394](https://github.com/duckdb/duckdb/pull/25394), merged 2026-09-10 |
| [CUDA Q/K RMSNorm on B200](cases/sol-execbench-rmsnorm.md) | A kernel implementation evaluated over 16 shapes | Recorded SOL Score 0.577614; Fast₁ 16/16 |

Wasmtime merges were verified on 2026-10-04; the other PR statuses were recorded on 2026-09-18. Test and timing summaries refer to the experiments recorded in the linked contributions; they are not continuous assertions about newer upstream revisions. The CUDA case identifies the provenance of its recorded result separately.

## Supporting projects

[Wasm static-data / SQLite benchmark](https://github.com/Byte-Naut/wasm-static-data-sqlite-bench) investigates the loading and instantiation cost of embedding large static data, including a negative result relative to external SQLite in the tested CLI setting.

[Mini-LevelDB](https://github.com/Byte-Naut/mini-leveldb) is an educational C++20 storage engine covering a complete LSM pipeline.

[HAMi-core initialization topology study](https://github.com/Byte-Naut/hami-core-init-topology-study) is an independent reproduction and evidence report about concurrent host-PID discovery. Its scope is process-level initialization, with limits stated in the report.

[Working together](WORKING_TOGETHER.md) · [GitHub profile](https://github.com/Byte-Naut) · [Contact](mailto:Byte-Naut@proton.me)
