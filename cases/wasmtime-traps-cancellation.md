# Wasmtime: cleanup across concurrency, traps, and cancellation

This series tracks ownership when asynchronous execution is interrupted. It starts with guest-resource destruction inside the concurrent event loop, moves to trap-safe transfer of live fibers, and ends at the public API boundary for canceling a host-initiated read.

**Delivery sequence:** [#14291](https://github.com/bytecodealliance/wasmtime/issues/14291) → [#14241](https://github.com/bytecodealliance/wasmtime/issues/14241) → [#12090](https://github.com/bytecodealliance/wasmtime/issues/12090). The first two produced merged runtime changes and a merged test-maintenance follow-up. The reader-cancellation work stopped at a documented design boundary after upstream discussion. Status checked 2026-11-01.

| Stage | Issue / pull requests | Result |
| --- | --- | --- |
| Concurrent destruction | #14291 → [#14362](https://github.com/bytecodealliance/wasmtime/pull/14362) | Added `ResourceAny::resource_drop_concurrent` and verified guest-context destruction inside `Store::run_concurrent` |
| Trap-safe fiber transfer | #14241 → [#14385](https://github.com/bytecodealliance/wasmtime/pull/14385) → [#14418](https://github.com/bytecodealliance/wasmtime/pull/14418) → [#14620](https://github.com/bytecodealliance/wasmtime/pull/14620) | Explored structural and local ownership models, then merged a focused guard around handoff intervals |
| Test maintenance | [#14681](https://github.com/bytecodealliance/wasmtime/pull/14681) | Removed temporary white-box fault-injection tests after validation while retaining the stable WAST regression |
| Reader cancellation | #12090 | Clarified ownership-return and API-semantics questions; implementation deferred until a concrete use case constrains the design |

## 1. Destroy guest resources while the concurrent store is running

[#14362](https://github.com/bytecodealliance/wasmtime/pull/14362) adds a supported way to destroy a guest-exported `ResourceAny` when the embedder only has an `Accessor`, such as inside `Store::run_concurrent` or an `AccessorTask`.

The implementation queues the destructor on the store's worker fiber, releases the short-lived store borrow, and awaits the result. It reuses `resource_drop_impl`, keeping resource-table handling and destructor behavior aligned with the synchronous and asynchronous APIs. Once queued, canceling the waiting future does not cancel the drop.

`resource_drop_concurrent_in_run_concurrent` creates and destroys a guest resource within one concurrent run. Its destructor checks `thread.index`, so the regression verifies execution in guest thread context rather than merely checking for a successful return.

This established the concurrent environment and cleanup behavior used to examine a harder case: interruption while the scheduler itself is transferring a live execution resource.

## 2. Keep fibers owned across traps, internal errors, and cancellation

Several paths in `concurrent.rs` moved a live `StoreFiber` between store-owned locations and performed a fallible operation in the middle. If a lookup, `bail_bug!`, trap path, or panic interrupted that transfer, the unfinished fiber could be dropped directly. `StoreFiber::drop` then panicked, escalating the original failure into a host panic or process abort.

The solution converged through three PRs:

- [#14385](https://github.com/bytecodealliance/wasmtime/pull/14385) used local RAII guards and reordered lookups so ownership moved only after fallible checks. Fault-injection tests reproduced aborts on unpatched builds.
- [#14418](https://github.com/bytecodealliance/wasmtime/pull/14418) tested a structural alternative in which resting fibers lived in `GuestThreadState`. It made ownership explicit, but added scheduler states and consistency requirements across thread state, wait records, and work queues.
- [#14620](https://github.com/bytecodealliance/wasmtime/pull/14620) retained the existing scheduler representation and kept the useful part of the exploration: a `ResumingFiber` guard around the interval where a fiber is outside store-owned scheduler state.

The merged repair constructs the guard before returning the futures from `resume_fiber` and `handle_work_item`, so cancellation before the first poll still disposes of the fiber through the store. Destination slots are checked before ownership leaves the guard. `mark_ready`, `set_thread_running`, and worker-fiber handling perform fallible work before transfer, and queue promotion scans in place so a predicate panic does not drop pending fibers.

Validation for #14620 recorded:

- 221 Wasmtime unit tests, including 27 focused ownership tests;
- 128 async Component Model WAST trials on Cranelift and Winch;
- 258 Component Model integration tests;
- focused runs with debug assertions disabled;
- `clippy`, non-async, and `no_std` builds.

Five fault-injection paths were checked against unpatched builds that exited with `SIGABRT`, including pre-first-poll cancellation, invalid destinations, a full table while preserving a switch item, and a panic during work-item promotion. `thread-wait-resume.wast` verifies the externally visible rule that a waiting thread is not treated as suspended.

### Post-merge test cleanup — #14681

The focused unit tests deliberately assembled invalid scheduler internals. They were useful for comparing patched and unpatched paths, but were tightly coupled to a fast-changing implementation. After the fix merged, [#14681](https://github.com/bytecodealliance/wasmtime/pull/14681) removed 916 lines of white-box test scaffolding and retained `thread-wait-resume.wast` as the durable regression.

This separates development-time fault injection from long-lived upstream coverage: use invasive tests to establish the failure mechanism, then keep the smallest stable test that protects observable behavior.

## 3. Stop at the semantic boundary for reader cancellation

[#12090](https://github.com/bytecodealliance/wasmtime/issues/12090) asks how a host can begin reading a `FutureReader` or `StreamReader`, cancel the operation, recover the original reader, and potentially pass it back into WebAssembly. A related gap is that `FutureConsumer` cannot currently report `closed`.

The investigation separated three decisions:

1. whether canceling the host-side read leaves the guest writer suspended or signals cancellation;
2. how linear ownership of the reader returns to the host;
3. whether the missing `FutureConsumer` closed state is a separate API change.

Upstream discussion concluded that the semantics and overhead should be chosen around a real use case; the original fuzzer motivation was no longer active. No speculative public API or runtime change was submitted. The issue remains open as a documented capability gap.

## Capability record

The series provides concrete evidence of:

- reasoning about ownership across suspension, transfer, trap propagation, and future cancellation;
- RAII cleanup and protection against cancellation before a future's first poll;
- auditing multi-owner scheduler handoffs and moving failure points before ownership transfer;
- fault-injection validation of states that normally appear only after an invariant has failed;
- testing across execution backends, integration suites, disabled debug assertions, and reduced-feature builds;
- reducing brittle test debt after the failure mechanism and durable regression were established;
- distinguishing an implementation-ready correctness bug from an API gap whose semantics depend on a concrete use case.

The original issues and reviewer discussion supplied the failure reports and design constraints. My contribution is the implementation and test progression for #14291 and #14241, the post-merge test cleanup, and the scoped API analysis recorded on #12090.

Related: [resource ownership, borrowing, and the official embedding example](wasmtime-resource-lifecycle.md).

[Back to the index](../README.md)
