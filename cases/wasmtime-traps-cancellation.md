# Wasmtime: resource lifecycle across concurrency, traps, and cancellation

This series follows one cleanup problem through three layers of the Component Model runtime: concurrent destruction, trap-safe fiber ownership, and host-side read cancellation. The technical thread is ownership under interruption: first making destruction possible while the concurrent event loop is running, then preventing traps and internal errors from dropping a live fiber mid-transfer, and finally examining how a canceled read could return ownership of a `FutureReader` or `StreamReader`.

**Delivery sequence:** [#14291](https://github.com/bytecodealliance/wasmtime/issues/14291) → [#14241](https://github.com/bytecodealliance/wasmtime/issues/14241) → [#12090](https://github.com/bytecodealliance/wasmtime/issues/12090). The first two reached merged runtime changes; the last remains an API design gap and was intentionally left without an implementation after upstream discussion. Status checked 2026-10-12.

| Stage | Issue / PR | Result carried forward |
| --- | --- | --- |
| Concurrent destruction | #14291 → [#14362](https://github.com/bytecodealliance/wasmtime/pull/14362) | Added `ResourceAny::resource_drop_concurrent` and a regression that verifies guest-context destruction inside `Store::run_concurrent` |
| Trap-safe fiber handoff | #14241 → [#14385](https://github.com/bytecodealliance/wasmtime/pull/14385) → [#14418](https://github.com/bytecodealliance/wasmtime/pull/14418) → [#14620](https://github.com/bytecodealliance/wasmtime/pull/14620) | Explored two ownership representations, then merged a smaller repair that guards the intervals where a live fiber is outside store-owned scheduler state |
| Read cancellation | #12090 | Identified cancellation semantics, ownership-return, and `FutureConsumer` closed-state questions; implementation deferred until a concrete use case constrains the API and overhead |

## 1. Establish cleanup while the concurrent event loop is active

[#14362](https://github.com/bytecodealliance/wasmtime/pull/14362) added a supported way to destroy a guest-exported `ResourceAny` when the host only has an `Accessor`, such as inside `Store::run_concurrent` or an `AccessorTask`.

The implementation queues the destructor on the store's worker fiber, ends the short-lived store borrow, and then awaits the result. It reuses the existing `resource_drop_impl`, so table bookkeeping and destructor behavior stay shared with the synchronous and asynchronous APIs.

The focused regression, `resource_drop_concurrent_in_run_concurrent`, obtains a guest resource and destroys it in the same concurrent run. Its destructor checks `thread.index`, verifying that the destruction executes in guest thread context. Once the drop has been queued, canceling the waiting future does not cancel the queued destruction.

That work established the concurrent execution environment and cleanup behavior used as the starting model for the next problem: what happens when interruption occurs while the runtime itself is transferring a live execution resource.

## 2. Extend the cleanup model from resources to live fibers

[#14241](https://github.com/bytecodealliance/wasmtime/issues/14241) tracks a different failure mode in the same async component runtime. Several scheduler paths moved a live `StoreFiber` from one store-owned location to another and performed a fallible operation in between. If a table lookup, trap path, `bail_bug!`, or panic occurred during that interval, dropping the unfinished fiber triggered `StoreFiber::drop` and could escalate the original failure into a host panic or process abort.

The implementation converged through three PRs.

### Local guards and preflight checks — #14385

[#14385](https://github.com/bytecodealliance/wasmtime/pull/14385) protected individual handoffs with a private RAII `DisposeFiber` guard and reordered several paths so fallible lookups happened before the fiber was taken. Fault-injection tests constructed invalid scheduler states directly and verified that the original error returned while the fiber remained store-owned or was disposed through the store. On an unpatched build, the covered cases aborted the test process.

Review raised a structural question: a local guard still relies on every future handoff remembering to use the abstraction. That led to a second design experiment.

### Store-owned resting fibers — #14418

[#14418](https://github.com/bytecodealliance/wasmtime/pull/14418) moved resting guest fibers into `GuestThreadState`, leaving wait sets and work items to carry thread identifiers. This made fiber ownership structural, but it also required the scheduler to represent distinctions such as waiting, scheduled, suspended, and ready explicitly through `FiberKind`.

The implementation exposed the cost of that representation change: the new ownership invariant also expanded the scheduler state space and introduced consistency requirements between thread state, wait records, and work queues. Review retained one part as useful — the `ResumingFiber` protection around the actual resume interval — and the next revision separated ownership repair from scheduler-state redesign.

### Guard only the transfer interval — #14620

[#14620](https://github.com/bytecodealliance/wasmtime/pull/14620), merged on 2026-10-12, keeps the existing `GuestThreadState`, `WaitMode`, and `WorkItem` shapes. The fix concentrates on the interval where a fiber has been removed from one store-owned location but has not yet reached the next.

`ResumingFiber` owns the live fiber across that interval. `resume_fiber` and `handle_work_item` return futures only after the guard has been created, so dropping a future before its first poll still disposes the fiber through the store. Destination checks happen before ownership leaves the guard. `mark_ready`, `set_thread_running`, and worker-fiber handling were reordered so fallible operations complete before transfer. Queue promotion scans in place, so a predicate panic does not drop queued fibers.

The final validation records:

- 221 Wasmtime unit tests, including 27 focused fiber-ownership tests;
- 128 async Component Model WAST trials on Cranelift and Winch;
- 258 Component Model integration tests;
- focused tests with debug assertions disabled;
- `clippy`, non-async, and `no_std` builds.

Five fault-injection paths were checked against unpatched builds that exited with `SIGABRT`, including pre-first-poll cancellation of `resume_fiber` and `handle_work_item`, invalid destination state, a full table during switch preservation, and a panic during work-item promotion. The `thread-wait-resume.wast` test carried forward from #14418 checks that a waiting thread is not misclassified as suspended.

This stage turns the earlier cleanup model into a scheduler invariant: failures and cancellation may interrupt a handoff, but ownership of the live fiber remains explicit until it reaches another store-owned location or is disposed.

## 3. Apply the cancellation/ownership model to reader APIs

[#12090](https://github.com/bytecodealliance/wasmtime/issues/12090) asks how a host can start reading a `FutureReader` or `StreamReader`, cancel that read, regain the original reader, and potentially pass it back into WebAssembly. A related gap is that `FutureConsumer` currently has no way to report `closed`.

The investigation separated three design questions before implementation:

1. whether canceling the host-side read should leave the guest writer suspended for a future reader or signal cancellation to the writer;
2. how linear ownership of the reader should be returned to the host — for example through a dedicated result/wrapper or another guarded representation;
3. whether the missing `FutureConsumer` closed state should be a separate API change or part of the cancellation work.

Upstream discussion concluded that the design space and runtime overhead could not be chosen well without a current concrete use case; the original fuzzer motivation no longer applied. No speculative runtime change or public API was submitted. The issue remains open as a documented capability gap.

The useful result of this stage is a boundary on implementation: the same cancellation-safe ownership problem seen in fiber transfer also exists at the public reader API, but here the unresolved semantics and cost model come before code.

## Capability record

Across the series, the merged work and upstream discussion provide concrete evidence of:

- reasoning about ownership during asynchronous suspension, transfer, trap propagation, and future cancellation;
- RAII-based cleanup for live runtime resources and pre-first-poll cancellation;
- auditing multi-owner scheduler handoffs and moving fallible checks before ownership transfer;
- fault-injection testing of states that normally appear only after an invariant has already failed;
- validating runtime changes across interpreter/compiler backends, integration tests, disabled debug assertions, and reduced-feature builds;
- separating an implementation-ready runtime correctness bug from an API-design gap whose semantics and overhead still depend on a concrete use case.

The original issues and reviewer discussion supplied the failure reports and design constraints. My contribution is the implementation and test progression for #14291 and #14241, plus the scoped API analysis recorded on #12090.

Related: [resource ownership, borrowing, and the official embedding example](wasmtime-resource-lifecycle.md).

[Back to the index](../README.md)
