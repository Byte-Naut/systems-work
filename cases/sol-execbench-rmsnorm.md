# CUDA: Q/K per-head RMSNorm on NVIDIA B200

**Work:** a CUDA C++ implementation for [SOL-ExecBench kernel #38](https://research.nvidia.com/benchmarks/sol-execbench/kernel/38), optimizing Q/K per-head RMSNorm across 16 shapes.

## Recorded result

| Field | Recorded value |
| --- | --- |
| Target GPU | NVIDIA B200 |
| SOL Score | 0.577614 |
| Fast₁ | 16/16 |
| Workload coverage | 16 shapes, with correctness and performance requirements |
| Leaderboard rank | #12 |
| Latency | 0.025256 ms |
| Average speedup | 1.26× |
| Submission date | 2026-09-15 |

[Submission #47128](https://research.nvidia.com/benchmarks/sol-execbench/submission/47128), submitted as Byte-Naut, was accepted with Evaluation Stack v1.1. The submission page includes all 16 workload results, with per-workload speedups of 1.04×–1.92×.

## Engineering scope

The work involved warp reductions, 128-bit vectorized memory access, kernel fusion, occupancy and memory-hierarchy considerations, and numerical validation. B200 evaluation feedback was used alongside tests on other GPUs during development.

## Reading the metric

NVIDIA defines SOL Score as a normalized function of submission latency, stored baseline latency, and an estimated hardware limit. A single kernel's score averages its workload scores. See the [official methodology](https://research.nvidia.com/benchmarks/sol-execbench/blog/introducing-sol-execbench).

[Back to the index](../README.md)
