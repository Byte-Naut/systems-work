# Systems work — Byte-Naut

Selected work on language semantics, runtime resource lifetimes, and low-level execution. Each case links to its upstream discussion or benchmark and distinguishes the implementation, its validation, and its current status.

My current concentration is WebAssembly runtime correctness and resource management. The database, JavaScript-runtime, and CUDA work provides additional evidence across system boundaries.

## Selected cases

| Work | Contribution | Status / result |
| --- | --- | --- |
| [WasmEdge SIMD lowering](cases/wasmedge-v128-store.md) | LLVM memory representation and a width-safe endian conversion | [PR #5329](https://github.com/WasmEdge/WasmEdge/pull/5329), merged 2026-09-14 |
| [workerd socket garbage collection](cases/workerd-socket-close-gc.md) | Traceable continuation captures and retention regression tests | [PR #7258](https://github.com/cloudflare/workerd/pull/7258), open |
| [DuckDB BETWEEN semantics](cases/duckdb-between-null.md) | NULL handling and statistics propagation fixes | [PR #25394](https://github.com/duckdb/duckdb/pull/25394), merged 2026-09-10 |
| [CUDA Q/K RMSNorm on B200](cases/sol-execbench-rmsnorm.md) | A kernel implementation evaluated over 16 shapes | Recorded SOL Score 0.577614; Fast₁ 16/16 |

PR states were checked on 2026-09-18. Test and timing summaries refer to the experiments recorded in the linked contributions; they are not continuous assertions about newer upstream revisions. The CUDA case identifies the provenance of its recorded result separately.

## Supporting projects

[Wasm static-data / SQLite benchmark](https://github.com/Byte-Naut/wasm-static-data-sqlite-bench) investigates the loading and instantiation cost of embedding large static data, including a negative result relative to external SQLite in the tested CLI setting.

[Mini-LevelDB](https://github.com/Byte-Naut/mini-leveldb) is an educational C++20 storage engine covering a complete LSM pipeline.

[HAMi-core initialization topology study](https://github.com/Byte-Naut/hami-core-init-topology-study) is an independent reproduction and evidence report about concurrent host-PID discovery. Its scope is process-level initialization, with limits stated in the report.

[Working together](WORKING_TOGETHER.md) · [GitHub profile](https://github.com/Byte-Naut) · [Contact](mailto:Byte-Naut@proton.me)
