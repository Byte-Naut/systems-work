# CUDA: Q/K per-head RMSNorm on NVIDIA B200

**Work:** a CUDA C++ implementation for [SOL-ExecBench kernel #38](https://research.nvidia.com/benchmarks/sol-execbench/kernel/38), optimizing Q/K per-head RMSNorm across 16 shapes.

## Recorded result

| Field | Recorded value |
| --- | --- |
| Target GPU | NVIDIA B200 |
| SOL Score | 0.577614 |
| Fast₁ | 16/16 |
| Workload coverage | 16 shapes, with correctness and performance requirements |
| Leaderboard rank | #12 on kernel #38 / B200, checked 2026-09-19 |
| Latency (leaderboard aggregate) | 0.025256 ms |
| Avg Speedup (leaderboard aggregate) | 1.26× |
| Submission date | 2026-09-15 |

[Submission #47128](https://research.nvidia.com/benchmarks/sol-execbench/submission/47128), submitted as Byte-Naut, was accepted with Evaluation Stack v1.1. The [public B200 leaderboard](https://research.nvidia.com/benchmarks/sol-execbench/leaderboard/kernel/38/B200) lists the aggregate result. The [saved submission results (HTML)](../evidence/sol-execbench-47128.html), captured on 2026-09-18, preserve the Accepted status, timestamps, and all 16 workload results. Per-workload speedups range from 1.04× to 1.92×.

## Benchmark results

Values below are transcribed from the saved submission page, retaining its displayed precision.

| Workload | Latency (ms) | Baseline (ms) | Speedup | SOL Score |
| --- | ---: | ---: | ---: | ---: |
| `batch_size=2, seq_len=128` | 0.0069 | 0.0104 | 1.50x | 0.629520 |
| `batch_size=4, seq_len=1657` | 0.0973 | 0.1072 | 1.10x | 0.541761 |
| `batch_size=4, seq_len=1024` | 0.0618 | 0.0667 | 1.08x | 0.532605 |
| `batch_size=8, seq_len=773` | 0.0927 | 0.1027 | 1.11x | 0.543451 |
| `batch_size=8, seq_len=128` | 0.0191 | 0.0225 | 1.18x | 0.562601 |
| `batch_size=2, seq_len=293` | 0.0122 | 0.0175 | 1.44x | 0.625094 |
| `batch_size=16, seq_len=256` | 0.0619 | 0.0665 | 1.07x | 0.530403 |
| `batch_size=1, seq_len=1024` | 0.0190 | 0.0224 | 1.18x | 0.561869 |
| `batch_size=4, seq_len=256` | 0.0189 | 0.0224 | 1.19x | 0.563960 |
| `batch_size=1, seq_len=128` | 0.0049 | 0.0093 | 1.92x | 0.689702 |
| `batch_size=32, seq_len=128` | 0.0631 | 0.0666 | 1.06x | 0.522949 |
| `batch_size=4, seq_len=512` | 0.0330 | 0.0376 | 1.14x | 0.552979 |
| `batch_size=1, seq_len=131` | 0.0049 | 0.0094 | 1.92x | 0.688679 |
| `batch_size=8, seq_len=256` | 0.0330 | 0.0374 | 1.13x | 0.550881 |
| `batch_size=1, seq_len=8192` | 0.1188 | 0.1234 | 1.04x | 0.516844 |
| `batch_size=1, seq_len=256` | 0.0070 | 0.0105 | 1.50x | 0.628524 |

## Engineering scope

The work involved warp reductions, 128-bit vectorized memory access, kernel fusion, occupancy and memory-hierarchy considerations, and numerical validation. B200 evaluation feedback was used alongside tests on other GPUs during development.

## Reading the metrics

NVIDIA defines SOL Score as a normalized function of submission latency, stored baseline latency, and an estimated hardware limit. For one kernel, it is the arithmetic mean of the workload scores. Fast₁ 16/16 means all 16 workloads pass correctness checks and run faster than the scoring baseline. The summary latency and Avg Speedup above are the aggregate values displayed on the B200 leaderboard; individual workload measurements appear in the table. See the [official methodology](https://research.nvidia.com/benchmarks/sol-execbench/blog/introducing-sol-execbench).

[Back to the index](../README.md)
