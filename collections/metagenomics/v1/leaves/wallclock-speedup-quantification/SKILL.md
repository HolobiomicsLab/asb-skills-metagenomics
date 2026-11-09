---
name: wallclock-speedup-quantification
description: Use when reproducing or re-verifying runtime benchmark claims for Plassembler
  vs Unicycler, e.g. from run logs/timing records in Zenodo record 7996690, or when
  measuring assembler speedup across thread counts.
when_to_use_negative:
- Evaluating assembly accuracy rather than runtime — the article's accuracy claims
  (e.g. missed plasmid counts, the linear plasmid misassembly in K. variicola) require
  assembly comparison, not timing records
- Comparing against assemblers other than Unicycler (e.g. Dragonflye) — no timing
  records for those tools are present in the deposited benchmarking outputs
- Input read sets differ from the benchmarked ones (e.g. simulated R9 vs R10), since
  the article notes such comparisons are 'not truly a fair comparison'
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Parses recorded wall-clock and memory values from run logs and aggregates
    them into median/min/max statistics and speedup ratios
  repo: https://github.com/gbouras13/plassembler
- name: Plassembler
  role: Benchmarked tool whose runtime (Flye and Raven modes) is measured and compared
    to Unicycler
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard baseline assembler whose runtime provides the denominator of
    the speedup ratio
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: Long-read assembler used in Plassembler Flye mode for one set of timed runs
  repo: https://github.com/mikolmogorov/Flye
- name: Raven
  role: Alternative long-read assembler used in Plassembler Raven mode for a second
    set of timed runs
  repo: https://github.com/lbcb-sci/raven
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:1e326b0134f78c6489fccb1de2225a58290240ff7cf8da0ee668557fdbdb18cd
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# wallclock-speedup-quantification

## Summary

Quantifies wall-clock runtime (and peak memory) speedup of Plassembler (Flye or Raven modes) versus Unicycler across 20 simulated and 6 real read sets at 1, 8, and 16 threads, verifying the paper's 3- to 10-fold improvement claim using deposited Zenodo benchmarking outputs.

## When to use

Use when reproducing or re-verifying runtime benchmark claims for Plassembler vs Unicycler, e.g. from run logs/timing records in Zenodo record 7996690, or when measuring assembler speedup across thread counts.

## When NOT to use

- Evaluating assembly accuracy rather than runtime — the article's accuracy claims (e.g. missed plasmid counts, the linear plasmid misassembly in K. variicola) require assembly comparison, not timing records
- Comparing against assemblers other than Unicycler (e.g. Dragonflye) — no timing records for those tools are present in the deposited benchmarking outputs
- Input read sets differ from the benchmarked ones (e.g. simulated R9 vs R10), since the article notes such comparisons are 'not truly a fair comparison'

## Inputs

- Deposited benchmarking run logs and timing records from Zenodo record 7996690
- Wall-clock time (s) and maximum memory usage (MB) per run for Plassembler (Flye and Raven modes) and Unicycler
- Sample metadata distinguishing 20 simulated read sets and 6 real Wick et al 2021 read sets, with thread counts 1, 8, 16

## Outputs

- runtime_summary_table.tsv with median/min/max wall clock time (s) and maximum memory (MB) per program and thread count
- Plassembler-to-Unicycler speedup ratios at each thread count, split by simulated vs real read sets

## How to apply

Download and unpack the benchmarking run logs and timing records from Zenodo record 7996690, which cover the 20 simulated read sets and the 6 real Wick et al 2021 samples. Use Python to parse the recorded wall-clock time (s) and maximum memory (MB) for each Plassembler (Flye mode and Raven mode) and Unicycler run at 1, 8, and 16 threads. Aggregate replicate runs per program and thread count into median, minimum, and maximum values, then compute the Plassembler-to-Unicycler wall-clock speedup ratio separately for simulated and real read sets. Write the aggregated statistics to runtime_summary_table.tsv and validate that the compiled medians reproduce the reported pattern that Plassembler is consistently 3-10x faster, with the largest gains single-threaded (e.g. simulated 1-thread medians of 7012 s / 3893 s for Flye / Raven modes vs 32619 s for Unicycler).

## Related tools

- **Python** (Parses recorded wall-clock and memory values from run logs and aggregates them into median/min/max statistics and speedup ratios) — https://github.com/gbouras13/plassembler
- **Plassembler** (Benchmarked tool whose runtime (Flye and Raven modes) is measured and compared to Unicycler) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard baseline assembler whose runtime provides the denominator of the speedup ratio) — https://github.com/rrwick/Unicycler
- **Flye** (Long-read assembler used in Plassembler Flye mode for one set of timed runs) — https://github.com/mikolmogorov/Flye
- **Raven** (Alternative long-read assembler used in Plassembler Raven mode for a second set of timed runs) — https://github.com/lbcb-sci/raven

## Examples

```
python aggregate_runtimes.py --indir zenodo_7996690 --threads 1 8 16 --tools plassembler_flye plassembler_raven unicycler --out runtime_summary_table.tsv
```

## Evaluation signals

- Speedup ratio falls in the reported 3- to 10-fold range for every sample (20 simulated + 6 real) at all thread counts (1, 8, 16)
- Simulated 1-thread medians reproduce ~7012 s (Plassembler Flye mode), ~3893 s (Raven mode), and ~32619 s (Unicycler)
- runtime_summary_table.tsv contains median, minimum, and maximum wall clock time (s) and maximum memory (MB) for each program/thread combination
- Speedup is computed separately for simulated vs real read sets, with the largest gains observed single-threaded
- Each of the 26 samples appears for both Plassembler modes and Unicycler at all three thread counts before aggregation

## Limitations

- Flye --deterministic is not deterministic between thread counts ('even Flye using --deterministic isn't deterministic between threads either'), so replicate timings can vary across thread settings
- The largest speedups depend on sample, thread count, and long-read assembler choice (Flye vs Raven); gains at higher thread counts are smaller than single-threaded
- Runtime comparison is only fair for the same read sets — the v1.2.0 benchmarks used different read sets (simulated R9 vs R10) and are explicitly noted as 'not truly a fair comparison'

## Evidence

- [abstract] increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads"
- [methods] Zenodo benchmarking outputs: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
- [methods] Flye non-determinism across threads: "even Flye using `--deterministic` isn't deterministic between threads either"
- [results] Unfair comparison caveat for cross-version runtime claims: "they were done with different read sets (simulated R9 vs R10) so it's not truly a fair comparison"
- [readme] Benchmarking pipeline repository: [evidence span withheld — anchor status: external_doc]
