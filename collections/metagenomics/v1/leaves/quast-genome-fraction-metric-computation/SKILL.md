---
name: quast-genome-fraction-metric-computation
description: Use when benchmarking plasmid (or bacterial) assemblies against a curated
  ground truth and you need QUAST accuracy metrics — genome fraction, mismatches/indels
  per 100 kbp — plus counts of missed and fragmented plasmids.
when_to_use_negative:
- Input is raw reads rather than assembled plasmid contigs — QUAST accuracy comparison
  requires FASTA assemblies and a ground truth reference
- There is no curated ground truth available; genome fraction and missed-plasmid counts
  are undefined without a reference inventory
- The goal is runtime/efficiency benchmarking of assemblers rather than assembly accuracy
  evaluation
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: aligns each plasmid assembly to the Trycycler-derived ground truth and computes
    genome fraction, indels per 100 kbp, and mismatches per 100 kbp
- name: Plassembler
  role: produces one of the benchmarked plasmid assemblies (with Flye or Raven as
    the long-read assembler)
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: gold-standard comparator whose plasmid assemblies are evaluated with the same
    QUAST metrics
  repo: https://github.com/rrwick/Unicycler
- name: Flye
  role: long-read assembler used by Plassembler (>=2.9) in the Plassembler-Flye benchmark
    arm
  repo: https://github.com/fenderglass/Flye
- name: Raven
  role: alternative long-read assembler used by Plassembler (>=1.8) in the Plassembler-Raven
    benchmark arm
  repo: https://github.com/lbcb-sci/raven
- name: plassembler_simulation_benchmarking
  role: Snaketool pipeline that generated the assemblies and benchmark results being
    evaluated
  repo: https://github.com/gbouras13/plassembler_simulation_benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:f7d866d2120b3e8da9cb8444064c76f380eac7c05810906d98c1bf21cadb351f
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
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# quast-genome-fraction-metric-computation

## Summary

Reproduce the QUAST-based accuracy evaluation of plasmid assemblies from the Plassembler paper by aligning each program's plasmid FASTA (Plassembler-Flye, Plassembler-Raven, Unicycler) against the Trycycler-derived ground truth across 20 simulated read sets, extracting genome fraction, indels per 100 kbp, and mismatches per 100 kbp.

## When to use

Use when benchmarking plasmid (or bacterial) assemblies against a curated ground truth and you need QUAST accuracy metrics — genome fraction, mismatches/indels per 100 kbp — plus counts of missed and fragmented plasmids.

## When NOT to use

- Input is raw reads rather than assembled plasmid contigs — QUAST accuracy comparison requires FASTA assemblies and a ground truth reference
- There is no curated ground truth available; genome fraction and missed-plasmid counts are undefined without a reference inventory
- The goal is runtime/efficiency benchmarking of assemblers rather than assembly accuracy evaluation

## Inputs

- Plasmid assembly FASTA files for Plassembler-Flye, Plassembler-Raven, and Unicycler across 20 simulated read sets (from Zenodo 10.5281/zenodo.7996690)
- Trycycler-derived, manually curated ground truth plasmid sequences (FASTA) from rrwick/Small-plasmid-Nanopore
- Benchmarking pipeline from github.com/gbouras13/plassembler_simulation_benchmarking

## Outputs

- Per-sample QUAST reports with genome fraction, indels per 100 kbp, and mismatches per 100 kbp
- Counts of missed plasmids and fragmented/incomplete plasmids per program
- Summary table with one row per program (median with min/max in brackets, plus means) matching Table 4

## How to apply

Download the benchmarking deposit (10.5281/zenodo.7996690) containing the Plassembler-Flye, Plassembler-Raven, and Unicycler plasmid assemblies for the 20 simulated read sets plus the Trycycler-derived ground truth plasmid sequences. Run QUAST for each of the 3 programs across all 20 samples, aligning each plasmid contig set against the ground truth to extract genome fraction, indels per 100 kbp, and mismatches per 100 kbp, reporting each as median with min/max in brackets. Separately compare each assembled contig set to the ground truth plasmid inventory to count missed plasmids (expected 1 vs 1 vs 7) and incomplete/fragmented plasmids (expected 3 for Plassembler-Flye, 5 for Plassembler-Raven, 1 for Unicycler). Aggregate per-sample QUAST metrics into a summary table with one row per program; validate by checking reproduction of published figures such as Plassembler-Flye genome fraction mean 99.78 / median 99.97 versus Unicycler 93.81 mean / 99.88 median.

## Related tools

- **QUAST** (aligns each plasmid assembly to the Trycycler-derived ground truth and computes genome fraction, indels per 100 kbp, and mismatches per 100 kbp)
- **Plassembler** (produces one of the benchmarked plasmid assemblies (with Flye or Raven as the long-read assembler)) — https://github.com/gbouras13/plassembler
- **Unicycler** (gold-standard comparator whose plasmid assemblies are evaluated with the same QUAST metrics) — https://github.com/rrwick/Unicycler
- **Flye** (long-read assembler used by Plassembler (>=2.9) in the Plassembler-Flye benchmark arm) — https://github.com/fenderglass/Flye
- **Raven** (alternative long-read assembler used by Plassembler (>=1.8) in the Plassembler-Raven benchmark arm) — https://github.com/lbcb-sci/raven
- **plassembler_simulation_benchmarking** (Snaketool pipeline that generated the assemblies and benchmark results being evaluated) — https://github.com/gbouras13/plassembler_simulation_benchmarking

## Examples

```
quast.py plassembler_flye_sample01_plasmids.fasta -r trycycler_ground_truth_plasmids.fasta -o quast_plassembler_flye_sample01 --min-contig 500
```

## Evaluation signals

- Recomputed Plassembler-Flye genome fraction mean ≈ 99.78 and median ≈ 99.97; Unicycler mean ≈ 93.81 and median ≈ 99.88
- Missed plasmid counts reproduce 1 (Plassembler-Flye) vs 1 (Plassembler-Raven) vs 7 (Unicycler), with the missed Plassembler plasmid being the 2370 bp plasmid in Enterobacter kobei MSB1 1B
- Fragmented/incomplete plasmid counts reproduce 3 (Plassembler-Flye), 5 (Plassembler-Raven), and 1 (Unicycler)
- All 3 programs evaluated over exactly 20 simulated read sets, each QUAST run aligned to the Trycycler-derived ground truth
- Metrics reported as median with min/max in brackets and match the published summary table structure (one row per program)

## Limitations

- The old v1.2.0 benchmarking used different read sets (simulated R9 vs R10), so cross-version comparisons are not truly fair
- Barcode08 Serratia marcescens was excluded from benchmarking due to sample heterogeneity that resisted even Trycycler assembly
- Fragmented-assembly counts differ between the task card (4 with Flye) and the paper workflow (3 for Plassembler-Flye) — verify against the published table when reproducing

## Evidence

- [methods] Run QUAST for each of the 3 programs across the 20 simulated samples, aligning each plasmid assembly against the Trycycler-derived ground truth: [evidence span withheld — anchor status: not_found]
- [methods] genome fraction metrics reported as median with min/max: [evidence span withheld — anchor status: not_found]
- [results] Plassembler-Flye genome fraction 99.78 mean / 99.97 median: [evidence span withheld — anchor status: not_found]
- [methods] plassembler missed only 1 plasmid while Unicycler missed 7: "`plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp)."
- [abstract] benchmarking deposit location: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
