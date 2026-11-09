---
name: small-plasmid-recovery-comparison
description: Use when comparing plasmid assembly tools on real (not simulated) bacterial
  read sets, to quantify genome fraction and whether small plasmids missed by Unicycler
  are recovered by Plassembler.
when_to_use_negative:
- Sample is highly heterogeneous (e.g. barcode08 Serratia marcescens), which was excluded
  from benchmarking because it was difficult even for Trycycler manual curation
- Relying on QUAST-like accuracy numbers for the plassembler changelog (long-only
  kmer_mode) benchmarks — QUAST was not run there ('I was too lazy to run QUAST')
- Comparing against the older v1.2.0 results — those used different simulated read
  sets (R9 vs R10), so the comparison is not truly fair
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Plassembler
  role: Primary plasmid assembler under evaluation; recovers small plasmids Unicycler
    misses via long-read-first hybrid assembly
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard comparison assembler (>=0.4.8) run on the same real read sets
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: Long-read assembler (>=2.9, nano-hq mode) generating Plassembler's initial
    long-read-only assembly
  repo: https://github.com/mikolmogorov/Flye
- name: Raven
  role: Alternative long-read assembler (>=1.8) for the Plassembler initial assembly
  repo: https://github.com/lbcb-sci/raven
- name: QUAST
  role: Computes genome fraction, indels and mismatches per 100 kbp against ground
    truth plasmids
- name: minimap2
  role: Maps long and short reads (>=2.11) for chromosomal read removal in Plassembler's
    workflow
  repo: https://github.com/lh3/minimap2
- name: Small-plasmid-Nanopore
  role: Source of independent Trycycler-derived manually curated ground truth plasmids
    and read sets
  repo: https://github.com/rrwick/Small-plasmid-Nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:a764f93797997e94abead5df5882cd17604874780f2a50bbc0ef2bf46e9ae6fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Flye
  - Plassembler
  - QUAST
  - Raven
  - Small-plasmid-Nanopore
  - Unicycler
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# small-plasmid-recovery-comparison

## Summary

Benchmarks Plassembler (Flye- or Raven-based long-read-first hybrid assembly) against Unicycler on real Nanopore read sets, scoring plasmid recovery against Trycycler-derived manually curated ground truth with QUAST, with specific attention to small plasmids (<10 kbp) that Unicycler misses.

## When to use

Use when comparing plasmid assembly tools on real (not simulated) bacterial read sets, to quantify genome fraction and whether small plasmids missed by Unicycler are recovered by Plassembler.

## When NOT to use

- Sample is highly heterogeneous (e.g. barcode08 Serratia marcescens), which was excluded from benchmarking because it was difficult even for Trycycler manual curation
- Relying on QUAST-like accuracy numbers for the plassembler changelog (long-only kmer_mode) benchmarks — QUAST was not run there ('I was too lazy to run QUAST')
- Comparing against the older v1.2.0 results — those used different simulated read sets (R9 vs R10), so the comparison is not truly fair

## Inputs

- Pooled 60x subsampled Nanopore long-read FASTQs for the six Wick et al. 2021 isolates (A. baumannii J9, C. koseri MINF 9D, E. kobei MSB1 1B, Haemophilus sp002998595 M1C132 1, K. oxytoca MSB1 2C, K. variicola INF345)
- Illumina short-read FASTQ pairs for the same isolates
- Trycycler-derived manually curated ground truth plasmid FASTAs from the rrwick/Small-plasmid-Nanopore deposit
- Plassembler PLSDB-based database
- QUAST for assembly evaluation

## Outputs

- Plasmid FASTA assemblies from Plassembler (Flye and Raven modes) and Unicycler per isolate
- real_reads_assembly_accuracy_table.tsv with genome fraction (mean/median/min/max), indels and mismatches per 100 kbp (median/min/max), additional plasmids recovered, total missed plasmids, fragmented plasmids and misassemblies per program

## How to apply

Obtain the pooled 60x subsampled real-read FASTQs for the six Wick et al. 2021a isolates (e.g. K. variicola INF345, K. oxytoca MSB1 2C) and the Trycycler-derived manually curated ground truth plasmids from the Small-plasmid-Nanopore deposit. Assemble plasmids with Plassembler driven by Flye (>=2.9, nano-hq mode) and optionally Raven (>=1.8), and run Unicycler (>=0.4.8) on the same read sets, using 8 threads to match the reported i7-10700K Ubuntu 20.04.6 benchmarking conditions. Evaluate each assembly against ground truth with QUAST, computing genome fraction (mean/median/min/max) plus indels and mismatches per 100 kbp (median, min, max). Tabulate additional plasmids recovered, total missed, fragmented plasmids and misassemblies per program, writing real_reads_assembly_accuracy_table.tsv. Validate against the reported result: Plassembler and Unicycler have identical genome fractions (mean 97.02, median 100) with zero misassemblies, but Plassembler recovers the 1934 bp K. variicola INF345 and 10697 bp K. oxytoca MSB1 2C plasmids (2 additional recovered, 2 total missed) that Unicycler misses (0 additional, 2 total missed).

## Related tools

- **Plassembler** (Primary plasmid assembler under evaluation; recovers small plasmids Unicycler misses via long-read-first hybrid assembly) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard comparison assembler (>=0.4.8) run on the same real read sets) — https://github.com/rrwick/Unicycler
- **Flye** (Long-read assembler (>=2.9, nano-hq mode) generating Plassembler's initial long-read-only assembly) — https://github.com/mikolmogorov/Flye
- **Raven** (Alternative long-read assembler (>=1.8) for the Plassembler initial assembly) — https://github.com/lbcb-sci/raven
- **QUAST** (Computes genome fraction, indels and mismatches per 100 kbp against ground truth plasmids)
- **minimap2** (Maps long and short reads (>=2.11) for chromosomal read removal in Plassembler's workflow) — https://github.com/lh3/minimap2
- **Small-plasmid-Nanopore** (Source of independent Trycycler-derived manually curated ground truth plasmids and read sets) — https://github.com/rrwick/Small-plasmid-Nanopore

## Examples

```
plassembler -l long_reads.fastq -1 R1.fastq -2 R2.fastq -d plassembler_db -o plassembler_out -t 8
```

## Evaluation signals

- Plassembler and Unicycler assemblies show identical genome fractions (mean 97.02, median 100) against ground truth with low indel/mismatch rates and zero misassemblies
- Plassembler recovers the 1934 bp K. variicola INF345 and 10697 bp K. oxytoca MSB1 2C plasmids (2 additional recovered, 2 total missed) while Unicycler recovers none additionally (2 total missed)
- QUAST output contains genome fraction mean/median/min/max and indels and mismatches per 100 kbp (median, min, max) for each program
- All three programs ran with 8 threads under the reported benchmark conditions (Intel Core i7-10700K, Ubuntu 20.04.6)
- real_reads_assembly_accuracy_table.tsv is populated for all six Wick et al. 2021 isolates

## Limitations

- The linear plasmid in K. variicola (30408 bp recovered incomplete vs 31780 bp ground truth) is a known misassembly issue of the Unicycler approach
- barcode08 Serratia marcescens was excluded from benchmarking due to sample heterogeneity that defeated even Trycycler curation
- Long-only kmer_mode for very high quality Nanopore reads (R10.4+) is experimental and unvalidated, especially for small plasmids

## Evidence

- [methods] plassembler recovered 2 additional small plasmids missed by Unicycler of lengths 1934 bp: "`plassembler` recovered 2 additional small plasmids missed by Unicycler of lengths 1934 bp  _(_K. variicola_ INF345) and 10697 bp (_K. oxytoca_ MSB1 2C)."
- [methods] initial long-read only assembly with Flye or Raven: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [methods] independent curated ground truth source: "independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md)"
- [readme] real read sets require custom FASTQ paths in benchmarking repo: [evidence span withheld — anchor status: external_doc]
