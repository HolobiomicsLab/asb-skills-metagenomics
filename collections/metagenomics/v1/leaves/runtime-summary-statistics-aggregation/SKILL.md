---
name: runtime-summary-statistics-aggregation
description: Use when benchmarking runs have recorded wall-clock time and max memory
  (e.g. from Zenodo-deposited run logs) and you need per-program, per-thread-count
  summary statistics and speedup ratios.
when_to_use_negative:
- The input is raw sequencing data or assembly outputs (FASTQ/FASTA) rather than recorded
  timing/memory logs — use the assemblers themselves instead
- The goal is assembly accuracy comparison (plasmid recovery, misassemblies) rather
  than runtime/memory benchmarking
- Only a single run per condition exists without replicates, making median/min/max
  aggregation meaningless
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Plassembler
  role: Benchmarked assembler whose runtime is being summarized (Flye and Raven long-read
    assembler modes)
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard baseline assembler against which wall-clock speedup ratios are
    computed
  repo: https://github.com/rrwick/Unicycler
- name: Python
  role: Used to parse run logs, aggregate median/min/max statistics, and write the
    summary TSV
- name: Flye
  role: Long-read assembler backend for Plassembler mode whose runtime is summarized
    (>=2.9)
  repo: https://github.com/fenderglass/Flye
- name: Raven
  role: Optional long-read assembler backend for Plassembler mode whose runtime is
    summarized (>=1.8)
  repo: https://github.com/lbcb-sci/raven
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:36cb12cec59ca0da6cc56e9ef40c9db3360478b2cb17e5db25c4ee9dfff21d4e
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# runtime-summary-statistics-aggregation

## Summary

Aggregate recorded wall-clock times and memory usage from assembler benchmarking runs (Plassembler/Flye, Plassembler/Raven, Unicycler) into median/min/max statistics per program and thread count, then compute speedup ratios to verify claims like Plassembler's 3-10x advantage over Unicycler.

## When to use

Use when benchmarking runs have recorded wall-clock time and max memory (e.g. from Zenodo-deposited run logs) and you need per-program, per-thread-count summary statistics and speedup ratios.

## When NOT to use

- The input is raw sequencing data or assembly outputs (FASTQ/FASTA) rather than recorded timing/memory logs — use the assemblers themselves instead
- The goal is assembly accuracy comparison (plasmid recovery, misassemblies) rather than runtime/memory benchmarking
- Only a single run per condition exists without replicates, making median/min/max aggregation meaningless

## Inputs

- Benchmarking run logs and timing records from Zenodo record 7996690 (Plassembler Flye mode, Plassembler Raven mode, Unicycler runs)
- Thread-count metadata (1, 8, 16 threads) and sample labels (20 simulated read sets, 6 real Wick et al 2021 samples)

## Outputs

- runtime_summary_table.tsv with median, min, max wall-clock time (s) and max memory (MB) per program and thread count
- Plassembler-to-Unicycler wall-clock speedup ratios per thread count, separately for simulated and real read sets

## How to apply

Download and unpack the deposited benchmarking outputs (run logs and timing records, e.g. Zenodo record 7996690 covering 20 simulated read sets and 6 real Wick et al 2021 samples). Parse the recorded wall-clock time and maximum memory usage for each run — Plassembler in Flye mode and Raven mode, plus Unicycler — at 1, 8, and 16 threads using Python. Aggregate replicate runs per program and thread count into median, minimum, and maximum wall clock time (s) and maximum memory (MB). Compute the Plassembler-to-Unicycler wall-clock speedup ratio at each thread count, keeping simulated and real read sets separate, and write results to runtime_summary_table.tsv. Validate against the reported claim that Plassembler is consistently 3-10x faster in wall clock time with the largest gains single-threaded (e.g. simulated 1-thread medians of 7012 s / 3893 s for Flye / Raven modes vs 32619 s for Unicycler).

## Related tools

- **Plassembler** (Benchmarked assembler whose runtime is being summarized (Flye and Raven long-read assembler modes)) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard baseline assembler against which wall-clock speedup ratios are computed) — https://github.com/rrwick/Unicycler
- **Python** (Used to parse run logs, aggregate median/min/max statistics, and write the summary TSV)
- **Flye** (Long-read assembler backend for Plassembler mode whose runtime is summarized (>=2.9)) — https://github.com/fenderglass/Flye
- **Raven** (Optional long-read assembler backend for Plassembler mode whose runtime is summarized (>=1.8)) — https://github.com/lbcb-sci/raven

## Examples

```
python parse_runtimes.py --zenodo 7996690 --programs plassembler_flye plassembler_raven unicycler --threads 1 8 16 --out runtime_summary_table.tsv
```

## Evaluation signals

- Compiled 1-thread simulated medians reproduce the paper's values: Plassembler Flye mode 7012 s, Raven mode 3893 s, Unicycler 32619 s
- Every computed speedup ratio falls within the 3-10x range reported for all 26 samples across 1, 8, and 16 threads
- runtime_summary_table.tsv contains rows for all combinations of program (Plassembler-Flye, Plassembler-Raven, Unicycler), thread count (1, 8, 16), and read-set type (simulated, real)
- Median memory usage values are consistent with the maximum memory recorded in the raw run logs (no unit mix-ups between s and MB)
- Simulated and real read set statistics remain separated; no sample is dropped (barcode08 Serratia marcescens was already excluded upstream)

## Limitations

- Flye --deterministic is not deterministic between thread counts, so runs at different thread counts are not bitwise-comparable
- Timing conclusions are specific to the benchmarked hardware and read sets (20 simulated R10 isolates, 6 real Wick et al 2021 samples); the old v1.2.0 benchmarking used different read sets (simulated R9 vs R10), so cross-version runtime comparisons are not truly fair
- QUAST evaluation was not run for these benchmarks, so this skill captures runtime only, not output quality

## Evidence

- [methods] Download and unpack the deposited benchmarking outputs (run logs and timing records) from Zenodo record 7996690: [evidence span withheld — anchor status: not_found]
- [methods] Parse the recorded wall-clock time and maximum memory usage: [evidence span withheld — anchor status: not_found]
- [methods] simulated 1-thread medians of 7012 s / 3893 s for Flye / Raven modes vs 32619 s for Unicycler: [evidence span withheld — anchor status: not_found]
- [methods] even Flye using --deterministic isn't deterministic between threads: "Interestingly, I found out that even Flye using `--deterministic` isn't deterministic between threads either"
- [readme] results themselves at zenodo record 7996690: "the results themselves [here](https://zenodo.org/record/7996690)."
