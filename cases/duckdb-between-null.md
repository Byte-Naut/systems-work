# DuckDB: BETWEEN, NULL bounds, and representation-dependent results

**Contribution:** root-cause analysis, implementation, and regression tests in [PR #25394](https://github.com/duckdb/duckdb/pull/25394), addressing [issue #25170](https://github.com/duckdb/duckdb/issues/25170).

**Status:** merged on 2026-09-10. Status checked 2026-09-18. [Merge commit](https://github.com/duckdb/duckdb/commit/7b76e0f91a15e776148c44f94b9d7b4eeb26f3a0).

## Problem

The reported symptom was a row-count inconsistency involving duplicated RIGHT JOIN queries and UNION ALL. Investigation reduced it to SQL three-valued logic interacting with vector representation and optimizer rewriting.

`BETWEEN` combines two comparisons with AND. A FALSE comparison must still make the result FALSE when the other comparison is UNKNOWN. The expression executor's default NULL handling bypassed that logic when an argument arrived as a CONSTANT NULL vector. Changing the physical representation or the query plan could therefore change a logically identical result.

The following reduced witness should return FALSE:

```sql
SELECT ((EXISTS (SELECT 1)) BETWEEN NULL::BOOLEAN AND false);
```

## Change

I gave the internal `__between` function special NULL handling so its own three-valued logic determines the result. I also corrected `StatisticsPropagator::PropagateBetween`: its FALSE_OR_NULL folding conditions and NULL guards must account for the other comparison and use the relevant bound.

These changes address both runtime expression evaluation and the optimizer's interpretation of the same semantics.

## Validation recorded in the PR

The added `test/sql/join/test_between_null_bound_join.test` covers the reduced RIGHT JOIN reproducer, CONSTANT versus FLAT inputs, the scalar NULL-bound witness, statistics propagation, and the original UNION ALL/shared-subplan query.

The PR records failure on the baseline and success after the fix, plus passing focused join, optimizer, filter, subquery, CTE, set-operation, projection, and NULL-related suites. The full unittest run retained an unrelated pre-existing timing-based C API failure; it is not described here as an entirely passing run.

## Contribution boundary

The initial issue and reproducer were supplied by another contributor. My contribution is the diagnosis, repair, and regression coverage. This case does not claim authorship of the initial discovery, a general database performance improvement, or release inclusion beyond the verified merge.

The case illustrates why correctness needs to survive physical representation changes and optimizer transformations, even when the visible failure initially resembles a query-planning problem.

[Back to the index](../README.md)
