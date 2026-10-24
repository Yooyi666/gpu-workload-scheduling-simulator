# Verified project description for resumes

**GPU Workload Scheduling Simulator** | Go, Python, Docker  
*Independent Systems Engineering Project | 2026*

- Developed a deterministic multi-GPU scheduling simulator for **1,000 synthetic training, inference, and evaluation workloads**, modeling per-GPU memory/compute limits, arrivals, non-preemptive execution, and resource release.
- Implemented and benchmarked **FIFO, strict-priority, and adaptive policies** with aging, guarded backfilling, and workload-aware placement; improved **simulated compute-resource utilization from 79.42% to 86.17%** and reduced **p95 waiting time by 31.07%** versus FIFO in a fixed-seed workload experiment.
- Built automated correctness and concurrent resource-allocation tests, validated tested code paths with Go's **race detector**, and reproduced benchmark results using **Docker**, **Python** visualizations, and **10-seed workload stress tests**.

## Accuracy notes

- Always describe these numbers as **simulation** results. They do not represent actual NVIDIA GPU utilization or real model-training speedups.
- Don't claim the event loop executes tasks on real GPUs or goroutines; concurrent goroutine tests validate the resource manager.
- Avoid replacing the measured 79.42% → 86.17% figures with the original unverified 62% → 78% claim.
- The 31.07% reduction comes from **one fixed seed**; use the multi-seed study separately to discuss robustness and its limitations.
