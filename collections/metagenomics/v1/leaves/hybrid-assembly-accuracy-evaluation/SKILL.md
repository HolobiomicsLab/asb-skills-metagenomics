---
name: hybrid-assembly-accuracy-evaluation
description: Use when comparing plasmid recovery accuracy of assemblers (e.g. Plassembler
  vs Unicycler) on real or subsampled Nanopore/Illumina read sets, where a manually
  curated ground truth exists and QUAST-based accuracy metrics are needed.
when_to_use_negative:
- No independent, manually curated ground truth assembly exists for the isolate, so
  QUAST genome-fraction and misassembly checks cannot be trusted
- You are comparing read-level or runtime performance only rather than assembly accuracy
  against a reference (QUAST evaluation is the core of this skill)
- The input is metagenomic or eukaryotic data, outside the bacterial isolate scope
  of Plassembler/Unicycler benchmarking
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Plassembler
  role: Hybrid plasmid assembler under evaluation; recovers plasmids missed by Unicycler
  repo: https://github.com/gbouras13/plassembler
- name: Flye >=2.9
  role: Long-read-only assembler used by Plassembler for the initial long-read assembly
    (nano-hq mode)
  repo: https://github.com/fenderglass/Flye
- name: Raven >=1.8
  role: Alternative long-read-only assembler used by Plassembler in benchmarking
  repo: https://github.com/lbcb-sci/raven
- name: Unicycler >=0.4.8
  role: Gold-standard hybrid assembler used as the comparison baseline and inside
    Plassembler for final hybrid assembly of extracted reads
  repo: https://github.com/rrwick/Unicycler
- name: QUAST
  role: 'Reference-based evaluation of assemblies: genome fraction, indels, mismatches
    per 100 kbp'
- name: minimap2 >=2.11
  role: Maps long and short reads to non-chromosome contigs and unmapped reads for
    read extraction prior to assembly
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:28f6216d2d268f00b6ee2d573c0f247085685af5ba6cd5d11c80acd53fc3f4b0
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hybrid-assembly-accuracy-evaluation

## Summary

Evaluate plasmid assemblies produced by hybrid and long-read-first assemblers (Plassembler with Flye or Raven, Unicycler) against Trycycler-derived manually curated ground truth using QUAST metrics such as genome fraction, indels and mismatches per 100 kbp, and counts of missed, fragmented and misassembled plasmids.

## When to use

Use when comparing plasmid recovery accuracy of assemblers (e.g. Plassembler vs Unicycler) on real or subsampled Nanopore/Illumina read sets, where a manually curated ground truth exists and QUAST-based accuracy metrics are needed.

## When NOT to use

- No independent, manually curated ground truth assembly exists for the isolate, so QUAST genome-fraction and misassembly checks cannot be trusted
- You are comparing read-level or runtime performance only rather than assembly accuracy against a reference (QUAST evaluation is the core of this skill)
- The input is metagenomic or eukaryotic data, outside the bacterial isolate scope of Plassembler/Unicycler benchmarking

## Inputs

- Pooled 60x subsampled Nanopore FASTQ read sets (plus matching Illumina FASTQs) for the six Wick et al. 2021 isolates
- Trycycler-derived manually curated ground truth plasmid sequences (FASTA) from rrwick/Small-plasmid-Nanopore
- Plasmid assemblies (FASTA) from Plassembler (Flye and Raven modes) and Unicycler

## Outputs

- Per-assembly QUAST accuracy metrics: genome fraction (mean/median/min/max), indels and mismatches per 100 kbp (median, min, max)
- Comparison table real_reads_assembly_accuracy_table.tsv with additional plasmids recovered, total missed plasmids, fragmented plasmids and misassemblies per program
- List of specific recovered plasmids (e.g. 1934 bp K. variicola INF345, 10697 bp K. oxytoca MSB1 2C)

## How to apply

Obtain the pooled 60x subsampled real-read FASTQs for the Wick et al. 2021 isolates (e.g. Acinetobacter baumannii J9, Klebsiella variicola INF345, K. oxytoca MSB1 2C) and the Trycycler-derived manually curated ground truth plasmids from the Small-plasmid-Nanopore deposit. Assemble plasmids with Plassembler using Flye (>=2.9, nano-hq mode) and optionally Raven (>=1.8), and run Unicycler (>=0.4.8) on the same read sets with matched thread counts (8 threads) to keep runtime comparisons fair. Evaluate every plasmid assembly against the ground truth with QUAST, computing genome fraction (mean/median/min/max) plus indels and mismatches per 100 kbp (median, min, max). Tabulate additional plasmids recovered, total missed plasmids, fragmented plasmids and misassemblies per program, and write the comparison to real_reads_assembly_accuracy_table.tsv. Validate by checking that Plassembler and Unicycler share identical genome fractions (mean 97.02, median 100) with zero misassemblies, and that Plassembler additionally recovers the 1934 bp K. variicola INF345 and 10697 bp K. oxytoca MSB1 2C plasmids Unicycler misses.

## Related tools

- **Plassembler** (Hybrid plasmid assembler under evaluation; recovers plasmids missed by Unicycler) — https://github.com/gbouras13/plassembler
- **Flye >=2.9** (Long-read-only assembler used by Plassembler for the initial long-read assembly (nano-hq mode)) — https://github.com/fenderglass/Flye
- **Raven >=1.8** (Alternative long-read-only assembler used by Plassembler in benchmarking) — https://github.com/lbcb-sci/raven
- **Unicycler >=0.4.8** (Gold-standard hybrid assembler used as the comparison baseline and inside Plassembler for final hybrid assembly of extracted reads) — https://github.com/rrwick/Unicycler
- **QUAST** (Reference-based evaluation of assemblies: genome fraction, indels, mismatches per 100 kbp)
- **minimap2 >=2.11** (Maps long and short reads to non-chromosome contigs and unmapped reads for read extraction prior to assembly) — https://github.com/lh3/minimap2

## Examples

```
conda install -c bioconda plassembler
```

## Evaluation signals

- Plassembler and Unicycler genome fractions are identical (mean 97.02, median 100) with zero misassemblies on the six real read sets
- Indel and mismatch rates per 100 kbp are low (median values reported alongside min/max) for all evaluated assemblers
- Plassembler recovers 2 additional small plasmids Unicycler misses (1934 bp K. variicola INF345; 10697 bp K. oxytoca MSB1 2C), giving 2 additional/2 missed vs 0 additional/2 missed for Unicycler
- real_reads_assembly_accuracy_table.tsv contains rows for all six isolates and all three assembler configurations with complete metric columns
- All programs were run under matched conditions (8 threads, same benchmark machine) so runtime and accuracy comparisons are directly comparable

## Limitations

- Flye is not deterministic across thread counts even with --deterministic, so reruns may differ slightly
- The only misassembly observed was of the linear plasmid in K. variicola (30408 bp recovered vs 31780 bp ground truth), a known limitation of the Unycyler-based approach for linear plasmids
- barcode08 Serratia marcescens was excluded from benchmarking because even Trycycler-based manual curation was too difficult, so heterogeneous samples are out of scope
- Results depend on the specific subsampled (60x) read sets; simulated read benchmarks gave different plasmid-miss counts (Plassembler 1, Unicycler 7)

## Evidence

- [methods] plassembler with Flye and Raven recovered 2 additional small plasmids missed by Unicycler: "`plassembler` recovered 2 additional small plasmids missed by Unicycler of lengths 1934 bp  _(_K. variicola_ INF345) and 10697 bp (_K. oxytoca_ MSB1 2C)."
- [methods] Long-read assembly basis for Plassembler evaluation: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [methods] Read extraction feeding the hybrid assembly: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [methods] Hybrid assembly step with Unicycler: "These reads are then assembled using Unicycler (Wick et al 2017)."
- [methods] Ground truth provenance for accuracy evaluation: "independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md)"
- [readme] Unicycler hybrid assembly role (README): [evidence span withheld — anchor status: external_doc]
