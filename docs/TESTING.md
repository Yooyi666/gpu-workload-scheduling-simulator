# Test strategy and concurrency scope

## Run tests

```powershell
go fmt ./...
go test ./... -count=1
go test -race ./... -count=1
```

On Linux (including the GitHub Actions Ubuntu runner), `go test -race` normally uses the available CGO/GCC toolchain. On Windows, the race detector requires a compatible C compiler and CGO enabled. In a VS Code PowerShell with MSYS2 UCRT64 GCC installed:

```powershell
$env:PATH = "C:\msys64\ucrt64\bin;$env:PATH"
$env:CGO_ENABLED = '1'
$env:CC = 'gcc'
gcc --version
gcc --print-file-name libsynchronization.a
go test -race ./... -count=1
```

The last GCC command should resolve to a path, not the bare string `libsynchronization.a`.

## Automated coverage goals

| Area | Assertion examples |
|---|---|
| Task and GPU model | Valid priorities/types/resources; reject invalid values |
| Resource manager | Over-allocation rejected; release reclaims capacity; double releases rejected |
| Concurrency | Multiple goroutines cannot reserve beyond capacity; no leaked accounting |
| FIFO | Arrival order, strict head-of-line blocking, deterministic event order |
| Priority | Priority ordering, stable ties, invalid input rejection |
| Adaptive | Aging, reserved-head backfilling, type-aware best-fit placement |
| Schedule validation | Exactly-once completion, no resource-limit violations, valid timings |
| Reproducibility | Same seed leads to identical schedule and metrics |
| Stress experiment | Paired inputs across all three strategies; summary generation |
| Docker integration | Full pipeline exports analysis plots and metrics in a mounted folder |

## What a passing race detector means

Race-detector checks found no data races **on the paths exercised in the test run**. This does not guarantee freedom from all concurrency bugs, deadlocks, or algorithmic mistakes, and does not imply the discrete-event simulator runs on a real parallel GPU runtime.

## Engineering checks before a release

1. Run `go test ./... -count=1` and `go test -race ./... -count=1`.
2. Reproduce the 1,000-task results and compare `summary.csv` with [documented baseline](data/stage5_summary.csv).
3. Reproduce the multi-seed experiment and inspect the low-arrival-rate boundary case.
4. Run the Python plot generator; verify six image files and `metrics.json`.
5. Run the Docker container end-to-end and confirm the same fixed-seed headline results.
6. Push a new commit and confirm GitHub Actions runs to completion.
