# CUDA: Q/K per-head RMSNorm on NVIDIA B200

**Work:** a CUDA C++ implementation for [SOL-ExecBench kernel #38](https://research.nvidia.com/benchmarks/sol-execbench/kernel/38), optimizing Q/K per-head RMSNorm across 16 shapes.

## Recorded result

| Field | Recorded value |
| --- | --- |
| Target GPU | NVIDIA B200 |
| SOL Score | 0.577614 |
| Fast₁ | 16/16 |
| Workload coverage | 16 shapes, with correctness and performance requirements |

These are my recorded submission results. A submission-specific export and dated leaderboard snapshot are not attached to this case yet. The linked kernel page is the public benchmark entry; the table above is not a claim about the current leaderboard position.

## Engineering scope

The work involved warp reductions, 128-bit vectorized memory access, kernel fusion, occupancy and memory-hierarchy considerations, and numerical validation. B200 evaluation feedback was used alongside tests on other GPUs during development.

Measurements from a different GPU serve as development feedback and are not interchangeable with the recorded B200 result. This case does not infer per-shape latency or a general model-level speedup from the aggregate score.

## Reading the metric

NVIDIA defines SOL Score as a normalized function of submission latency, stored baseline latency, and an estimated hardware limit. A single kernel's score averages its workload scores. The value 0.577614 should therefore not be read as 57.7614% hardware utilization or converted directly into an overall speedup. See the [official methodology](https://research.nvidia.com/benchmarks/sol-execbench/blog/introducing-sol-execbench).

## Evidence boundary

The CUDA source revision, submission identifier, evaluation-stack version, submission date, and the 16-shape output are needed to tie the result to a fully reproducible snapshot. They are not reconstructed or invented here. The implementation result is retained without attaching an undated rank to the public profile.

[Back to the index](../README.md)
