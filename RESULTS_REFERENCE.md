# Results: 213 GUPS Random Memory

**Main goal:** Measure random 64-bit memory update performance  
**Tools:** Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions

Reference results from the validation run on the system shown in each section (recorded 2026-11-22). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 80GB HBM3 (81559 MiB) · Intel(R) Xeon(R) Platinum 8470 · Ubuntu 24.04.5 LTS · kernel 6.8.0-149-generic · driver 595.112.07
- **Run:** 2026-11-22T14:21:27Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 01:00 · benchmark 00:01 · whole run 01:06
- **Source:** run 213_20261122_142127
- **Command:** `bash run_benchmark.sh --smoke --numa-node 0 --num-threads 1 --thread-affinity node:0 --table-size 4194304 --num-updates 1000000 --num-iterations 1 --seed 42`

| # | Metric | Result |
|---|---|---|
| 1 | Giga-updates score (giga_updates_score) | 0.041732 |
| 2 | Random 64-bit update rate (updates_sec) | 41732478.785712 |
| 3 | 8-byte payload update rate, GB/s (payload_update_gb_s) | 0.33385983028569605 |
<!-- /profile:smoke -->
