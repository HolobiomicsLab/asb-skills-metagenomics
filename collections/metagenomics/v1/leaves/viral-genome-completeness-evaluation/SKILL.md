---
name: viral-genome-completeness-evaluation
description: Use when viral scaffolds have been binned into vMAGs (e.g., by vRhyme)
  and you need to classify bins as low- vs. high-redundancy and estimate completeness/contamination
  per bin.
when_to_use_negative:
- Input is an entire unfiltered metagenome or microbial scaffolds — vRhyme is built
  for viral sequences/scaffolds and 'is not meant to bin microbes'
- Binning non-viral or multiple inactive (non-replicating) prophages from the same
  host genome, where vRhyme has documented difficulties
- The question is raw binning coverage rather than quality — completeness/redundancy
  evaluation presupposes bins already exist
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: vRhyme
  role: Produces the binned vMAGs, protein redundancy scores, and vRhyme_bin_scoring.tsv
    used to classify bins by contamination
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Mmseqs2
  role: Protein redundancy scoring engine underlying vRhyme's bin refinement and contamination
    proxy
  repo: https://github.com/soedinglab/MMseqs2
- name: CheckV
  role: Estimates completeness and contamination of the resulting vMAGs to confirm
    near-complete genomes (e.g., the Herelleviridae vMAG)
- name: Prodigal
  role: Predicts ORFs on scaffolds ('-m -p meta') providing the protein inputs for
    redundancy checks
  repo: https://github.com/hyattpd/Prodigal
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:17db7b0ece47cbd207aa03117a7090614f254d9190a9dabdf7bd03745b56119b
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# viral-genome-completeness-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the quality of viral metagenome-assembled genomes (vMAGs) produced by vRhyme by combining vRhyme's Mmseqs2-based protein redundancy scoring with CheckV completeness/contamination estimation, exploiting the observation that viruses seldom encode redundant genes.

## When to use

Use when viral scaffolds have been binned into vMAGs (e.g., by vRhyme) and you need to classify bins as low- vs. high-redundancy and estimate completeness/contamination per bin.

## When NOT to use

- Input is an entire unfiltered metagenome or microbial scaffolds — vRhyme is built for viral sequences/scaffolds and 'is not meant to bin microbes'
- Binning non-viral or multiple inactive (non-replicating) prophages from the same host genome, where vRhyme has documented difficulties
- The question is raw binning coverage rather than quality — completeness/redundancy evaluation presupposes bins already exist

## Inputs

- vRhyme best-iteration output files: vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, vRhyme_bin_scoring.tsv
- Binned vMAG FASTA files from vRhyme
- Predicted protein files (Prodigal '-m -p meta' format) used for Mmseqs2 redundancy checks
- Reference viral genomes/completeness expectations for CheckV-based validation

## Outputs

- Per-bin protein redundancy counts and low-contamination/low-redundancy classification (e.g., 11,909 low-contamination bins on GOV2; 762 low-redundancy vMAGs of 849 total on human skin)
- CheckV completeness and contamination estimates per vMAG
- Benchmark summary table (e.g., gov2_vrhyme_benchmark_summary.csv) of redundant-protein totals and completeness distributions across binners

## How to apply

After binning, read the best-iteration vRhyme outputs (vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, vRhyme_bin_scoring.tsv) and use the Mmseqs2-based protein redundancy metric to classify bins: 0-1 redundant proteins indicates low contamination, while >6 suggests a contaminated bin (allowing ~10 for NCLDVs). Then run CheckV on the binned vMAGs to quantify completeness and contamination, confirming candidate near-complete genomes — e.g., in the human skin cohort the 22-scaffold Herelleviridae vMAG should reach ~100% completeness. Rationale: vRhyme's bin refinement deliberately exploits that few viruses encode redundant proteins, so protein redundancy is a virus-appropriate proxy for contamination where universal single-copy marker genes are absent. Compile per-bin redundancy, completeness, and contamination values into a summary table (e.g., gov2_vrhyme_benchmark_summary.csv) for cross-tool comparison.

## Related tools

- **vRhyme** (Produces the binned vMAGs, protein redundancy scores, and vRhyme_bin_scoring.tsv used to classify bins by contamination) — https://github.com/AnantharamanLab/vRhyme
- **Mmseqs2** (Protein redundancy scoring engine underlying vRhyme's bin refinement and contamination proxy) — https://github.com/soedinglab/MMseqs2
- **CheckV** (Estimates completeness and contamination of the resulting vMAGs to confirm near-complete genomes (e.g., the Herelleviridae vMAG))
- **Prodigal** (Predicts ORFs on scaffolds ('-m -p meta') providing the protein inputs for redundancy checks) — https://github.com/hyattpd/Prodigal

## Evaluation signals

- On GOV2, reproduced vRhyme statistics match Supplementary Table S7: 15,106 redundant proteins and 11,909 low-contamination bins from 56,642 scaffolds in 13,175 bins
- On human skin, the run yields 849 vMAGs total with 762 having low protein redundancy (0-1 redundant proteins), covering 2,794 of 10,601 binned scaffolds
- The 22-scaffold Herelleviridae vMAG reaches ~100% completeness by CheckV
- vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, and vRhyme_bin_scoring.tsv all exist in the output hierarchy and parse correctly
- Bins with >6 redundant proteins are flagged as likely contaminated (with ~10 tolerated for NCLDVs) in the classification step

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme is not meant to bin microbes and performance on a whole metagenome as input has not been fully evaluated
- When applied to a synthetic dataset of predominately non-viral sequences, MetaBat2 performed better than vRhyme, so completeness evaluation via viral-specific redundancy does not transfer to non-viral bins

## Evidence
