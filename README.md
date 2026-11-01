# Systems work — Byte-Naut

Selected work on system-boundary correctness: runtime ownership, compiler ABI, database semantics, distributed control paths, and low-level execution. Each case links to its upstream discussion or benchmark and distinguishes implementation, validation, and current status.

My current concentration is WebAssembly runtime correctness. Across projects, the recurring question is how state survives a boundary or failure: a guest resource crossing the host API, results crossing a compiled-call ABI, query options crossing sessions and transports, remote failures crossing a cluster collector, or a versioned artifact crossing a CLI workflow.

## Selected cases

| Work | Contribution | Status / result |
| --- | --- | --- |
| Wasmtime Component Model lifecycle | [Ownership and embedding](cases/wasmtime-resource-lifecycle.md): concurrent destruction, synthetic borrows, and documented ownership transfer. [Traps and cancellation](cases/wasmtime-traps-cancellation.md): trap-safe fiber handoffs, pre-first-poll cleanup, test maintenance, and reader-cancellation boundaries | [#14362](https://github.com/bytecodealliance/wasmtime/pull/14362), [#14384](https://github.com/bytecodealliance/wasmtime/pull/14384), [#14434](https://github.com/bytecodealliance/wasmtime/pull/14434), [#14620](https://github.com/bytecodealliance/wasmtime/pull/14620), and [#14681](https://github.com/bytecodealliance/wasmtime/pull/14681) merged; [#12090](https://github.com/bytecodealliance/wasmtime/issues/12090) deferred pending a concrete use case |
| WasmEdge LLVM lowering | [SIMD memory representation](cases/wasmedge-v128-store.md) and [multi-value compiled-call ABI](cases/wasmedge-multivalue-tailcalls.md) | [#5329](https://github.com/WasmEdge/WasmEdge/pull/5329) merged; [#5440](https://github.com/WasmEdge/WasmEdge/pull/5440) open |
| [GreptimeDB query and control-path correctness](cases/greptimedb-query-control.md) | Session-aware query-option regressions and explicit failure instead of partial distributed process lists | [#9956](https://github.com/GreptimeTeam/greptimedb/pull/9956) and [#9999](https://github.com/GreptimeTeam/greptimedb/pull/9999) merged 2026-10-26–28 |
| [workerd socket garbage collection](cases/workerd-socket-close-gc.md) | Traceable continuation captures and retention regression tests | [#7258](https://github.com/cloudflare/workerd/pull/7258), open |
| [DuckDB BETWEEN semantics](cases/duckdb-between-null.md) | NULL handling and statistics propagation fixes | [#25394](https://github.com/duckdb/duckdb/pull/25394), merged 2026-09-10 |
| [TestSprite definition transfer](cases/testsprite-definition-transfer.md) | Deterministic export/import, optimistic concurrency, staged idempotency, and atomic artifact output | [#417](https://github.com/TestSprite/testsprite-cli/pull/417), open |
| [CUDA Q/K RMSNorm on B200](cases/sol-execbench-rmsnorm.md) | A kernel implementation evaluated over 16 shapes | Recorded SOL Score 0.577614; Fast₁ 16/16 |

Statuses were checked on 2026-11-01. Test and timing summaries refer to the experiments recorded in the linked contributions; open pull requests are identified as open, and the CUDA case records a submitted result rather than a live ranking.

## Supporting projects

[Wasm static-data / SQLite benchmark](https://github.com/Byte-Naut/wasm-static-data-sqlite-bench) investigates the loading and instantiation cost of embedding large static data, including a negative result relative to external SQLite in the tested CLI setting.

[Mini-LevelDB](https://github.com/Byte-Naut/mini-leveldb) is an educational C++20 storage engine covering a complete LSM pipeline.

[HAMi-core initialization topology study](https://github.com/Byte-Naut/hami-core-init-topology-study) is an independent reproduction and evidence report about concurrent host-PID discovery. Its scope is process-level initialization, with limits stated in the report.

[Working together](WORKING_TOGETHER.md) · [GitHub profile](https://github.com/Byte-Naut) · [Contact](mailto:Byte-Naut@proton.me)
