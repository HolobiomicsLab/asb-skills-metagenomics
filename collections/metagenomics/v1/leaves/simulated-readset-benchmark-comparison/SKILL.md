---
name: simulated-readset-benchmark-comparison
description: 'Use when benchmarking plasmid assembler accuracy on simulated read sets:
  QUAST genome fraction, indels/mismatches per 100 kbp, missed-plasmid counts and
  fragmented-assembly counts vs a curated ground truth.'
when_to_use_negative:
- The input is a real (non-simulated) read benchmark — the paper benchmarks real reads
  separately, and the old v1.2.0 comparison used different read sets (simulated R9
  vs R10) so is 'not truly a fair comparison'.
- No Trycycler-derived ground truth exists for the samples — missed-plasmid and fragmentation
  counts cannot be computed without it.
- You need QUAST metrics for the changelog/v1.3.0 benchmark on badread nanopore2023
  reads — the author notes 'I was too lazy to run QUAST' for those results.
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Plassembler
  role: Tool under benchmark; runs with Flye or Raven to assemble plasmids from simulated
    hybrid read sets
  repo: https://github.com/gbouras13/plassembler
- name: Unicycler
  role: Gold-standard comparator assembler on the same 20 simulated read sets
  repo: https://github.com/rrwick/Unicycler
- name: Flye >=2.9
  role: Long-read-only assembler used inside Plassembler for one benchmark arm
  repo: https://github.com/fenderglass/Flye
- name: Raven >=1.8
  role: Alternative long-read-only assembler used inside Plassembler for the second
    benchmark arm
  repo: https://github.com/lbcb-sci/raven
- name: QUAST
  role: 'Evaluation: aligns each plasmid assembly against ground truth to extract
    genome fraction and error rates'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:dea1880534e4d80edd9799763bb94e1363ba7609a19de10ed4271b2d3e1e49ae
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# simulated-readset-benchmark-comparison

## Summary

Reproduces the Plassembler paper's simulated-read benchmark (20 read sets) comparing Plassembler-Flye, Plassembler-Raven and Unicycler using QUAST accuracy metrics plus missed-plasmid and fragmented-assembly counts against Trycycler-derived ground truth.

## When to use

Use when benchmarking plasmid assembler accuracy on simulated read sets: QUAST genome fraction, indels/mismatches per 100 kbp, missed-plasmid counts and fragmented-assembly counts vs a curated ground truth.

## When NOT to use

- The input is a real (non-simulated) read benchmark — the paper benchmarks real reads separately, and the old v1.2.0 comparison used different read sets (simulated R9 vs R10) so is 'not truly a fair comparison'.
- No Trycycler-derived ground truth exists for the samples — missed-plasmid and fragmentation counts cannot be computed without it.
- You need QUAST metrics for the changelog/v1.3.0 benchmark on badread nanopore2023 reads — the author notes 'I was too lazy to run QUAST' for those results.

## Inputs

- Plassembler-Flye, Plassembler-Raven and Unicycler plasmid assemblies (FASTA) for 20 simulated read sets (Zenodo 10.5281/zenodo.7996690)
- Trycycler-derived manually curated ground truth plasmid sequences (FASTA, from rrwick/Small-plasmid-Nanopore)
- Simulated hybrid read FASTQ sets (long + short reads)

## Outputs

- Per-sample QUAST metrics table: genome fraction, indels per 100 kbp, mismatches per 100 kbp (median with min/max in brackets)
- Missed-plasmid counts per program (Plassembler-Flye 1, Plassembler-Raven 1, Unicycler 7)
- Incomplete/fragmented plasmid counts per program (Plassembler-Flye 3, Plassembler-Raven 5, Unicycler 1)
- One-row-per-program summary table reproducing Table 4

## How to apply

Download the benchmarking deposit from Zenodo (10.5281/zenodo.7996690), which holds Plassembler-Flye, Plassembler-Raven and Unicycler plasmid assemblies for 20 simulated read sets plus Trycycler-derived ground truth plasmid sequences. Run QUAST per program per sample, aligning each plasmid assembly against the ground truth to extract genome fraction, indels per 100 kbp and mismatches per 100 kbp. Then compare each contig set to the ground truth plasmid inventory to count missed plasmids (expected 1 for both Plassembler variants vs 7 for Unicycler) and incomplete/fragmented plasmids (expected 3 for Plassembler-Flye, 5 for Plassembler-Raven, 1 for Unicycler). Aggregate into a one-row-per-program summary table reporting median with min/max in brackets (e.g. Plassembler-Flye genome fraction 99.78 mean / 99.97 median, Unicycler 93.81 mean / 99.88 median). The full Snaketool-based pipeline is available in gbouras13/plassembler_simulation_benchmarking and works 'out of the box' for the simulated reads.

## Related tools

- **Plassembler** (Tool under benchmark; runs with Flye or Raven to assemble plasmids from simulated hybrid read sets) — https://github.com/gbouras13/plassembler
- **Unicycler** (Gold-standard comparator assembler on the same 20 simulated read sets) — https://github.com/rrwick/Unicycler
- **Flye >=2.9** (Long-read-only assembler used inside Plassembler for one benchmark arm) — https://github.com/fenderglass/Flye
- **Raven >=1.8** (Alternative long-read-only assembler used inside Plassembler for the second benchmark arm) — https://github.com/lbcb-sci/raven
- **QUAST** (Evaluation: aligns each plasmid assembly against ground truth to extract genome fraction and error rates)

## Examples

```
quast.py -r trycycler_ground_truth_plasmids.fasta -o quast_sim sample01_plassembler_flye.fasta sample01_plassembler_raven.fasta sample01_unicycler.fasta
```

## Evaluation signals

- Recomputed genome fraction mean equals 99.78 for Plassembler-Flye (median 99.97) and 93.81 mean / 99.88 median for Unicycler
- Missed-plasmid counts match 1 vs 1 vs 7 (Plassembler-Flye vs Plassembler-Raven vs Unicycler); the single Plassembler miss is the 2370 bp plasmid in Enterobacter kobei MSB1 1B
- Fragmented/incomplete plasmid counts match 3 (Plassembler-Flye), 5 (Plassembler-Raven) and 1 (Unicycler) across the 20 samples
- All 3 programs ran on all 20 simulated samples (60 QUAST alignments) with each assembly aligned against the Trycycler-derived ground truth
- Summary table reports median with min/max in brackets, matching the paper's reporting format

## Limitations

- Plassembler produces more fragmented assemblies than Unicycler on simulated reads (4 with Flye, 5 with Raven vs 1), despite missing fewer plasmids
- Flye --deterministic is not deterministic between thread counts, so reruns may differ slightly
- Cross-version comparisons with different read sets (R9 vs R10 simulation) are not fair comparisons
- QUAST evaluation was not run for the v1.3.0 badread nanopore2023 changelog benchmark

## Evidence

- [methods] plassembler missed only 1 plasmid (2370 bp plasmid in Enterobacter kobei MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp).: "`plassembler` missed only 1 plasmid (2370 bp plasmid in _Enterobacter kobei_ MSB1 1B) while Unicycler missed 7, of which were small (under 10kbp)."
- [abstract] the benchmarking input FASTQ and output files can be found at Zenodo 7996690: "the benchmarking input FASTQ and output files can be found at https://doi.org/10.5281/zenodo.7996690"
- [methods] independent Trycyler derived and manually curated ground truth available in Small-plasmid-Nanopore repo: "independent Trycyler derived and manually curated ground truth you can find [here](https://github.com/rrwick/Small-plasmid-Nanopore/blob/main/method.md)"
- [methods] the results themselves are at zenodo.org/record/7996690: "the results themselves [here](https://zenodo.org/record/7996690)."
- [results] the new approach seems to provide a speed-up vs v1.2.0 for more complicated assemblies: "the new approach seems to provide a speed-up vs v1.2.0  for more complicated assemblies (e.g. _Klebsiella variicola_ INF345 )"
