---
name: protein-redundancy-contamination-assessment
description: 'Use when scoring vRhyme bins for contamination: after running vRhyme
  on viral scaffolds, interpret vRhyme_bin_scoring.tsv redundant-protein counts per
  bin and select the iteration, then classify low- vs high-contamination vMAGs.'
when_to_use_negative:
- Input is a whole (non-viral) metagenome or microbial genomes — vRhyme is not meant
  to bin microbes and redundant-gene logic does not transfer
- Assessing bins from bacterial/archaeal MAG tools where redundant genes are normal,
  making the virus-centric redundancy assumption invalid
- Binning multiple inactive (non-replicating) prophages from the same host genome,
  where vRhyme has known difficulty separating genomes
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: vRhyme
  role: Runs the bin scoring and protein redundancy refinement across binning iterations
    and emits vRhyme_bin_scoring.tsv
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mmseqs2
  role: Performs the protein homology searches underlying vRhyme's redundancy checks
  repo: https://github.com/soedinglab/MMseqs2
- name: Prodigal
  role: Predicts open reading frames ('-m -p meta') whose redundancy is scored
  repo: https://github.com/hyattpd/Prodigal
- name: CheckV
  role: Independent completeness/contamination assessment of the selected vMAGs
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:17bda30fb13cd6c9c99f52736999c782acb57984d6c14ef21806f00aca9948de
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - CheckV
  - CoCoNet
  - MetaBat2
  - Mmseqs2
  - Prodigal
  - Python
  - Samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# Protein redundancy contamination assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess contamination of viral metagenome-assembled genomes (vMAGs) produced by vRhyme using its protein redundancy scoring mechanism, which exploits the observation that viruses seldom encode redundant proteins, complemented by CheckV completeness/contamination estimates.

## When to use

Use when scoring vRhyme bins for contamination: after running vRhyme on viral scaffolds, interpret vRhyme_bin_scoring.tsv redundant-protein counts per bin and select the best iteration, then classify low- vs high-contamination vMAGs.

## When NOT to use

- Input is a whole (non-viral) metagenome or microbial genomes — vRhyme is not meant to bin microbes and redundant-gene logic does not transfer
- Assessing bins from bacterial/archaeal MAG tools where redundant genes are normal, making the virus-centric redundancy assumption invalid
- Binning multiple inactive (non-replicating) prophages from the same host genome, where vRhyme has known difficulty separating genomes

## Inputs

- vRhyme output files: vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, vRhyme_bin_scoring.tsv
- Viral scaffold nucleotide FASTA (e.g., GOV2 scaffolds pre-filtered to ≥5 kb and 98% dereplicated)
- Prodigal-predicted protein files in 'name_#' format (-p) and gene files (-g), or scaffolds for ORF prediction
- Sorted BAM read-alignment files for per-sample coverage

## Outputs

- Per-bin redundant-protein counts used to flag low-contamination (0-1) vs likely contaminated (>6) bins
- Selected best-iteration vMAG membership and summary tables
- CheckV completeness/contamination estimates per vMAG (e.g., benchmark summary like gov2_vrhyme_benchmark_summary.csv)

## How to apply

Run vRhyme with Prodigal ORF prediction ('-m -p meta') so that predicted proteins can be scored across alternate binning iterations with an Mmseqs2-based homology search; vRhyme writes per-bin protein redundancy counts in vRhyme_bin_scoring.tsv and the best iteration in vRhyme_best_bins.#.summary.tsv. Classify bins with 0-1 redundant proteins as low-redundancy (likely low contamination), filter bins with more than 6 redundant proteins as likely contaminated, and allow roughly 10 redundant proteins for NCLDV bins, which legitimately encode duplicate genes. For the GOV2 benchmark this logic yielded 15,106 redundant proteins and 11,909 low-contamination bins out of 13,175 bins, compared with 29,334 (MetaBat2) and 71,364 (CoCoNet) redundant proteins, supporting vRhyme's precision. Confirm completeness/contamination independently with CheckV on the selected bins.

## Related tools

- **vRhyme** (Runs the bin scoring and protein redundancy refinement across binning iterations and emits vRhyme_bin_scoring.tsv) — https://github.com/AnantharamanLab/vRhyme
- **Mmseqs2** (Performs the protein homology searches underlying vRhyme's redundancy checks) — https://github.com/soedinglab/MMseqs2
- **Prodigal** (Predicts open reading frames ('-m -p meta') whose redundancy is scored) — https://github.com/hyattpd/Prodigal
- **CheckV** (Independent completeness/contamination assessment of the selected vMAGs)

## Examples

```
prodigal -i my.metagenome.fna -o my.genes -a my.proteins.faa -p meta
```

## Evaluation signals

- vRhyme_bin_scoring.tsv exists and redundant-protein totals for the GOV2 run match Supplementary Table S7 (15,106 redundant proteins; 11,909 low-contamination bins of 13,175)
- Best-iteration files (vRhyme_best_bins.#.membership.tsv and vRhyme_best_bins.#.summary.tsv) are present in the vRhyme output hierarchy
- Every retained low-contamination vMAG has 0-1 redundant proteins; bins with >6 redundant proteins are excluded (up to ~10 tolerated for NCLDV)
- Human skin cohort run yields 849 total vMAGs with 762 low-redundancy vMAGs covering 2,794 scaffolds
- CheckV reports the 22-scaffold Herelleviridae vMAG at ~100% completeness with low contamination

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme is built for viral sequences; performance on whole metagenomes has not been fully evaluated and it is not meant to bin microbes
- NCLDV bins legitimately encode redundant genes, so a fixed redundancy cutoff would misclassify them as contaminated
- When applied to a synthetic dataset of predominately non-viral sequences, MetaBat2 performed better than vRhyme

## Evidence

- [readme] Provirion ORFs: [evidence span withheld — anchor status: external_doc]
