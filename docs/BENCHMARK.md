# Benchmark protocol and results

## Controlled experiment

- **One workload set:** 1,000 task records generated using local seeded Go PRNG (`20261024`).
- **Same input per policy:** FIFO, Priority and Adaptive see the same arrivals, durations, priorities and resource demands.
- **GPU fleet:** 4 identical simulated GPUs; **24,576 MiB memory** and **100 abstract compute units** each.
- **Task types:** generated mixture of training (~32%), inference (~43%) and evaluation (~25%) according to code's categorical probabilities. Their resource and duration distributions differ.
- **Execution model:** deterministic, discrete-event, non-preemptive simulation. Time units are **simulated ms**; no real GPU kernels are executed.
- **Integrity:** schedule validator checks completion once, timing, GPU placement, resource bounds and resource cleanup.

## Definitions

For policy `p` and task `i`:

- **Waiting time:** `start_ms(i) - arrival_ms(i)`.
- **p95 waiting time:** the **nearest-rank** percentile; sort waits ascending and select index `ceil(0.95 × N) - 1` (zero-based).
- **Makespan:** maximum task completion timestamp; workload generator starts with arrival 0.
- **Compute utilization:** `100 × [sum over GPUs integral(reserved_compute_units(g,t) dt)] / [sum over GPUs capacity_compute(g) × makespan]`.
- **Memory utilization:** analogous using reserved simulated memory.

**Compute utilization means reservation occupancy, not NVIDIA GPU hardware telemetry or training throughput.**

## Stage 5: fixed seed 20261024

| Metric | FIFO | Priority | Adaptive |
|---|---:|---:|---:|
| Tasks completed | 1,000 | 1,000 | 1,000 |
| Compute utilization (%) | 79.4225 | 79.9673 | 86.1697 |
| Memory utilization (%) | 49.5792 | 49.9192 | 53.7911 |
| Mean waiting time (ms) | 1,967.01 | 858.31 | 762.93 |
| p50 waiting time (ms) | 1,682 | 27 | 315 |
| p95 waiting time (ms) | 3,953 | 5,772 | 2,725 |
| p99 waiting time (ms) | 4,016 | 8,719 | 3,000 |
| Makespan (ms) | 13,065 | 12,976 | 12,042 |

Adaptive vs FIFO:

- **Compute utilization:** `86.1697 - 79.4225 = +6.7472 percentage points` (displayed as +6.75 pp).
- **p95 waiting reduction:** `(3953 - 2725)/3953 × 100 = 31.065%` (displayed as 31.07%).
- **Makespan reduction:** `(13065 - 12042)/13065 × 100 = 7.8301%`.

Note that strict priority improves some short-task waits but its **overall p95 is worse** on this workload, so the result is not uniformly favorable to Priority scheduling.

## Stage 6: robustness study

Run **10 consecutive seeds**, with **1,000 tasks per seed**, **3 arrival-rate scenarios**, and **3 policies** (`10 × 3 × 3 = 90` policy runs). In each seed/scenario, all policies use the same inputs. Scenario scaling modifies arrival-time spacing, not GPU capacity or task resource/duration requirements.

| Scenario | FIFO mean utilization (%) | Adaptive mean utilization (%) | Mean paired p95 reduction (%) |
|---|---:|---:|---:|
| High arrival rate | 79.88 | 86.00 | 16.93 |
| Baseline | 79.63 | 85.34 | 24.11 |
| Low arrival rate | 63.87 | 63.88 | 17.76 |

The paired p95 reduction is first computed per seed as `(p95_fifo - p95_adaptive) / p95_fifo × 100`, then **the ten percentages are averaged**. This quantity can differ materially from the reduction inferred from the *mean p95 values*.

**Boundary condition:** under low arrival rates, average GPU utilization is essentially unchanged, indicating little opportunity for packing-oriented gains even if tail waiting time may improve.

## Artifacts and verification

- [Curated Stage 5 summary](data/stage5_summary.csv) — CSV matching the fixed-seed reported results.
- [Curated Stage 6 summary](data/stage6_aggregate.csv) and [paired per-seed comparison](data/stage6_paired_improvements.csv).
- `stage5_results/{workloads.csv,summary.csv,schedule.csv,manifest.json,report.md}` — full generated Stage 5 experiment.
- `stage6_results/{per_seed.csv,paired_improvements.csv,aggregate.csv,report.md}` — full generated Stage 6 experiment.
- `stage8_analytics/` or Docker-mounted `stage8_results/analytics/` — six charts, REPORT and metrics JSON.

The full per-task schedules are **generated**, rather than checked into the portfolio by default. Re-run with the documented seed to reproduce them; confirm the generator/policy code has not changed.

## Limitations

These results depend on synthetic distributions, 4 modeled GPUs, deterministic task duration, non-preemptive policies, a particular random-seed selection and arrival-rate settings. They are not statistical statements about real GPU platforms. The robustness study supplies across-seed variability but does not by itself establish generalizability to production workloads.
