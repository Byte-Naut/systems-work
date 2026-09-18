# CUDA: Q/K per-head RMSNorm on NVIDIA B200

**Work:** a CUDA C++ implementation for [SOL-ExecBench kernel #38](https://research.nvidia.com/benchmarks/sol-execbench/kernel/38), optimizing Q/K per-head RMSNorm across 16 shapes.

**Status:** [submission #47128](https://research.nvidia.com/benchmarks/sol-execbench/submission/47128) accepted (AC), submitted under **Byte-Naut** in Public mode on 2026-09-15 at 09:15:29 UTC+08:00, using **Evaluation Stack v1.1**.

## Recorded result

| Field | Recorded value |
| --- | --- |
| Target GPU | NVIDIA B200 |
| SOL Score | 0.577614 |
| Fast₁ | 16/16 |
| Leaderboard position | #12 in the saved leaderboard excerpt |
| Leaderboard latency | 0.025256 ms |
| Leaderboard average speedup | 1.26× |

The submission-page snapshot saved on 2026-09-18 at 11:47:16 UTC+08:00 records the accepted result and all 16 workload rows. Displayed per-workload latencies range from 0.0049 to 0.1188 ms, with speedups of 1.04×–1.92× against the corresponding baselines.

Rank, aggregate latency, average speedup, and Fast₁ come from a separately saved leaderboard excerpt for the same alias and score. Its capture time is not recorded; #12 describes that excerpt, with no claim about the current ranking.

## Engineering scope

The work involved warp reductions, 128-bit vectorized memory access, kernel fusion, occupancy and memory-hierarchy considerations, and numerical validation. B200 evaluation feedback was used alongside tests on other GPUs during development.

Measurements from a different GPU serve as development feedback and are not interchangeable with the recorded B200 result. The workload timings describe this kernel benchmark; they do not establish a general model-level speedup.

## Reading the metric

NVIDIA defines SOL Score as a normalized function of submission latency, stored baseline latency, and an estimated hardware limit. A single kernel's score averages its workload scores. The value 0.577614 should therefore not be read as 57.7614% hardware utilization or converted directly into an overall speedup. See the [official methodology](https://research.nvidia.com/benchmarks/sol-execbench/blog/introducing-sol-execbench).

## Evidence scope

The linked submission and saved page document the public alias, evaluation version, acceptance status, and per-workload results. This case summarizes the evaluated outcome; implementation source is not included.

[Back to the index](../README.md)
