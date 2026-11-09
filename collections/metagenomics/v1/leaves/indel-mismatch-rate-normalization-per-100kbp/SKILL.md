---
name: indel-mismatch-rate-normalization-per-100kbp
description: Use when comparing small-plasmid assembly accuracy across tools on simulated
  reads, and you need length-normalized error rates (indels/100 kbp, mismatches/100
  kbp) plus genome fraction from QUAST alignments to a curated ground truth.
when_to_use_negative:
- When evaluating only computational efficiency (runtime, memory) rather than assembly
  sequence accuracy — QUAST error-rate metrics are not the relevant output
- When no curated ground truth plasmid sequences exist, since per-100 kbp error normalization
  requires a trusted reference to align against
- When the benchmarking question involves real (not simulated) read sets where the
  article's QUAST-based Table 4 comparison was not the evaluation framework
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Aligns each plasmid assembly to the Trycycler-derived ground truth and reports
    genome fraction, indels per 100 kbp, and mismatches per 100 kbp
- name: Plassembler
  role: Produces the plasmid assemblies being evaluated (using Flye or Raven for the
    long-read-only first assembly), on both simulated and real reads
  repo: https://github.com/gbouras13/plassembler
- name: Flye >=2.9
  role: Long-read-only assembler invoked by Plassembler for one of the two Plassembler
    benchmarking variants
  repo: https://github.com/fenderglass/Flye
- name: Raven >=1.8
  role: Alternative long-read-only assembler invoked by Plassembler for the second
    benchmarking variant
  repo: https://github.com/lbcb-sci/raven
- name: Unicycler >=0.4.8
  role: Gold-standard hybrid assembler whose plasmid assemblies serve as the comparison
    baseline in the QUAST accuracy table
  repo: https://github.com/rrwick/Unicycler
- name: plassembler_simulation_benchmarking
  role: Snaketool pipeline that generated the benchmarking assemblies and QUAST comparisons
  repo: https://github.com/gbouras13/plassembler_simulation_benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:dc61436beebd39ef34e49c9eeb15c147a9bacb05dc1036efd1344983c7879d4b
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
  - plassembler_simulation_benchmarking
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# indel-mismatch-rate-normalization-per-100kbp

## Summary

Normalizes assembly error rates by computing QUAST-reported indels and mismatches per 100 kbp of aligned sequence, so plasmid assemblies from Plassembler (Flye/Raven) and Unicycler can be compared on the 20 simulated read sets against Trycycler-derived ground truth.

## When to use

Use when comparing small-plasmid assembly accuracy across tools on simulated reads, and you need length-normalized error rates (indels/100 kbp, mismatches/100 kbp) plus genome fraction from QUAST alignments to a curated ground truth.

## When NOT to use

- When evaluating only computational efficiency (runtime, memory) rather than assembly sequence accuracy — QUAST error-rate metrics are not the relevant output
- When no curated ground truth plasmid sequences exist, since per-100 kbp error normalization requires a trusted reference to align against
- When the benchmarking question involves real (not simulated) read sets where the article's QUAST-based Table 4 comparison was not the evaluation framework

## Inputs

- Plasmid assembly FASTA files from Plassembler (Flye and Raven modes) and Unicycler for each of the 20 simulated read sets (from Zenodo 10.5281/zenodo.7996690)
- Trycycler-derived manually curated ground truth plasmid sequences (from github.com/rrwick/Small-plasmid-Nanopore)

## Outputs

- Per-sample QUAST metrics: genome fraction, indels per 100 kbp, mismatches per 100 kbp
- Summary table with one row per program reporting each QUAST metric as median with min/max in brackets (e.g. Plassembler-Flye genome fraction 99.78 mean / 99.97 median)
- Counts of missed plasmids (Plassembler 1 vs Unicycler 7) and fragmented/incomplete plasmids (Flye 3, Raven 5, Unicycler 1)

## How to apply

Download the benchmarking deposit (10.5281/zenodo.7996690) containing Plassembler-Flye, Plassembler-Raven, and Unicycler plasmid assemblies for the 20 simulated read sets plus the Trycycler-derived ground truth plasmid sequences. For each of the 3 programs across all 20 samples, run QUAST to align each plasmid assembly against the ground truth, extracting genome fraction, indels per 100 kbp, and mismatches per 100 kbp. Report each metric as a median with min/max in brackets, and aggregate per-sample QUAST metrics with one row per program. Validate by checking the recomputed table reproduces published figures, e.g. Plassembler-Flye genome fraction mean 99.78 / median 99.97 vs Unicycler 93.81 / 99.88, missed plasmids 1 vs 1 vs 7, and fragmented counts 3/5 vs 1. Note that the small size of plasmids makes per-100 kbp normalization essential so error counts are comparable across plasmids of very different lengths (e.g. the 1934 bp K. variicola INF345 plasmid vs the 30408 bp K. variicola linear plasmid).

## Related tools

- **QUAST** (Aligns each plasmid assembly to the Trycycler-derived ground truth and reports genome fraction, indels per 100 kbp, and mismatches per 100 kbp)
- **Plassembler** (Produces the plasmid assemblies being evaluated (using Flye or Raven for the long-read-only first assembly), on both simulated and real reads) — https://github.com/gbouras13/plassembler
- **Flye >=2.9** (Long-read-only assembler invoked by Plassembler for one of the two Plassembler benchmarking variants) — https://github.com/fenderglass/Flye
- **Raven >=1.8** (Alternative long-read-only assembler invoked by Plassembler for the second benchmarking variant) — https://github.com/lbcb-sci/raven
- **Unicycler >=0.4.8** (Gold-standard hybrid assembler whose plasmid assemblies serve as the comparison baseline in the QUAST accuracy table) — https://github.com/rrwick/Unicycler
- **plassembler_simulation_benchmarking** (Snaketool pipeline that generated the benchmarking assemblies and QUAST comparisons) — https://github.com/gbouras13/plassembler_simulation_benchmarking

## Examples

```
quast.py --min-contig 200 -r ground_truth_plasmids.fasta -o quast_out plassembler_flye_plasmids.fasta unicycler_plasmids.fasta
```

## Evaluation signals

- Recomputed Plassembler-Flye genome fraction matches published values: mean 99.78 / median 99.97 (Unicycler: mean 93.81 / median 99.88)
- Missed-plasmid counts reproduce 1 (Plassembler-Flye) vs 1 (Plassembler-Raven) vs 7 (Unicycler), with the single Plassembler miss being the 2370 bp plasmid in Enterobacter kobei MSB1 1B
- Fragmented/incomplete plasmid counts reproduce 3 (Plassembler-Flye), 5 (Plassembler-Raven), and 1 (Unicycler)
- All QUAST metrics are reported as median with min/max in brackets across the 20 simulated samples, with one summary row per program
- Ground truth used for alignment is the Trycycler-derived, manually curated set, not the raw read-origin sequences

## Limitations

- Plassembler produced more fragmented assemblies than Unicycler (4 with Flye, 5 with Raven vs 1) despite fewer missed plasmids, so error-rate normalization alone understates assembly fragmentation
- The simulated-read benchmarking conclusions do not transfer to Flye determinism issues: Flye --deterministic is not deterministic between thread counts, which can affect reproducibility of assemblies feeding QUAST
- barcode08 Serratia marcescens was excluded from benchmarking because it could not be assembled even with Trycycler, so the metric denominator is 20 curated samples, not the full original set

## Evidence

- [methods] QUAST for each of the 3 programs across the 20 simulated samples, aligning each plasmid assembly against the Trycycler-derived ground truth to extract genome fraction, indels per 100 kbp, and mismatches per 100 kbp (reported as median with min/max in brackets).: [evidence span withheld — anchor status: not_found]
- [methods] `plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp).: "`plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7"
- [results] On the simulated read sets, Plassembler missed fewer plasmids than Unicycler (1 vs 7) but produced more fragmented assemblies (4 with Flye, 5 with Raven vs 1 for Unicycler), with otherwise comparable accuracy.: [evidence span withheld — anchor status: not_found]
- [abstract] the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
- [methods] independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md): "independent Trycyler derived and manually curated ground truth you can find"
