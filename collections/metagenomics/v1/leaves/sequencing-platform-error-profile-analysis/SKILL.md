---
name: sequencing-platform-error-profile-analysis
description: Use when gene-predicting short metagenomic reads from error-prone platforms
  (454/Sanger with indels, Illumina with substitutions), especially when truncated
  ORFs and frame shifts from homopolymer-run artifacts must be resolved.
when_to_use_negative:
- Input consists of assembled, high-quality contigs where indel/substitution error
  rates are negligible and frame tracking is unnecessary
- The goal is platform-independent classification or abundance estimation rather than
  gene prediction on error-prone reads
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Glimmer-MG
  role: Performs ORF-based gene prediction with branching into alternate reading frames
    at low-quality base calls (indel modeling) and merging of adjacent ORFs separated
    by substitution-introduced stop codons
  repo: github.com/davek44/Glimmer-MG
- name: 454 Life Sciences
  role: Source of simulated reads carrying homopolymer-run indel errors used to train/validate
    the indel error model
- name: Illumina
  role: Source of simulated reads carrying substitution errors used to validate stop-codon
    substitution modeling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:180bd8767e5cb24523f10b026cf8df8e5299f499dac18b0ce54e126666ede922
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - Glimmer-MG
  - Illumina
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sequencing-platform-error-profile-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Model platform-specific sequencing errors (454 homopolymer-run indels and Illumina stop-codon-introducing substitutions) so that metagenomic gene prediction can track coding frames across error-prone raw reads, as implemented in Glimmer-MG.

## When to use

Use when gene-predicting short metagenomic reads from error-prone platforms (454/Sanger with indels, Illumina with substitutions), especially when truncated ORFs and frame shifts from homopolymer-run artifacts must be resolved.

## When NOT to use

- Input consists of assembled, high-quality contigs where indel/substitution error rates are negligible and frame tracking is unnecessary
- The goal is platform-independent classification or abundance estimation rather than gene prediction on error-prone reads

## Inputs

- Metagenomic shotgun read sets (FASTA/FASTQ) from 454, Sanger, or Illumina sequencing
- Per-base quality values for candidate indel positions
- Simulated metagenomes with ground-truth gene annotations for validation

## Outputs

- Gene predictions with predicted insertion/deletion positions and alternate-frame branch points
- Merged ORF predictions where Illumina substitution errors were resolved
- Per-configuration sensitivity/precision metrics (e.g. error_model_ablation.csv)

## How to apply

Load simulated or real read sets carrying per-base quality values: 454/Sanger reads with indel errors concentrated in homopolymer runs and Illumina reads with substitution errors that introduce spurious stop codons. For indel modeling, traverse each predicted ORF, flag low-quality base calls as indel candidates, and branch the ORF into three reading frames (original plus two alternates), capped at two indels per ORF and penalized by base-quality log-likelihoods. For substitution modeling on Illumina reads, merge adjacent predicted ORFs whose apparent stop codons are attributable to sequencing substitutions rather than true gene ends. Validate by re-running with indel modeling disabled as an ablation baseline and computing gene-prediction sensitivity and precision against simulated ground truth.

## Related tools

- **Glimmer-MG** (Performs ORF-based gene prediction with branching into alternate reading frames at low-quality base calls (indel modeling) and merging of adjacent ORFs separated by substitution-introduced stop codons) — github.com/davek44/Glimmer-MG
- **454 Life Sciences** (Source of simulated reads carrying homopolymer-run indel errors used to train/validate the indel error model)
- **Illumina** (Source of simulated reads carrying substitution errors used to validate stop-codon substitution modeling)

## Evaluation signals

- On simulated 454 reads with 2.0% error rate, sensitivity rises from the no-indel-modeling baseline (~41.9%) to ~73.2% with indel modeling enabled
- Precision at the same setting should be ~54.7%, matching the paper's reported deltas
- No ORF branches into more than two alternate frames; at most two indels are predicted per ORF
- Indel positions identified by the model match simulated ground-truth indel locations more accurately than FragGeneScan
- Ablation run with indel modeling disabled reproduces the lower sensitivity baseline

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem; precision is only 54.7% at 2.0% 454 error rate
- Glimmer-MG is more computationally expensive than existing software
- Substantial further gains are more likely to come from reducing the error rate via improvements to assembly and base-calling rather than computation

## Evidence

- [readme] README indel branching: [evidence span withheld — anchor status: external_doc]
