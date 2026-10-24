# Publishing checklist (Windows / VS Code)

## 1. Add this documentation pack

The ZIP is a **documentation + CI overlay**. It contains **no replacement `*.go`, Dockerfile, analytics script or requirements file**. Unzip into the root of your already working local project. If you have an existing README or `.gitignore`, back it up or merge carefully.

Expected additions: `README.md`, `.gitignore`, `.github/workflows/ci.yml`, `docs/ARCHITECTURE.md`, `docs/BENCHMARK.md`, `docs/TESTING.md`, `docs/REPRODUCE.md`, `docs/RESUME_BULLETS.md`, `docs/data/*`, `docs/assets/*`.

## 2. Confirm original source code remains

Ensure `go.mod`, `main.go`, `model.go`, `allocator.go`, `cluster.go`, `priority.go`, `adaptive.go`, `benchmark.go`, `robustness.go`, `analytics/plot_results.py`, `Dockerfile`, and `docker/run-all.sh` still exist. Keep all current `_test.go` files.

```powershell
go test ./... -count=1
go run . -stage 5 -tasks 1000 -seed 20261024 -out stage5_results
```

The repository should have no accidental secrets or private files. Review files with `git status --short` before committing.

## 3. Initialize Git locally (only if not already a Git repository)

```powershell
git status
```

If Git reports that the directory is **not** a Git repository:

```powershell
git init
git branch -M main
```

If it already *is* a repository, skip initialization. Then:

```powershell
git add .
git status --short
git commit -m "Prepare GPU scheduling simulator portfolio release"
```

`.gitignore` excludes local venvs, generated workloads and full benchmark logs by default. Curated `docs/data/` reference CSVs and `docs/assets/` plots **are** committed. Do not commit account credentials, tokens, local private data or downloaded archives.

## 4. Push to GitHub

Create a **new empty repository** on GitHub, suggested name `gpu-workload-scheduling-simulator`. Avoid initializing the remote with a README/.gitignore at creation since those already exist locally. Copy your repository URL and run:

```powershell
git remote add origin https://github.com/YOUR-USERNAME/gpu-workload-scheduling-simulator.git
git push -u origin main
```

**Replace** `YOUR-USERNAME` with your actual GitHub username. If `origin` already exists, check `git remote -v` and adjust your existing remote rather than adding a duplicate. GitHub authentication may open a browser sign-in flow.

Make it Public only after reviewing the repository content and access rights.

## 5. Verify portfolio presentation

- [ ] GitHub README renders tables, architecture Mermaid diagram and four embedded performance charts.
- [ ] `docs/` includes methodology and explicit simulation limitations.
- [ ] `docs/data/` includes fixed-seed and multi-seed CSV summaries.
- [ ] `.github/workflows/ci.yml` is visible and GitHub Actions status is green.
- [ ] Local code/tests and Docker reproduction continue to pass.
- [ ] Repository Topics: `golang`, `gpu-scheduling`, `discrete-event-simulation`, `systems-engineering`, `python`, `docker`, `benchmarking`.
- [ ] Add repository link to tech resume/portfolio/LinkedIn only after verifying it opens publicly.

## 6. Recommended next improvements

Optional: matrix tests on Windows and Linux; additional workload generators (bursty, long-tailed); per-GPU fragmentation figures; starvation/fairness metrics and 95% confidence intervals; algorithm complexity and larger clusters.
