---
name: sensitivity-precision-computation
description: Use when scoring gene-finder output (e.g. Glimmer-MG, FragGeneScan, MetaGeneMark
  predictions) on simulated metagenomic reads where true gene coordinates are known
  and per-condition sensitivity/precision must be tabulated by read technology and
  error rate.
when_to_use_negative:
- Real metagenomic reads with no reference annotation — sensitivity cannot be computed
  without ground truth gene coordinates
- Comparing against whole-genome gene finders on complete genomes — Glimmer3 already
  identifies >99% of genes there; fragmented-gene scoring specific to metagenomic
  reads is the relevant regime
- Evaluating phylogenetic classification or clustering quality itself rather than
  gene-prediction accuracy
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Glimmer-MG
  role: Primary gene finder being scored; run in phylogenetic-classification-parameterized
    and cluster-retrained modes on every simulated read set
  repo: github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparator gene finder run on identical read sets under the same error conditions
    for sensitivity/precision comparison
- name: MetaGeneMark
  role: Comparator gene finder run on identical read sets under the same error conditions
    for sensitivity/precision comparison
- name: Glimmer
  role: Underlying framework on which Glimmer-MG is built; whole-genome Glimmer accuracy
    (>99%) is the baseline motivation
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:1fe55f92b30492d4bc4086439e512de841c14580708c7dac19f2cbed80e39bd8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  tools:
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  - MetaGeneMark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sensitivity-precision-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes nucleotide-level (and gene-level) sensitivity and precision of metagenomic gene predictions by aligning predicted coordinates against simulated reference annotations, as used to benchmark Glimmer-MG against FragGeneScan and MetaGeneMark across Sanger, 454, and Illumina reads at 0–2% error rates.

## When to use

Use when scoring gene-finder output (e.g. Glimmer-MG, FragGeneScan, MetaGeneMark predictions) on simulated metagenomic reads where true gene coordinates are known and per-condition sensitivity/precision must be tabulated by read technology and error rate.

## When NOT to use

- Real metagenomic reads with no reference annotation — sensitivity cannot be computed without ground truth gene coordinates
- Comparing against whole-genome gene finders on complete genomes — Glimmer3 already identifies >99% of genes there; fragmented-gene scoring specific to metagenomic reads is the relevant regime
- Evaluating phylogenetic classification or clustering quality itself rather than gene-prediction accuracy

## Inputs

- Simulated metagenomic read sets (Sanger, 454, Illumina) at 0–2% error rates, with 454 simulations including homopolymer-run indel errors
- Ground-truth gene coordinates annotated on each simulated read set
- Predicted gene coordinates from Glimmer-MG (initial and final/retrained modes), FragGeneScan, and MetaGeneMark on the same read sets

## Outputs

- CSV table (e.g. table2_sensitivity_precision.csv) with one row per tool/read-type/error-rate combination of nucleotide-level sensitivity and precision
- Retraining-gain deltas (final minus initial sensitivity/precision) per simulated condition, e.g. retraining_gains.csv
- Indel-prediction sensitivity and precision values for error-prone (454) conditions

## How to apply

Run each gene finder (Glimmer-MG in both its phylogenetic-classification-parameterized mode and its Scimm cluster-retrained mode, plus FragGeneScan and MetaGeneMark) on the identical simulated read sets spanning Sanger, 454, and Illumina libraries at error rates from 0% to 2%. Align both predicted and true gene coordinates against the simulated reference annotations to compute nucleotide-level sensitivity (fraction of reference coding nucleotides recovered) and precision (fraction of predicted coding nucleotides that are correct) for each tool/read-type/error-rate combination. Include 3'-end accuracy for error-prone reads, since 454 homopolymer-run indels shift coding frames, and separately score indel-prediction sensitivity and precision (reported targets: 23.2% and 63.8% at 1.0% error on 454 reads). Compare the cluster-retrained Glimmer-MG predictions against its initial (post-classification) predictions per condition — retraining gains were +0.8% sensitivity without errors and +2.8% at 2.0% error on 454 reads, so deltas should grow with error rate. Assemble one row per tool/read-type/error-rate condition into a table replicating the paper's Table 2 grid.

## Related tools

- **Glimmer-MG** (Primary gene finder being scored; run in phylogenetic-classification-parameterized and cluster-retrained modes on every simulated read set) — github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparator gene finder run on identical read sets under the same error conditions for sensitivity/precision comparison)
- **MetaGeneMark** (Comparator gene finder run on identical read sets under the same error conditions for sensitivity/precision comparison)
- **Glimmer** (Underlying framework on which Glimmer-MG is built; whole-genome Glimmer accuracy (>99%) is the baseline motivation)

## Evaluation signals

- CSV contains exactly one row per tool/read-type/error-rate combination, with sensitivity and precision columns populated
- 454 1.0%-error indel sensitivity/precision entries reproduce the reported 23.2% and 63.8% within tolerance
- Retraining deltas are positive (final > initial sensitivity/precision) across all tested simulated conditions, with larger gains at higher error rates (e.g. +2.8% sensitivity at 2.0% 454 error vs +0.8% without errors)
- Values at the 2.0% 454-error operating point for Glimmer-MG match the reported 73.2% sensitivity / 54.7% precision
- Directional check: Glimmer-MG sensitivity and precision exceed FragGeneScan's at every tested read length and error rate (e.g. +5.8% sensitivity / +6.0% precision on 454 reads at 1.0% error)

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem — precision on 454 reads at 2.0% error is only 54.7%, indicating many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing software, which constrains how many error/read-length conditions can be scored
- Highly diverged organisms where the optimal model parameters are unclear present another challenge, potentially depressing scored accuracy
- Substantial accuracy gains are more likely to come from reducing the error rate via improved assembly and base-calling rather than computation

## Evidence

- [readme] Glimmer-MG can predict insertions and deletions in the sequence by branching into a different frame at low quality base calls such as homopolymer runs in 454 sequences.: [evidence span withheld — anchor status: external_doc]
