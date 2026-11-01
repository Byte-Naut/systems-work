# GreptimeDB: query options and distributed process visibility

Two merged contributions address correctness when hidden configuration or remote failure changes what a query appears to mean. The first strengthens configuration-sensitive query regressions; the second prevents a distributed management query from returning a plausible but incomplete result.

| Merged | Pull request | Contribution |
| --- | --- | --- |
| 2026-10-26 | [#9956](https://github.com/GreptimeTeam/greptimedb/pull/9956) | SQLness coverage for session-scoped query options and merge-scan plan variants |
| 2026-10-28 | [#9999](https://github.com/GreptimeTeam/greptimedb/pull/9999) | Propagate remote process-list failures instead of returning a partial cluster view |

## 1. Make query-option regressions observe the triggering configuration

The existing merge-scan semi-join case ran at parallelism four, while the historical missing-row trigger occurred at parallelism two. The result rows could remain correct even if a non-default option stopped reaching the query, so the fixture needed to assert both the configured state and the resulting plan.

PR #9956 adds persistent-session `SET`/`SHOW` coverage for MySQL and PostgreSQL, including invalid-value rejection and restoration of the original values. The merge-scan case exercises:

- parallelism 2 with remote dynamic filtering enabled;
- parallelism 1, which selects the collect-left plan;
- parallelism 2 with remote dynamic filtering disabled;
- restoration to parallelism 4 and the corresponding repartitioned plan.

The change is test-only. A negative control that replaced the first N=2 setting with N=4 left the rows correct but failed the option and plan assertions, showing that the regression detects loss of the triggering configuration rather than merely comparing output rows.

Recorded verification covered eight SQLness case/environment combinations across standalone and distributed modes, plus formatting and the focused binaries.

## 2. Fail explicitly when the cluster process list is incomplete

In a distributed deployment without `internal_grpc`, a frontend can advertise a public gRPC address that does not serve the management RPC. Before PR #9999, `SHOW PROCESSLIST` could ignore a remote `ListProcess` failure and return only local and reachable peer rows. The output looked successful even though it omitted a running query.

`ProcessManager::list_all_processes` now propagates a failed remote call through the existing `InvokeFrontend` error path. Successful calls still aggregate local and remote processes. The fix leaves listener registration, authentication boundaries, internal-channel marking, and remote `KILL` behavior unchanged; it does not expose the management service on the public listener.

The regression uses real metadata discovery and gRPC calls for both the failure and aggregation paths. On the baseline, the missing-service case returned `Ok([local query])`; after the fix it returns the remote `Unimplemented` error. Focused frontend tests passed 14/14 and catalog tests passed 53/53, with an additional two-frontend SQLness check of both `SHOW PROCESSLIST` forms.

## Capability record

Together, the contributions show configuration-aware regression design, plan-level negative controls, session-state restoration, distributed error propagation, real service discovery and RPC testing, and preservation of security/listener boundaries while changing failure semantics.

[Back to the index](../README.md)
