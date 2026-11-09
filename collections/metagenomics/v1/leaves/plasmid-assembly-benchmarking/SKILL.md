---
name: plasmid-assembly-benchmarking
description: Use when comparing plasmid assembly tools on simulated or real (pooled,
  subsampled) Nanopore read sets against an independently curated plasmid ground truth,
  to quantify accuracy (QUAST) and plasmid recovery/missed-plasmid counts.
when_to_use_negative:
- When no independently curated ground truth (e.g. Trycycler-derived plasmid sequences)
  is available, so QUAST accuracy and missed-plasmid counts cannot be computed
- When the sample is heterogeneous or too difficult for ground-truth assembly — e.g.
  barcode08 Serratia marcescens was excluded from this benchmarking because Trycycler
  assembly was unreliable
- When the goal is whole-chromosome (not plasmid) assembly quality assessment, since
  this pipeline scores only plasmid recovery against plasmid ground truth
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0091
tools:
- name: Plassembler
  role: Plasmid assembler under test (long-read assembly with Flye or Raven followed
    by hybrid assembly of extracted reads)
  repo: https://github.com/gbouras13/plassembler
- name: Flye >=2.9
  role: Long-read assembler backend for Plassembler's initial long-read-only assembly
    (nano-hq mode)
  repo: https://github.com/fenderglass/Flye
- name: Raven >=1.8
  role: Alternative long-read assembler backend for Plassembler's initial long-read-only
    assembly
  repo: https://github.com/lbcb-sci/raven
- name: Unicycler >=0.4.8
  role: Gold-standard comparator assembler run on the same read sets
  repo: https://github.com/rrwick/Unicycler
- name: QUAST
  role: Evaluates each plasmid assembly against the ground truth to compute genome
    fraction, indels per 100 kbp, and mismatches per 100 kbp
- name: rasusa
  role: Subsamples fixed FASTQ files to a target coverage (50x then 60x) for consistent
    benchmarking input
  repo: https://github.com/mbhall88/rasusa
- name: minimap2 >=2.11
  role: Maps long and short reads to the long-read assembly during Plassembler's chromosome-removal
    step preceding benchmarked assembly
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:98f718a1aa4e6bbe444f013e1837f194fc0b0537d73872b83c9cad8680008ef6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Flye
  - Flye >=2.9
  - Plassembler
  - QUAST
  - Raven
  - Raven >=1.8
  - Unicycler
  - Unicycler >=0.4.8
  - minimap2
  - minimap2 >=2.11
  - rasusa
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# plasmid-assembly-benchmarking

## Summary

Benchmark plasmid assemblers (Plassembler-Flye, Plassembler-Raven, Unicycler) against a Trycycler-derived manually curated ground truth on Nanopore/Illumina read sets, scoring assemblies with QUAST (genome fraction, indels/mismatches per 100 kbp) plus counts of missed, additional, fragmented, and misassembled plasmids.

## When to use

Use when comparing plasmid assembly tools on simulated or real (pooled, subsampled) Nanopore read sets against an independently curated plasmid ground truth, to quantify accuracy (QUAST) and plasmid recovery/missed-plasmid counts.

## When NOT to use

- When no independently curated ground truth (e.g. Trycycler-derived plasmid sequences) is available, so QUAST accuracy and missed-plasmid counts cannot be computed
- When the sample is heterogeneous or too difficult for ground-truth assembly — e.g. barcode08 Serratia marcescens was excluded from this benchmarking because Trycycler assembly was unreliable
- When the goal is whole-chromosome (not plasmid) assembly quality assessment, since this pipeline scores only plasmid recovery against plasmid ground truth

## Inputs

- Pooled, coverage-subsampled (60x) Nanopore FASTQ read sets for the six Wick et al. 2021 isolates (A. baumannii J9, C. koseri MINF 9D, E. kobei MSB1 1B, Haemophilus sp002998595 M1C132 1, K. oxytoca MSB1 2C, K. variicola INF345)
- Simulated read FASTQs and assemblies for 20 samples (Zenodo 10.5281/zenodo.7996690)
- Trycycler-derived, manually curated ground truth plasmid FASTA sequences (Small-plasmid-Nanopore deposit)
- Assembler output plasmid FASTA files from Plassembler (Flye/Raven) and Unicycler

## Outputs

- Per-program QUAST accuracy table: genome fraction (mean/median/min/max) and indels and mismatches per 100 kbp (median, min, max)
- Per-program plasmid recovery counts: missed, additional, fragmented/incomplete, and misassembled plasmids
- Summary comparison table (e.g. real_reads_assembly_accuracy_table.tsv), one row per program

## How to apply

Obtain read FASTQs (e.g. Wick et al. 2021a real read sets subsampled to 60x with rasusa, or the 20 simulated read sets from Zenodo 10.5281/zenodo.7996690) and the Trycycler-derived ground truth plasmids from the Small-plasmid-Nanopore deposit. Run each assembler — Plassembler with Flye (>=2.9, nano-hq) or Raven (>=1.8), and Unicycler (>=0.4.8) — under matched conditions (e.g. 8 threads on the benchmark machine) to produce plasmid FASTAs. Run QUAST for each program against the ground truth to extract genome fraction (mean/median/min/max) and indels and mismatches per 100 kbp (median, min, max). Separately diff each assembly's contig inventory against the ground truth plasmid list to count missed, additional, fragmented/incomplete, and misassembled plasmids. Aggregate per-sample metrics into a summary table (one row per program, e.g. real_reads_assembly_accuracy_table.tsv) and validate against published figures (e.g. Plassembler-Flye genome fraction mean 99.78; real reads: Plassembler and Unicycler identical mean 97.02/median 100 genome fraction with zero misassemblies).

## Related tools

- **Plassembler** (Plasmid assembler under test (long-read assembly with Flye or Raven followed by hybrid assembly of extracted reads)) — https://github.com/gbouras13/plassembler
- **Flye >=2.9** (Long-read assembler backend for Plassembler's initial long-read-only assembly (nano-hq mode)) — https://github.com/fenderglass/Flye
- **Raven >=1.8** (Alternative long-read assembler backend for Plassembler's initial long-read-only assembly) — https://github.com/lbcb-sci/raven
- **Unicycler >=0.4.8** (Gold-standard comparator assembler run on the same read sets) — https://github.com/rrwick/Unicycler
- **QUAST** (Evaluates each plasmid assembly against the ground truth to compute genome fraction, indels per 100 kbp, and mismatches per 100 kbp)
- **rasusa** (Subsamples fixed FASTQ files to a target coverage (50x then 60x) for consistent benchmarking input) — https://github.com/mbhall88/rasusa
- **minimap2 >=2.11** (Maps long and short reads to the long-read assembly during Plassembler's chromosome-removal step preceding benchmarked assembly) — https://github.com/lh3/minimap2

## Examples

```
quast.py plassembler_flye_plasmids.fasta -r ground_truth_plasmids.fasta -o quast_plassembler_flye && python scripts/compare_plasmid_inventories.py --assembly plassembler_flye_plasmids.fasta --truth ground_truth_plasmids.fasta
```

## Evaluation signals

- On the six real read sets, Plassembler and Unicycler reproduce identical genome fractions (mean 97.02, median 100) with zero misassemblies
- On real reads, Plassembler recovers the two additional small plasmids Unicycler misses: 1934 bp (K. variicola INF345) and 10697 bp (K. oxytoca MSB1 2C) — 2 additional / 2 missed for Plassembler vs 0 additional / 2 missed for Unicycler
- On the 20 simulated read sets, recomputed counts match: missed plasmids 1 (both Plassembler variants) vs 7 (Unicycler); fragmented assemblies 3 (Plassembler-Flye) / 5 (Plassembler-Raven) vs 1 (Unicycler)
- Simulated-read genome fraction matches published figures, e.g. Plassembler-Flye 99.78 mean / 99.97 median vs Unicycler 93.81 mean / 99.88 median
- The only misassembly is the K. variicola linear plasmid recovered incomplete (30408 bp vs 31780 bp ground truth), a known limitation of the Unicycler approach

## Limitations

- Flye with --deterministic is not deterministic between thread counts, so byte-identical assemblies cannot be expected across thread settings
- The only misassembly observed was the linear plasmid in K. variicola (30408 bp recovered vs 31780 bp linear ground truth), a known issue of the Unicycler approach
- The old v1.2.0 benchmarking used different read sets (simulated R9 vs R10), so cross-version comparisons are not truly fair
- barcode08 Serratia marcescens was excluded from benchmarking due to sample heterogeneity that defied even Trycycler assembly

## Evidence

- [methods] missed only 1 plasmid (2370 bp plasmid in Enterobacter kobei MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp).: "`plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7"
- [methods] recovered 2 additional small plasmids missed by Unyccler of lengths 1934 bp (K. variicola INF345) and 10697 bp (K. oxytoca MSB1 2C).: "`plassembler` recovered 2 additional small plasmids missed by Unicycler of lengths 1934 bp"
- [methods] I ran rasusa to get 50x coverage on each fixed fastq file to reduce the read sizes: "I ran [rasusa](https://github.com/mbhall88/rasusa) to get 50x coverage on each fixed fastq file to reduce the read sizes"
- [discussion] Adds subsampling with rasusa, and -s to change subsampling depth, and --no_subsample to turn off subsampling: "Adds subsampling with rasusa, and `-s` to change subsampling depth, and `--no_subsample` to turn off subsampling"
- [readme] For the real read sets, you will need to change the csv input files in this directory with the correct FASTQ paths of the: [evidence span withheld — anchor status: external_doc]
- [results] The only misassembly is of the linear plasmid in K variicola, which is a known issue of the Unicycler approach: "The only misassembly is of the linear plasmid in _K variicola_ , which is a known issue of the Unicycler approach"
