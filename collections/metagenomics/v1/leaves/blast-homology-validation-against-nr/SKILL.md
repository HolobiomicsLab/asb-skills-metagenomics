---
name: blast-homology-validation-against-nr
description: Use when raw 454 metagenomic reads (e.g. human gut microbiome samples
  TS28/TS50) have gene predictions from multiple callers and an independent, database-grounded
  estimate of prediction correctness is needed for tool comparison.
when_to_use_negative:
- Evaluating predictors on simulated metagenomes where true gene coordinates are known
  — use nucleotide-level sensitivity/precision against the known truth instead of
  BLAST validation
- Input is assembled genomes or complete prokaryotic genomes — the validation problem
  for short, error-prone metagenomic reads (truncated genes, homopolymer-run indels)
  does not apply in the same way
- The analysis goal is functional annotation rather than gene-predictor benchmarking;
  BLAST hits would then be an end product, not a validation signal
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Glimmer-MG
  role: Primary gene predictor being validated; produces predicted coding regions
    on 454 reads using Phymm classification, Scimm clustering, and indel error modeling
  repo: github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparison gene predictor; its predictions are BLAST-validated alongside Glimmer-MG's
- name: MetaGeneMark
  role: Comparison gene predictor; its predictions are BLAST-validated alongside Glimmer-MG's
- name: 454 Life Sciences
  role: Sequencing platform producing the raw reads (with homopolymer-run indels)
    that the gene predictions are generated from
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:46b9093037a785b4dcbeea5708617aafba8ad602962581bbfff4de468ceaa2cb
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
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  - MetaGeneMark
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# blast-homology-validation-against-nr

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates metagenomic gene predictions by translating predicted coding regions and BLAST-searching the translated proteins against the NCBI non-redundant protein database, counting predictions with hits (E < 0.001) as an independent accuracy proxy for Glimmer-MG, FragGeneScan, and MetaGeneMark on real 454 human gut microbiome reads.

## When to use

Use when raw 454 metagenomic reads (e.g. human gut microbiome samples TS28/TS50) have gene predictions from multiple callers and an independent, database-grounded estimate of prediction correctness is needed for tool comparison.

## When NOT to use

- Evaluating predictors on simulated metagenomes where true gene coordinates are known — use nucleotide-level sensitivity/precision against the known truth instead of BLAST validation
- Input is assembled genomes or complete prokaryotic genomes — the validation problem for short, error-prone metagenomic reads (truncated genes, homopolymer-run indels) does not apply in the same way
- The analysis goal is functional annotation rather than gene-predictor benchmarking; BLAST hits would then be an end product, not a validation signal

## Inputs

- Raw 454 metagenomic sequencing reads (human gut microbiome samples TS28 and TS50 from the obese/lean twin study)
- Gene prediction files (predicted coding regions) from Glimmer-MG, FragGeneScan, and MetaGeneMark
- NCBI non-redundant (nr) protein database

## Outputs

- Per-predictor, per-sample counts of predicted genes
- Counts and proportions of predictions with BLAST hits (E < 0.001) against nr
- Comparison table (matching Table 3 of the paper) ranking predictors by BLAST-validated predictions

## How to apply

Run each gene predictor (Glimmer-MG pipeline with Phymm classification, Scimm clustering, cluster retraining, and homopolymer-run indel error modeling; FragGeneScan; MetaGeneMark) on the same raw reads, then translate the predicted coding regions. BLAST the translated predictions against the NCBI non-redundant protein database using an E-value cutoff of 0.001. Per predictor and per sample (TS28, TS50), tabulate the total number of predicted genes and the counts and proportions of predictions with BLAST hits. Because the ground truth is unknown for real reads, the proportion of predictions matching database proteins serves as the validation metric: the correct outcome is the predictor with the most and highest-proportion aligning predictions, which for this paper was Glimmer-MG (4.5% more BLAST-aligning predictions than FragGeneScan and 6.5% more than MetaGeneMark).

## Related tools

- **Glimmer-MG** (Primary gene predictor being validated; produces predicted coding regions on 454 reads using Phymm classification, Scimm clustering, and indel error modeling) — github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparison gene predictor; its predictions are BLAST-validated alongside Glimmer-MG's)
- **MetaGeneMark** (Comparison gene predictor; its predictions are BLAST-validated alongside Glimmer-MG's)
- **454 Life Sciences** (Sequencing platform producing the raw reads (with homopolymer-run indels) that the gene predictions are generated from)

## Evaluation signals

- Per-sample prediction counts and BLAST-hit proportions reproduce Table 3 of the Kelley et al. NAR paper (qualitative agreement check)
- Glimmer-MG yields more BLAST-aligning predictions than FragGeneScan by ~4.5% and than MetaGeneMark by ~6.5% on the real TS28/TS50 reads
- Glimmer-MG shows the highest proportion of predictions matching nr proteins among the three predictors
- All BLAST searches consistently use the E < 0.001 cutoff against the same nr database version across predictors
- Total predicted genes per predictor per sample are tabulated before computing hit proportions, so proportions are computed over comparable denominators

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels (e.g. 454 homopolymer runs), remains a difficult problem; on simulated 454 reads with 2.0% error, Glimmer-MG precision is only 54.7%
- Glimmer-MG is more computationally expensive than existing gene prediction software
- Substantial accuracy gains are more likely to come from reducing the error rate via improvements to assembly of metagenomic reads rather than from gene-prediction computation
- No software versions for Glimmer-MG or the compared tools are stated, so exact reproducibility of Table 3 counts may vary with versions and nr database currency

## Evidence

- [results] On the real human gut microbiome 454 reads, Glimmer-MG made the most BLAST-aligning gene predictions, 4.5% more than FragGeneScan and 6.5% more than MetaGeneMark: [evidence span withheld — anchor status: not_found]
- [abstract] Glimmer-MG makes the most sensitive and precise predictions on simulated and real metagenomes for all read lengths and error rates tested: "Glimmer-MG makes the most sensitive and precise predictions on simulated and real metagenomes for all read lengths and error rates tested"
- [readme] Readme on indel handling in 454 homopolymer runs: [evidence span withheld — anchor status: external_doc]
