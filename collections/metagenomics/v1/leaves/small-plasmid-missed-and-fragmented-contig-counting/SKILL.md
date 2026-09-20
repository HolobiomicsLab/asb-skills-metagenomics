---
name: small-plasmid-missed-and-fragmented-contig-counting
description: Use when benchmarking plasmid assembly tools on simulated (or real) read
  sets and you must quantify missed plasmids, fragmented/incomplete plasmid assemblies,
  and QUAST accuracy metrics against a curated ground truth.
when_to_use_negative:
- You lack a curated ground truth plasmid inventory, so missed/fragmented counts cannot
  be defined
- Your goal is novel de novo plasmid assembly rather than benchmarking existing assemblies
- You want chromosomal assembly evaluation — this skill scores plasmid contigs only
  against plasmid ground truth
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Aligns each plasmid assembly against the ground truth to extract genome fraction,
    indels per 100 kbp, and mismatches per 100 kbp
- name: Plassembler
  role: Plasmid assembler under evaluation, run with Flye or Raven long-read assemblers
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard comparison assembler whose missed/fragmented plasmid counts
    are benchmarked
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: Long-read-only assembler used by Plassembler to create the initial assembly
    (>=2.9)
  repo: https://github.com/mikolmogorov/Flye
- name: Raven
  role: Optional long-read-only assembler used by Plassembler (>=1.8)
  repo: https://github.com/lbcb-sci/raven
- name: plassembler_simulation_benchmarking
  role: Snaketool pipeline that generated the simulated-read benchmarking assemblies
    being counted
  repo: https://github.com/gbouras13/plassembler_simulation_benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:e8c8c20d79b904a5ed0ef08c29c4fe94e61aa13ab35f93abaa63e867d0bb9369
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbouras13/plassembler
  tools:
  - Flye
  - Plassembler
  - QUAST
  - Raven
  - Unicycler
  - plassembler_simulation_benchmarking
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# small-plasmid-missed-and-fragmented-contig-counting

## Summary

A benchmarking evaluation skill that counts missed and fragmented/incomplete plasmids by comparing assembler outputs (Plassembler-Flye, Plassembler-Raven, Unicycler) against a Trycycler-derived ground truth, complemented by QUAST accuracy metrics such as genome fraction, indels per 100 kbp, and mismatches per 100 kbp.

## When to use

Use when benchmarking plasmid assembly tools on simulated (or real) read sets and you must quantify missed plasmids, fragmented/incomplete plasmid assemblies, and QUAST accuracy metrics against a curated ground truth.

## When NOT to use

- You lack a curated ground truth plasmid inventory, so missed/fragmented counts cannot be defined
- Your goal is novel de novo plasmid assembly rather than benchmarking existing assemblies
- You want chromosomal assembly evaluation — this skill scores plasmid contigs only against plasmid ground truth

## Inputs

- Plasmid assembly FASTA files for each program (Plassembler-Flye, Plassembler-Raven, Unicycler) across the 20 simulated read sets
- Trycycler-derived, manually curated ground truth plasmid FASTA sequences (from rrwick/Small-plasmid-Nanopore)
- Benchmarking deposit 10.5281/zenodo.7996690 containing input FASTQ and output assembly files

## Outputs

- Per-sample QUAST metrics: genome fraction, indels per 100 kbp, mismatches per 100 kbp (median with min/max in brackets)
- Counts of missed plasmids per program (Plassembler 1 vs Unicycler 7)
- Counts of incomplete/fragmented plasmids per program (3 Plassembler-Flye, 5 Plassembler-Raven, 1 Unicycler)
- Summary table with one row per program aggregating QUAST metrics and counts

## How to apply

Download the benchmarking deposit (10.5281/zenodo.7996690) containing Plassembler-Flye, Plassembler-Raven, and Unicycler plasmid assemblies for the 20 simulated read sets plus ground truth plasmid sequences. For each program across the 20 samples, run QUAST aligning each plasmid assembly against the Trycycler-derived ground truth to extract genome fraction, indels per 100 kbp, and mismatches per 100 kbp (reported as median with min/max in brackets). Compare each assembled contig set to the ground truth plasmid inventory to count missed plasmids (expected 1 for both Plassembler variants vs 7 for Unicycler) and incomplete/fragmented plasmids (expected 3 for Plassembler-Flye, 5 for Plassembler-Raven, 1 for Unicycler). Aggregate per-sample QUAST metrics into a one-row-per-program summary table and validate against published figures, e.g. Plassembler-Flye genome fraction mean 99.78 / median 99.97 vs Unicycler 93.81 / 99.88.

## Related tools

- **QUAST** (Aligns each plasmid assembly against the ground truth to extract genome fraction, indels per 100 kbp, and mismatches per 100 kbp)
- **Plassembler** (Plasmid assembler under evaluation, run with Flye or Raven long-read assemblers) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard comparison assembler whose missed/fragmented plasmid counts are benchmarked) — https://github.com/rrwick/Unicycler
- **Flye** (Long-read-only assembler used by Plassembler to create the initial assembly (>=2.9)) — https://github.com/mikolmogorov/Flye
- **Raven** (Optional long-read-only assembler used by Plassembler (>=1.8)) — https://github.com/lbcb-sci/raven
- **plassembler_simulation_benchmarking** (Snaketool pipeline that generated the simulated-read benchmarking assemblies being counted) — https://github.com/gbouras13/plassembler_simulation_benchmarking

## Examples

```
quast.py -r trycycler_ground_truth_plasmids.fasta -o quast_out plassembler_flye.fasta plassembler_raven.fasta unicycler.fasta
```

## Evaluation signals

- Missed plasmid counts reproduce 1 for both Plassembler variants vs 7 for Unicycler
- Fragmented/incomplete plasmid counts reproduce 3 (Plassembler-Flye), 5 (Plassembler-Raven), and 1 (Unicycler)
- Genome fraction mean/median matches published values, e.g. 99.78/99.97 for Plassembler-Flye and 93.81/99.88 for Unicycler
- The missed plasmid for Plassembler is the 2370 bp plasmid in Enterobacter kobei MSB1 1B
- Summary table contains exactly one row per program with median and min/max brackets for each QUAST metric

## Limitations

- Barcode08 Serratia marcescens was excluded from benchmarking because even Trycycler could not resolve it, so counts do not cover this sample
- Old v1.2.0 benchmarking used different read sets (simulated R9 vs R10), so cross-version comparisons are not truly fair
- The only misassembly observed was the linear plasmid in K. variicola (30408 bp recovered vs 31780 bp ground truth), a known issue of the Unicycler approach that fragment counts alone do not capture

## Evidence

- [methods] In simulated benchmarking, plassembler missed 1 plasmid while Unicycler missed 7: "`plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp)."
- [abstract] Zenodo deposit holds benchmarking inputs and outputs: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
- [methods] Trycycler-derived ground truth source: "independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md)"
- [readme] Benchmarking pipeline repository: [evidence span withheld — anchor status: external_doc]
- [methods] Sample excluded from benchmarking: "barcode08 Serratia Marcescens was excluded from benchmarking due to how devilishly difficult it was for Ryan Wick to assemble it, even with Trycycler"
