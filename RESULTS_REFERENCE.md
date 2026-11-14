# Results: 213 GUPS Random Memory

**Main goal:** Measure random 64-bit memory update performance  
**Tools:** Compile gups.c; run bin/gups; optionally wrap with perf stat -e dTLB-load-misses,cache-misses,instructions

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_141 · recorded 2026-11-14). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 PCIe (81559 MiB) · Intel(R) Xeon(R) Platinum 8352Y CPU @ 2.20GHz · Ubuntu 24.04.5 LTS · kernel 6.8.0-138-generic · driver 595.91.07
- **Run:** 2026-11-14T11:09:27Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 00:59 · benchmark 00:01 · whole run 01:09
- **Source:** run 213_20261114_110927
- **Command:** `bash run_benchmark.sh --smoke --numa-node 0 --num-threads 1 --thread-affinity node:0 --table-size 4194304 --num-updates 1000000 --num-iterations 1 --seed 42`

| # | Metric | Result |
|---|---|---|
| 1 | Giga-updates score (giga_updates_score) | 0.023618 |
| 2 | Random 64-bit update rate (updates_sec) | 23618138.163274 |
| 3 | 8-byte payload update rate, GB/s (payload_update_gb_s) | 0.18894510530619202 |
<!-- /profile:smoke -->
