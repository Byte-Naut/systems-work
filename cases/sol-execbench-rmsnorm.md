# CUDA: Q/K per-head RMSNorm on NVIDIA B200

**Work:** a CUDA C++ implementation for [SOL-ExecBench kernel #38](https://research.nvidia.com/benchmarks/sol-execbench/kernel/38), optimizing Q/K per-head RMSNorm across 16 shapes.

**Result:** ranked **#12**, with a SOL Score of **0.577614** and **1.26× average speedup** on NVIDIA B200. [Submission #47128](https://research.nvidia.com/benchmarks/sol-execbench/submission/47128).

## Problem

The task was to optimize Q/K per-head RMSNorm across 16 workload shapes while meeting the benchmark's numerical correctness requirements. The implementation needed to perform well across different batch sizes and sequence lengths.

## Implementation

I used warp reductions, 128-bit vectorized memory access, and kernel fusion, with attention to occupancy and the memory hierarchy. Development combined numerical validation and tests on other GPUs with B200 evaluation feedback.

## Results

The submission passed evaluation on NVIDIA B200 with Evaluation Stack v1.1 and beat the baseline on all 16 workloads.

| Metric | Result |
| --- | --- |
| Leaderboard rank | #12 |
| SOL Score | 0.577614 |
| Fast₁ | 16/16 |
| Latency | 0.025256 ms |
| Average speedup | 1.26× |
| Submission date | 2026-09-15 |

Per-workload speedups ranged from **1.04× to 1.92×**, with latencies from **0.0049 to 0.1188 ms**. The [submission page](https://research.nvidia.com/benchmarks/sol-execbench/submission/47128) provides the individual workload results; NVIDIA's [methodology](https://research.nvidia.com/benchmarks/sol-execbench/blog/introducing-sol-execbench) explains the SOL scoring.

[Back to the index](../README.md)
