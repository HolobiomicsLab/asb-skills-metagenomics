---
name: gene-prediction-accuracy-evaluation
description: Use when comparing gene caller output (e.g., Glimmer-MG predictions)
  to known gene coordinates from GenBank reference genomes or simulated metagenomes,
  to quantify sensitivity and precision per tool, read length, or error rate.
when_to_use_negative:
- Input is a real (non-simulated) metagenome without reference annotations — no ground
  truth exists to compute sensitivity/precision against.
- The goal is to run gene prediction itself rather than evaluate prediction quality
  (use the Glimmer-MG prediction pipeline, not this evaluation skill).
- Predictions need only ab-initio gene finding on complete genomes where Glimmer already
  identifies >99% of genes — evaluation adds overhead without discriminating between
  tools.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Metagenomic gene predictor whose sensitivity and precision (3'/5', nucleotide-
    and gene-level) are evaluated against simulated ground truth and reference annotations
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Baseline whole-genome gene caller; iterated Glimmer3.0 and length/adjacency-filtered
    Glimmer3.1 predictions are scored against the 12 reference prokaryotic genomes
- name: FragGeneScan
  role: Comparator gene predictor run on the same error-free read sets to benchmark
    sensitivity and precision against Glimmer-MG
- name: MetaGeneAnnotator
  role: Comparator gene predictor scored on the same simulated metagenome read sets
- name: MetaGeneMark
  role: Comparator gene predictor scored on the same simulated metagenome read sets
- name: Scimm
  role: Clustering step whose retraining gain on initial Glimmer-MG predictions is
    quantified by per-condition sensitivity/precision deltas
- name: Phymm
  role: Phylogenetic classifier providing ICM parameterization for Glimmer-MG predictions
    under evaluation
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:702b7a328600fd2ac82271e73780afd2cc2e94a2e64976cb81a70f89294fe76b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  - Illumina
  - MetaGeneAnnotator
  - MetaGeneMark
  - Phymm
  - Scimm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-prediction-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates gene prediction accuracy (sensitivity and precision, including 3'/5' start-site accuracy) of tools like Glimmer-MG, MetaGeneAnnotator, MetaGeneMark, and FragGeneScan against curated reference annotations, on whole genomes and simulated metagenomic read sets across read lengths and error rates.

## When to use

Use when comparing gene caller output (e.g., Glimmer-MG predictions) to known gene coordinates from GenBank reference genomes or simulated metagenomes, to quantify sensitivity and precision per tool, read length, or error rate.

## When NOT to use

- Input is a real (non-simulated) metagenome without reference annotations — no ground truth exists to compute sensitivity/precision against.
- The goal is to run gene prediction itself rather than evaluate prediction quality (use the Glimmer-MG prediction pipeline, not this evaluation skill).
- Predictions need only ab-initio gene finding on complete genomes where Glimmer already identifies >99% of genes — evaluation adds overhead without discriminating between tools.

## Inputs

- FASTA whole-genome sequences with curated gene annotations (truth set, e.g., 12 reference prokaryotic genomes)
- Simulated metagenomic read sets (Sanger, 454, Illumina) with per-base quality values and known gene coordinates from source GenBank genomes
- Gene prediction outputs (predicted gene coordinates and start positions) from Glimmer-MG, Glimmer3.0/3.1, Glimmer-MG, MetaGeneAnnotator, MetaGeneMark, FragGeneScan

## Outputs

- Per-genome and averaged sensitivity and precision CSV (e.g., supplementary_table_s1_accuracy.csv, table1_sensitivity_precision.csv)
- Per-condition retraining deltas (final minus initial) in sensitivity/precision (retraining_gains.csv)
- Per-configuration error-model ablation sensitivity/precision (error_model_ablation.csv)

## How to apply

Collect a truth set of FASTA genomes with curated gene annotations (or simulate reads from GenBank genomes, e.g., 454 reads via FlowSim with a Pareto abundance distribution and homopolymer-run error model). Run each predictor (Glimmer-MG in classification+clustering and Precise modes, plus comparators) on the same input and compare predicted genes and start positions against reference annotations, computing 3'/5' sensitivity and precision per genome or condition by counting exact and near-correct start codon calls. Compute deltas (final minus initial) when evaluating pipeline stages such as Glimmer-MG's Scimm clustering-based retraining step. Validate against paper-reported operating points, e.g., 73.2% sensitivity and 54.7% precision on simulated 454 reads at a 2.0% error rate, and accept tables only if all values fall in [0, 100] and deltas reproduce reported directions.

## Related tools

- **Glimmer-MG** (Metagenomic gene predictor whose sensitivity and precision (3'/5', nucleotide- and gene-level) are evaluated against simulated ground truth and reference annotations) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Baseline whole-genome gene caller; iterated Glimmer3.0 and length/adjacency-filtered Glimmer3.1 predictions are scored against the 12 reference prokaryotic genomes)
- **FragGeneScan** (Comparator gene predictor run on the same error-free read sets to benchmark sensitivity and precision against Glimmer-MG)
- **MetaGeneAnnotator** (Comparator gene predictor scored on the same simulated metagenome read sets)
- **MetaGeneMark** (Comparator gene predictor scored on the same simulated metagenome read sets)
- **Scimm** (Clustering step whose retraining gain on initial Glimmer-MG predictions is quantified by per-condition sensitivity/precision deltas)
- **Phymm** (Phylogenetic classifier providing ICM parameterization for Glimmer-MG predictions under evaluation)

## Examples

```
glimmer-mg --precise simulated_454_reads.fasta; python compare_predictions.py --pred glimmer-mg.predict --truth reference.gbk --metrics sensitivity,precision --out table1_sensitivity_precision.csv
```

## Evaluation signals

- CSV contains sensitivity and precision fields for every predictor with values in [0, 100]
- Glimmer-MG values match paper-reported operating points: 73.2% sensitivity and 54.7% precision on simulated 454 reads at a 2.0% error rate
- Error-model ablation shows sensitivity on 454 reads at 2.0% error rising from ~41.9% (no indel modeling) to ~73.2% (indel modeling enabled)
- Retraining deltas (final minus initial) are positive across all tested simulated conditions, consistent with +0.8% sensitivity without errors and +2.8% at 2.0% error rate on 454 reads
- Glimmer3.1 reproducibly shows 3' precision improved ~1.3% over Glimmer3.0 with 3' sensitivity maintained across the 12 reference genomes

## Limitations

- Predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem; precision on 454 reads at 2.0% error is only 54.7%, indicating many genes are predicted incorrectly
- Glimmer-MG is more computationally expensive than existing software
- Highly diverged organisms where the optimal model parameters are unclear present a challenge; substantial gains likely require lower error rates via improved assembly and base-calling rather than computation

## Evidence
