# TestSprite CLI: deterministic backend definition transfer

**Contribution:** backend test-definition export and import in [PR #417](https://github.com/TestSprite/testsprite-cli/pull/417), addressing [issue #125](https://github.com/TestSprite/testsprite-cli/issues/125).

**Status:** open as of 2026-11-01. This page describes the submitted behavior and validation; it does not claim merge or bonus award.

## Problem

Individual Python/pytest test definitions could not be exported as stable, reviewable artifacts and later restored through one workflow. A safe implementation also had to distinguish creating a copy from updating an existing test, preserve code-version provenance, handle expiring source URLs, and avoid leaving files or remote state in an ambiguous partial update.

## Proposed workflow

`testsprite test export <test-id>` combines test metadata and code into deterministic JSON. It preserves the supported definition fields and source text while removing runtime/status fields. Recursive key sorting makes the artifact stable without reordering arrays or changing source whitespace.

When the code API returns a presigned HTTPS URL, export fetches the source without forwarding facade authentication headers. Source and artifact size limits keep the result bounded and self-contained, and file output uses the existing atomic sibling-temp-file path.

`testsprite test import <file>` has two explicit modes:

- **Create:** creates a new backend test. Saved IDs, versions, and ETags remain provenance and never trigger an implicit overwrite.
- **Update:** requires `--test-id`, verifies target and project identity, and uses the saved `codeVersion` or an explicit expected version. Code is updated with `If-Match` before metadata, so a version conflict aborts before metadata changes. A later metadata failure reports the partial state and the idempotency key needed for a safe retry.

Caller-provided idempotency is split into stable `:create`, `:code`, and `:metadata` stage keys. Frontend test definitions are outside the submitted v1 scope and fail explicitly.

## Validation recorded in the PR

The submitted branch records:

- 4,612 unit tests passing, with two existing Windows-only skips;
- line coverage 91.96%, statement coverage 89.76%, function coverage 83.10%, and branch coverage 84.86%;
- 103 local end-to-end tests passing with one existing skip;
- 80 GitHub script tests passing with two existing skips;
- lint, formatting, and TypeScript checks passing;
- validation on WSL Ubuntu / Node 24 in an isolated loopback network namespace.

## Capability record

The work applies systems-style state management to a CLI/API boundary: deterministic serialization, bounded and credential-safe artifact retrieval, explicit identity and version checks, optimistic concurrency, idempotent staged retries, atomic file publication, and transparent reporting of unavoidable partial updates.

[Back to the index](../README.md)
