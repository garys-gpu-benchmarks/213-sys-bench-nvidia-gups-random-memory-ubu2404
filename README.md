# GUPS Random Memory Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/213-sys-bench-nvidia-gups-random-memory-ubu2404/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/213-sys-bench-nvidia-gups-random-memory-ubu2404/actions/workflows/ci.yml)

Target: Ubuntu 24.04 · NVIDIA · see Hardware Requirements. This is a host benchmark, not a laptop `pip install` project.

## Quick Start

```bash
git clone https://github.com/garys-gpu-benchmarks/213-sys-bench-nvidia-gups-random-memory-ubu2404.git
cd 213-sys-bench-nvidia-gups-random-memory-ubu2404
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --profile smoke --validate
```
Results are written to `results/benchmark.db` and `results/summary.json`.

This workload is executed on the validation host after the repository is copied there. `setup.sh` and `run_benchmark.sh` do not open an outbound SSH session.

Prerequisites: Ubuntu 24.04; NVIDIA; Python 3.12.3; root or sudo for `setup.sh`. Framework: Bash, SQLite, Python, PyYAML, C (not C++), GCC, OpenMP, Linux perf (optional). This is a host benchmark, not a laptop `pip install` project.

```mermaid
flowchart LR
  setup.sh --> run_benchmark.sh --> parse_results.py --> results/benchmark.db
```

## 1. Overview

Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration. num_iterations: requested loops, capped to 2/8/24 for smoke/baseline/extended. Sweep dimensions: numa_node, num_threads, thread_affinity, page_size, table_size, num_updates, num_iterations, seed.

## 2. What It Validates

- Validates GUPS throughput and TLB/L3 miss rates from the C++ random-update benchmark
- #1: Giga-updates score (giga_updates_score); is present and physically sensible.
- #2: Random 64-bit update rate (updates_sec); is present and physically sensible.
- #3: 8-byte payload update rate, GB/s (payload_update_gb_s) is present and physically sensible.

## 3. Metrics Captured

- **#1: Giga-updates score** — stored as `giga_updates_score`.
- **#2: Random 64-bit update rate** — stored as `updates_sec`.
- **#3: 8-byte payload update rate, GB/s** — stored as `payload_update_gb_s`.

## 4. Hardware Requirements

### Supported environment

- OS: Ubuntu 24.04
- GPU vendor: NVIDIA
- Framework family: Bash, SQLite, Python, PyYAML, C (not C++), GCC, OpenMP, Linux perf (optional)
- Python: Python 3.12.3

### Reference validation environment

The tables below describe the machine used to generate the reference results. They are not a requirement that every user buy that exact cloud instance.

### System

Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration.

### GPU

Ubuntu 24.04 / NVIDIA / Bash, SQLite, Python, PyYAML, C (not C++), GCC, OpenMP, Linux perf (optional)

## 5. Software Requirements

| Component | Version |
|---|---|
| OS | Ubuntu 24.04 |
| Kernel | kernel 6.8.0 |
| Python | Python 3.12.3 |
| ROCm | CUDA 12.8 |
| rocBLAS | N/A - rocBLAS not used |

Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration.

## 6. Installation

```bash
Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions
```

## 7. Running the Benchmark

```bash
Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions
```

**Validating results separately:**

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional
python3 -m venv .venv
source ".venv/bin/activate"
".venv/bin/python" scripts/validate_results.py
```

## 8. Output

### `results/benchmark.db` (SQLite)

CSV aggregate of the GUPS run, written as sample_index 0 and 1

sample_index,status,giga_updates_score,tlb_miss_rate,l3_cache_miss_rate,random_64_bit_update_rate,updates_sec,payload_update_gb_s,num_threads,num_iterations_ran,num_iterations_requested,microarch_measured,error_message
0,ok,0.01,0.02,0.4,1e7,1e7,0.08,8,2,8,1,

```bash
Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions
```

### `results/summary.json`

Consolidated metrics from the most recent run — suitable for CI artifact upload or dashboard ingestion.

### `results/raw/<timestamp>.txt`

CSV aggregate of the GUPS run, written as sample_index 0 and 1

sample_index,status,giga_updates_score,tlb_miss_rate,l3_cache_miss_rate,random_64_bit_update_rate,updates_sec,payload_update_gb_s,num_threads,num_iterations_ran,num_iterations_requested,microarch_measured,error_message
0,ok,0.01,0.02,0.4,1e7,1e7,0.08,8,2,8,1,

## 9. Baselines / Thresholds

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

## 10. Troubleshooting

**`setup.sh` missing collector**
Create cannot finish without `scripts/collect_workload.py`.

**`self_check` overlay rewritten**
Do not overwrite files listed in `results/overlay_lock.json`.

**Remote SSH drop during setup**
Reconnect and resume `bash setup.sh --assume-yes`. Do not wipe `.venv` or `.cache`.

## 11. NVIDIA H100 Coding Differences

Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Repository layout

```text
.
├── setup.sh
├── run_benchmark.sh
├── benchmark_specification.json
├── .github/workflows/      # thin CI callers (see Continuous Integration)
├── config/
├── scripts/
├── src/
├── tests/
├── docs/
├── results/
└── LICENSE
```

## Continuous Integration

| Workflow | Runs on | When | What it does |
|---|---|---|---|
| [CI](.github/workflows/ci.yml) | GitHub-hosted runner | every pull request, and every push to `main` | shellcheck, ruff, `bash -n`, `compileall`, `run_benchmark.sh --help`, specification schema, the results validator on a seeded fixture, required files, and actionlint. No GPU and no benchmark run. |
| [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) | self-hosted runner labeled `gpu`, `nvidia`, `ubu2404` | only when started by hand: **Actions → GPU Smoke Benchmark → Run workflow** (choose `smoke`, `baseline` or `extended`) | Verifies the pre-provisioned GPU stack, records `results/environment.json` (driver, runtime, kernel, GPU), runs the profile with `--validate`, shows headline metrics on the run page, and uploads the results. |

Both files are short callers. The steps themselves live once, for every workload in the suite, in [`garys-gpu-benchmarks/shared-workflows`](https://github.com/garys-gpu-benchmarks/shared-workflows), pinned at `@v1`. The GPU workflow is never triggered by pull requests, so code from a fork cannot run on the GPU host.

### Running it as part of the NVIDIA Ubuntu 24.04 bundle

This repository is one of the 32 workloads in [`bundle-nvidia-ubuntu-2404`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404), which holds them as git submodules. To put the whole bundle on a GPU host and run this workload from it:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404 /opt/benchmarks
cd /opt/benchmarks/213-sys-bench-nvidia-gups-random-memory-ubu2404
bash run_benchmark.sh --profile smoke --validate
```
