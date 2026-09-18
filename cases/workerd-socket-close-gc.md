# workerd: collecting pending Socket::close() retention cycles

**Contribution:** diagnosis, a proposed capture-tracing fix, and regression tests in [PR #7258](https://github.com/cloudflare/workerd/pull/7258), addressing [issue #7202](https://github.com/cloudflare/workerd/issues/7202).

**Status:** open as of 2026-09-18. [PR head revision](https://github.com/Byte-Naut/workerd/commit/ddba9f40ad0b929bded766adf967f60a96ff2a46). This case does not claim upstream approval or merge.

## Problem

`Socket::close()` used continuation captures that the tracing garbage collector could not see. The resulting strong references could root a cycle involving a socket, its writable stream, a pending flush promise, and the continuation capturing the socket. AsyncLocalStorage state belonging to completed requests remained reachable through that cycle.

The investigation distinguished this from the issue's initial explanation about unsettled promises at teardown. The no-close control could be collected; a pending close was the condition needed to reproduce the retention.

## Proposed change

I wrapped five captures in `JSG_VISITABLE_LAMBDA` so the visitor can trace the references. Reachable continuations still retain the socket, preserving the lifetime protection introduced by the earlier use-after-free fix. An otherwise unreachable cycle can be collected.

The change leaves IoContext cancellation semantics alone. It does not introduce a general policy of settling every pending promise during request teardown.

## Regression evidence recorded in the PR

The new tests cover legacy and TypeScript stream implementations. They distinguish a direct socket, a socket transferred over an RPC boundary, and a passive no-close control.

| Condition | Baseline | With proposed fix |
| --- | --- | --- |
| Direct socket, pending close | 12/12 request stores retained | Collected |
| RPC-transferred socket, pending close | 12/12 request stores retained | Collected |
| Pending reactions without close | Collected | Collected |

A fixture guard checks that close was still pending when the request returned. The PR also records focused socket and GC tests, including the pre-existing use-after-free regression, and a successful build. These are the contribution's recorded test results, not a claim that an internal upstream build has been approved.

The diagnostic checks found JS-heap retention while the underlying connection was dropped at teardown. The result is not presented as a file-descriptor leak fix.

## Follow-through

On 2026-09-14 I documented a rebase and resolution of the overlap with PR #7313, preserving its close reason along with the traced captures. Focused socket-close and GC tests, including the newly introduced pipe-close case, were reported passing in the [maintenance update](https://github.com/cloudflare/workerd/pull/7258#issuecomment-5658690769).

That update records maintenance after the initial submission. The next external step is upstream review and the requested internal build.

[Back to the index](../README.md)
