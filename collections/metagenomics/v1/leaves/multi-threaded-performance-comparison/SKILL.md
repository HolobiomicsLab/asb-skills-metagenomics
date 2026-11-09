---
name: multi-threaded-performance-comparison
description: Use when you need to compare computational efficiency (wall-clock time
  and max memory) of Plassembler versus Unicycler across multiple thread counts on
  simulated or real hybrid (short+long read) bacterial read sets.
when_to_use_negative:
- You need assembly accuracy comparison (plasmid recovery, misassemblies) rather than
  runtime/memory benchmarking
- Your input is not a bacterial isolate hybrid read set — Unicycler is designed exclusively
  for bacterial isolates, so timing comparisons on eukaryotic or metagenomic data
  are out of scope
- You are comparing Plassembler v1.2.0 results against v1.3.0 results directly, since
  the old benchmarking used different simulated read sets (R9 vs R10), making the
  comparison unfair
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Plassembler
  role: Tool under test; run in Flye mode and Raven mode to time automated plasmid
    assembly at 1, 8, and 16 threads
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard baseline assembler whose wall-clock runtime is compared against
    Plassembler
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: Long-read assembler used inside Plassembler's default mode; a major determinant
    of runtime
  repo: https://github.com/fenderglass/Flye
- name: Raven
  role: Optional long-read assembler inside Plassembler, benchmarked as an alternative
    timing configuration
  repo: https://github.com/lbcb-sci/raven
- name: Python
  role: Used to parse run logs, aggregate medians/min/max of wall-clock time and memory,
    and compute speedup ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:0f197ae1401827c838da0c24f0fbff4f75ca789fc92c5726a656debf2ed0b853
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multi-threaded-performance-comparison

## Summary

Benchmark the wall-clock runtime of Plassembler (Flye or Raven long-read assembler modes) against Unicycler at 1, 8, and 16 threads across 20 simulated and 6 real read sets, quantifying the reported 3- to 10-fold speedup.

## When to use

Use when you need to compare computational efficiency (wall-clock time and max memory) of Plassembler versus Unicycler across multiple thread counts on simulated or real hybrid (short+long read) bacterial read sets.

## When NOT to use

- You need assembly accuracy comparison (plasmid recovery, misassemblies) rather than runtime/memory benchmarking
- Your input is not a bacterial isolate hybrid read set — Unicycler is designed exclusively for bacterial isolates, so timing comparisons on eukaryotic or metagenomic data are out of scope
- You are comparing Plassembler v1.2.0 results against v1.3.0 results directly, since the old benchmarking used different simulated read sets (R9 vs R10), making the comparison unfair

## Inputs

- Deposited run logs and timing records for Plassembler (Flye mode, Raven mode) and Unicycler runs from Zenodo record 7996690
- 20 simulated hybrid bacterial read sets and 6 real Wick et al 2021 read sets (short-read FASTQ plus long-read FASTQ)
- Thread counts: 1, 8, and 16

## Outputs

- runtime_summary_table.tsv with median, minimum, and maximum wall-clock time (s) and maximum memory (MB) per program and thread count
- Plassembler-to-Unicycler wall-clock speedup ratios computed separately for simulated and real read sets at each thread count

## How to apply

Download and unpack the deposited benchmarking run logs and timing records from Zenodo record 7996690 covering the 20 simulated read sets and the 6 real Wick et al 2021 samples. Parse the recorded wall-clock time (s) and maximum memory usage (MB) for each Plassembler run in Flye mode and Raven mode and each Unicycler (>=0.4.8) run at 1, 8, and 16 threads using Python. Aggregate replicate runs per program and thread count into median, minimum, and maximum wall-clock time and maximum memory. Compute the Plassembler-to-Unicycler speedup ratio at each thread count, separately for simulated and real read sets, and write the aggregated statistics to runtime_summary_table.tsv. Rationale: Plassembler achieves computational efficiency over Unicycler by removing chromosomal reads via a minimap2 mapping approach, so timing differences reflect the assembly scope rather than single-kernel optimizations.

## Related tools

- **Plassembler** (Tool under test; run in Flye mode and Raven mode to time automated plasmid assembly at 1, 8, and 16 threads) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard baseline assembler whose wall-clock runtime is compared against Plassembler) — https://github.com/rrwick/Unicycler
- **Flye** (Long-read assembler used inside Plassembler's default mode; a major determinant of runtime) — https://github.com/fenderglass/Flye
- **Raven** (Optional long-read assembler inside Plassembler, benchmarked as an alternative timing configuration) — https://github.com/lbcb-sci/raven
- **Python** (Used to parse run logs, aggregate medians/min/max of wall-clock time and memory, and compute speedup ratios)

## Examples

```
conda install -c bioconda plassembler
```

## Evaluation signals

- Compiled medians reproduce the claim that Plassembler is consistently 3-10x faster than Unicycler in wall-clock time
- Largest gains appear single-threaded: simulated 1-thread medians of 7012 s (Flye mode) and 3893 s (Raven mode) versus 32619 s for Unicycler
- Plassembler is faster than Unicycler for every one of the 26 samples (20 simulated, 6 real) at all three thread counts
- runtime_summary_table.tsv contains per-program, per-thread-count median/min/max wall-clock time (s) and maximum memory (MB) with no missing cells
- Speedup ratios are reported separately for simulated and real read sets rather than pooled

## Limitations

- Even Flye run with --deterministic is not deterministic between thread counts, so runtimes can vary across thread settings
- The 3- to 10-fold speedup depends on sample, thread count, and long-read assembler choice (Flye vs Raven), and does not transfer to a single fixed ratio
- QUAST-based quality evaluation was not run for these benchmarks, so timing comparisons are not paired with assembly-quality scores in the same table

## Evidence

- [abstract] It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets"
- [methods] plassembler uses Flye or optionally Raven to create an initial long-read only assembly: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [results] Even Flye with --deterministic is not deterministic between threads: "Interestingly, I found out that even Flye using `--deterministic` isn't deterministic between threads either"
