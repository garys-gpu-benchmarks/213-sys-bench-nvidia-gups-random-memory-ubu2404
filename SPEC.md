# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration. num_iterations: requested loops, capped to 2/8/24 for smoke/baseline/extended. Sweep dimensions: numa_node, num_threads, thread_affinity, page_size, table_size, num_updates, num_iterations, seed.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| numa_node | `--numa-node` | smoke=0, baseline=0, extended=0 | 0 | From Parameter list; see Execution Description With Parameters. |
| num_threads | `--num-threads` | smoke=1, baseline=4, extended=8 | 4 | From Parameter list; see Execution Description With Parameters. |
| thread_affinity | `--thread-affinity` | smoke=node:0, baseline=node:0, extended=node:0 | node:0 | From Parameter list; see Execution Description With Parameters. |
| page_size | `--page-size` | smoke=4KB, baseline=4KB, extended=4KB | 4KB | From Parameter list; see Execution Description With Parameters. |
| table_size | `--table-size` | smoke=4194304, baseline=268435456, extended=1073741824 | 268435456 | From Parameter list; see Execution Description With Parameters. |
| num_updates | `--num-updates` | smoke=1000000, baseline=50000000, extended=200000000 | 50000000 | From Parameter list; see Execution Description With Parameters. |
| num_iterations | `--num-iterations` | smoke=1, baseline=880, extended=1240 | 880 | From Parameter list; see Execution Description With Parameters. |
| seed | `--seed` | smoke=42, baseline=42, extended=42 | 42 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions
```

## Raw Output Format

CSV aggregate of the GUPS run, written as sample_index 0 and 1

sample_index,status,giga_updates_score,tlb_miss_rate,l3_cache_miss_rate,random_64_bit_update_rate,updates_sec,payload_update_gb_s,num_threads,num_iterations_ran,num_iterations_requested,microarch_measured,error_message
0,ok,0.01,0.02,0.4,1e7,1e7,0.08,8,2,8,1,

## Metrics

- **#1: Giga-updates score** — stored as `giga_updates_score`.
- **#2: Random 64-bit update rate** — stored as `updates_sec`.
- **#3: 8-byte payload update rate, GB/s** — stored as `payload_update_gb_s`.

## Framework

Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration.

## Installation and Execution Summary

Compile src/gups.c and run bin/gups with yaml table_size, num_updates, and seed, optionally wrapping with perf stat -e dTLB-load-misses,cache-misses,instructions, then derive GB/s as 8-byte updates/s, to measure random 64-bit update rate. Profile caps iterations at 2 / 8 / 24 regardless of a larger yaml request

## Platform Portability

- **AMD (primary):** ```bash
Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions
```
- **NVIDIA:** Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

CSV aggregate of the GUPS run, written as sample_index 0 and 1

sample_index,status,giga_updates_score,tlb_miss_rate,l3_cache_miss_rate,random_64_bit_update_rate,updates_sec,payload_update_gb_s,num_threads,num_iterations_ran,num_iterations_requested,microarch_measured,error_message
0,ok,0.01,0.02,0.4,1e7,1e7,0.08,8,2,8,1,

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Builds src/gups.c (OpenMP C, not C++) into bin/gups and issues random 64-bit table updates. table_size: table bytes. num_updates: updates per iteration.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
