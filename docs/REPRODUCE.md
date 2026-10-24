# Reproducing all benchmarks

## Environment

- Go 1.27.3 (tested locally); `go.mod` may declare a lower language-version minimum.
- Python **3.12** for the pinned packages in `requirements.txt`.
- Docker Desktop with Linux container backend, or Docker Engine on Linux.

## Command-line interface

```powershell
# Stage 5, default policy comparison
 go run . -stage 5 -tasks 1000 -seed 20261024 -out stage5_results

# Stage 6, paired-seed robustness / arrival-rate scenarios
 go run . -stage 6 -tasks 1000 -seed 20261024 -seeds 10 -out stage6_results
```

The Go CLI accepts `-tasks`, `-seed`, `-out`, `-stage`, and (for Stage 6) `-seeds`. Do not change seeds, workload count, task generator, policies or GPU config when comparing against the fixed reference figures.

## Python analytics (Windows PowerShell)

```powershell
py -3.12 -m venv .venv312
.\.venv312\Scripts\python.exe -m pip install --only-binary=:all: -r requirements.txt
.\.venv312\Scripts\python.exe analytics\plot_results.py --stage5 stage5_results --stage6 stage6_results --out stage8_analytics
```

Expected: `VALIDATED: 1000 tasks x 3 strategies; 10 seeds`, six PNG charts, `REPORT.md`, `metrics.json`.

## Docker (Windows PowerShell)

```powershell
docker build -t gpu-scheduling-simulator:stage8 .
New-Item -ItemType Directory -Force stage8_results | Out-Null
$outputDir = (Resolve-Path .\stage8_results).Path
docker run --rm --mount "type=bind,source=$outputDir,target=/results" gpu-scheduling-simulator:stage8
```

Once complete, review:

- `stage8_results/stage5_results/summary.csv`
- `stage8_results/stage6_results/aggregate.csv`
- `stage8_results/analytics/REPORT.md` and `metrics.json`
- Six `*.png` in `stage8_results/analytics/`

## Troubleshooting

| Symptom | Check |
|---|---|
| `go.mod not found` | Open the project *root* (same folder as `go.mod`) |
| NumPy tries to compile / metadata generation failure | Use Python **3.12** virtualenv with `--only-binary=:all:`; avoid incompatible Python 3.14 + pinned NumPy |
| `docker daemon` unreachable | Start Docker Desktop and enable Linux containers |
| Mount invalid on Windows | Use the resolved absolute path and `--mount` shown above |
| Missing `metrics.json` | Confirm both Stage 5 and Stage 6 produced all CSV files; re-run Python analytics |
| `-race requires cgo` | Install MSYS2 UCRT64 GCC; enable CGO in the current PowerShell session |

Do not compare synthetic simulation milliseconds against wall-clock run times of the Go program or image build.
