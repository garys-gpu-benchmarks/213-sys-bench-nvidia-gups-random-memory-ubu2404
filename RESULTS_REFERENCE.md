# Results: 213 GUPS Random Memory

**Main goal:** Measure random 64-bit memory update performance  
**Tools:** Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_142 · recorded 2026-11-16). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 NVL (95830 MiB) · Intel(R) Xeon(R) Platinum 8452Y · Ubuntu 24.04.5 LTS · kernel 6.8.0-138-generic · driver 595.91.07
- **Run:** 2026-11-16T11:16:42Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 00:53 · benchmark 00:01 · whole run 01:00
- **Source:** run 213_20261116_111642
- **Command:** `bash run_benchmark.sh --smoke --numa-node 0 --num-threads 1 --thread-affinity node:0 --table-size 4194304 --num-updates 1000000 --num-iterations 1 --seed 42`

| # | Metric | Result |
|---|---|---|
| 1 | Giga-updates score (giga_updates_score) | 0.037087 |
| 2 | Random 64-bit update rate (updates_sec) | 37087452.621243 |
| 3 | 8-byte payload update rate, GB/s (payload_update_gb_s) | 0.296699620969944 |
<!-- /profile:smoke -->
