# Wasmtime: resource ownership across the host/guest boundary

Three merged contributions connect concurrent resource destruction, synthetic borrow conversion, and the official embedding example. Each step builds on the resource behavior or tests established by the preceding work, with ownership transfer and documentation as the final focus.

**Delivery sequence:** [#14291](https://github.com/bytecodealliance/wasmtime/issues/14291) → [#7793](https://github.com/bytecodealliance/wasmtime/issues/7793) → [#9946](https://github.com/bytecodealliance/wasmtime/issues/9946). All three PRs were merged into Wasmtime's `main`; verified 2026-10-04. Dates below are UTC.

| Merged | Issue → PR | Contribution carried forward |
| --- | --- | --- |
| 2026-09-22 | #14291 → [#14362](https://github.com/bytecodealliance/wasmtime/pull/14362) | Concurrent destruction through `Accessor`, with a regression that checks the guest execution context |
| 2026-09-25 | #7793 → [#14384](https://github.com/bytecodealliance/wasmtime/pull/14384) | Refined the same cleanup model to distinguish synthetic borrows from resources with host table state |
| 2026-09-29 | #9946 → [#14434](https://github.com/bytecodealliance/wasmtime/pull/14434) | Extended the official ownership example using that distinction and the earlier concurrent-drop regression |

## 1. Establish destruction in the concurrent execution environment

An embedder holding only an `Accessor` could not drop a guest-exported `ResourceAny` inside `Store::run_concurrent`. The existing async API needed a mutable store context across the destructor call, but that borrow could not escape `Accessor::with`.

I added `ResourceAny::resource_drop_concurrent`. It queues destruction on the store's worker fiber, releases the store borrow, then awaits the result. It shares `resource_drop_impl` with the synchronous and asynchronous APIs. Once destruction is queued, dropping the waiting future does not cancel the queued work.

The regression `resource_drop_concurrent_in_run_concurrent` obtains and destroys a guest resource within the same concurrent run. Its guest destructor traps when `thread.index` is zero, checking that destruction runs in a guest thread context. The change also documents which drop API to use for each way of driving the store.

This established a supported execution path, a focused runtime regression, and a common cleanup implementation for the next step. [Merged implementation and tests](https://github.com/bytecodealliance/wasmtime/commit/7c55e310786cdbde905d825e8c46683a9230f6da).

## 2. Refine which borrows need that cleanup

Working in the same resource implementation, I addressed conversion of `Resource<T>::new_borrow(rep)` and `ResourceDynamic::new_borrow` into `ResourceAny`. Conversion eagerly created a host table borrow, requiring an active component call scope. Outside a call, it hit `no current scope`: a debug-build panic or a release-build `WasmtimeBug` error.

The fix gives `ResourceAny` an internal distinction between a table entry and a synthetic borrow carrying its raw representation. Synthetic borrows are lowered when a guest call actually happens. The existing drop entry points continue through the shared cleanup implementation; for a synthetic borrow with no table state, cleanup is a no-op. Borrows lifted from a guest retain their table state and drop obligation.

This refines the first step's destruction model: the execution context determines how to run a destructor, while the resource's representation determines whether there is host state to release. The re-enabled `can_use_own_for_borrow` test covers conversion and a typed round-trip, type mismatch, rejection of borrow-to-own lowering, and the trap for a guest that leaves a borrow alive at call end. `resource_dynamic` adds conversion and borrowing coverage for dynamically typed host resources. [Merged fix, tests, and release note](https://github.com/bytecodealliance/wasmtime/commit/4ac4c8cbf7a8a541b4a0798923a08e68d283da62).

## 3. Carry the behavior model into the official embedding example

With the concurrent-drop API available and the borrow distinction documented, I extended Wasmtime's exported-resource example to show the complete host-side ownership flow. The WIT interface now includes `get-default-logger`, `inspect-logger`, and `take-logger`.

The example receives an owned guest resource, lends it through `borrow<logger>`, calls a resource method, and explicitly destroys it. A second resource is constructed and passed back through an owned parameter. The comments explain that the WIT signature determines ownership even though both parameter kinds appear as `ResourceAny` in Rust; copying a handle does not make it reusable after ownership has transferred.

The accompanying API documentation separates host-defined and guest-defined resources, explains that typed conversions only accept host-defined resources, and covers owned resources nested in records, variants, and lists. It preserves the synthetic-borrow distinction from #14384 and the three destruction APIs from #14362.

Validation directly reused `resource_drop_concurrent_in_run_concurrent` from the first contribution, alongside the existing `pass_guest_back_as_borrow` and `manually_destroy` runtime tests. Rustdoc checks compile the expanded embedding example; runtime regressions cover the underlying behavior. The final PR changes documentation and example bindings.

[Official embedding example at the merged revision](https://github.com/bytecodealliance/wasmtime/blob/c7ff273e233be6cbd9a4b94fc4fc79beac9023b4/crates/wasmtime/src/runtime/component/bindgen_examples/mod.rs#L444-L517) · [Example WIT and bindings](https://github.com/bytecodealliance/wasmtime/blob/c7ff273e233be6cbd9a4b94fc4fc79beac9023b4/crates/wasmtime/src/runtime/component/bindgen_examples/_6_exported_resources.rs) · [ResourceAny API documentation](https://docs.wasmtime.dev/api/wasmtime/component/struct.ResourceAny.html)

## Validation and contribution record

The linked PRs record focused resource tests, the component-model suite, formatting and lint checks for the implementation changes, and rustdoc builds for the documentation. Focused commands recorded during the series include:

```sh
cargo test --test all resource_drop
cargo test --test all component_model::resources
RUSTDOCFLAGS="--cfg docsrs" cargo test -p wasmtime --doc _6_exported_resources
```

The series leaves a reusable progression in one subsystem: a supported concurrent destructor path, a corrected borrow representation and cleanup model, then an official ownership walkthrough checked against the API and resource tests.

The original reports came from @SilverMira, @rvolosatovs, and @tliron; upstream discussion supplied API and deferred-lowering guidance. My contribution is the implementation, regression coverage, and documentation carried through these three merged PRs. The PRs record AI assistance and my responsibility for reviewing and testing the changes.

[Back to the index](../README.md)
