---
name: benchmarking-log-parsing
description: Use when deposited assembler benchmark logs (e.g. Zenodo record 7996690)
  contain per-run timing/memory records across thread counts and you must reproduce
  or re-aggregate speed claims such as the 3- to 10-fold Plassembler vs Unicycler
  improvement.
when_to_use_negative:
- The goal is assembly accuracy comparison (plasmid recovery, misassemblies) rather
  than runtime/memory benchmarking — this skill only handles timing and memory records
- The source data are raw FASTQ read sets rather than completed assembler run logs,
  so no timing records exist to parse
- Comparing against tools or versions not present in the deposited benchmarking outputs
  (e.g. the older v1.2.0 benchmarks used different simulated R9 vs R10 read sets and
  are not a fair comparison)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Plassembler
  role: Benchmarked assembler whose Flye-mode and Raven-mode run timings are parsed
    and compared against Unicycler
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard baseline assembler whose wall-clock times form the denominator
    of the speedup ratios
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: Long-read assembler used in Plassembler Flye mode; its run records are among
    the parsed logs
  repo: https://github.com/fenderglass/Flye
- name: Raven
  role: Optional long-read assembler used in Plassembler Raven mode; its run records
    are among the parsed logs
  repo: https://github.com/lbcb-sci/raven
- name: Python
  role: Language used to parse timing records, aggregate medians/min/max, compute
    speedup ratios, and write runtime_summary_table.tsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:944b810c2354588b40e0879bd1efe164166d005fb4af20b4d67b46c04aa1eaf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Flye
  - Plassembler
  - Python
  - Raven
  - Unicycler
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benchmarking-log-parsing

## Summary

Parse recorded wall-clock run times and maximum memory usage from deposited benchmarking outputs (Plassembler/Flye, Plassembler/Raven, and Unicycler runs) and aggregate them into per-thread-count summary statistics and Plassembler-to-Unicycler speedup ratios.

## When to use

Use when deposited assembler benchmark logs (e.g. Zenodo record 7996690) contain per-run timing/memory records across thread counts and you must reproduce or re-aggregate speed claims such as the 3- to 10-fold Plassembler vs Unicycler improvement.

## When NOT to use

- The goal is assembly accuracy comparison (plasmid recovery, misassemblies) rather than runtime/memory benchmarking — this skill only handles timing and memory records
- The source data are raw FASTQ read sets rather than completed assembler run logs, so no timing records exist to parse
- Comparing against tools or versions not present in the deposited benchmarking outputs (e.g. the older v1.2.0 benchmarks used different simulated R9 vs R10 read sets and are not a fair comparison)

## Inputs

- Deposited benchmarking run logs and timing records from Zenodo record 7996690 (20 simulated read sets + 6 real Wick et al 2021 samples)
- Per-run wall-clock time (s) and maximum memory usage (MB) records for Plassembler (Flye and Raven modes) and Unicycler at 1, 8, and 16 threads

## Outputs

- runtime_summary_table.tsv with per-program, per-thread-count median/min/max wall-clock time and maximum memory
- Plassembler-to-Unicycler wall-clock speedup ratios stratified by thread count and by simulated vs real read sets

## How to apply

Download and unpack the benchmarking outputs (run logs and timing records) from Zenodo record 7996690 covering the 20 simulated read sets and the 6 real Wick et al 2021 samples. Use Python to parse the recorded wall-clock time (s) and maximum memory (MB) for each Plassembler run in Flye mode and Raven mode and each Unicycler run at 1, 8, and 16 threads. Aggregate replicate runs per program and thread count into median, minimum, and maximum wall-clock time and maximum memory. Compute the Plassembler-to-Unyccler speedup ratio at each thread count, separately for simulated and real read sets, and write results to runtime_summary_table.tsv. Validate that the compiled medians reproduce the reported claim: Plassembler was faster for every sample at every thread count, with 3- to 10-fold gains (simulated 1-thread medians of 7012 s / 3893 s for Flye / Raven modes vs 32619 s for Unicycler).

## Related tools

- **Plassembler** (Benchmarked assembler whose Flye-mode and Raven-mode run timings are parsed and compared against Unicycler) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard baseline assembler whose wall-clock times form the denominator of the speedup ratios) — https://github.com/rrwick/Unicycler
- **Flye** (Long-read assembler used in Plassembler Flye mode; its run records are among the parsed logs) — https://github.com/fenderglass/Flye
- **Raven** (Optional long-read assembler used in Plassembler Raven mode; its run records are among the parsed logs) — https://github.com/lbcb-sci/raven
- **Python** (Language used to parse timing records, aggregate medians/min/max, compute speedup ratios, and write runtime_summary_table.tsv)

## Examples

```
python parse_bench_logs.py --zenodo 7996690 --programs plassembler_flye plassembler_raven unicycler --threads 1 8 16 --out runtime_summary_table.tsv
```

## Evaluation signals

- Medians reproduce the reported claim: Plassembler faster than Unicycler for every sample at all thread counts, with 3- to 10-fold speedup
- Simulated 1-thread medians match the reference values: 7012 s (Plassembler/Flye), 3893 s (Plassembler/Raven), 32619 s (Unicycler)
- runtime_summary_table.tsv contains one median/min/max row per program-mode x thread-count (1, 8, 16) x read-set-type (simulated, real) combination
- All 26 read sets (20 simulated + 6 real Wick et al 2021 samples) are represented for every program and thread count
- Speedup ratios fall within the 3-10x range, with the largest gains observed single-threaded

## Limitations

- Flye run even with --deterministic is not deterministic between thread counts, so replicate timings can vary
- Comparison is only valid within the deposited benchmarking outputs; older Plassembler v1.2.0 benchmarks used different read sets (simulated R9 vs R10) so cross-version timing comparisons are not truly fair
- barcode08 Serratia marcescens was excluded from the benchmarking due to sample heterogeneity, so results do not cover that sample

## Evidence

- [abstract] Zenodo benchmarking outputs location: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
- [methods] Assembler modes parsed in the logs: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [abstract] Computational efficiency claim being reproduced: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler"
- [methods] Non-determinism caveat: "even Flye using `--deterministic` isn't deterministic between threads either"
