---
name: classifier-accuracy-benchmarking
description: Use when evaluating a plasmid/chromosome classifier's accuracy on known-label
  fragment data, or when deciding which probability threshold balances accuracy against
  the fraction of unclassified sequences.
when_to_use_negative:
- Input sequences are shorter than 1000 bp — k-mer coverage is insufficient and results
  are unreliable; filter by length first
- The goal is to recover plasmids from an assembly graph with paired reads — use PlasmidSpades
  or Recycler instead, which use the assembly graph rather than genome signatures
- Input is raw reads rather than assembled metagenomic contigs — PlasFlow is designed
  for assembled contigs
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PlasFlow
  role: Neural-network classifier whose predictions are re-scored at multiple probability
    thresholds to measure accuracy and unclassified fraction
  repo: https://github.com/smaegol/PlasFlow
- name: TensorFlow
  role: Runs the trained deep neural network models used by PlasFlow for classification
- name: scikit-learn
  role: Python dependency supporting model training/evaluation in the PlasFlow pipeline
- name: filter_sequences_by_length.pl
  role: Pre-filters input FASTA by sequence length threshold (e.g., 1000 bp) before
    classification
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:c671e1b6a8611be2c11600e44eb1f6c15bd4039483ad99db263996d85c8c14df
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/smaegol/PlasFlow
  repo_url: https://github.com/smaegol/PlasFlow
  tools:
  - PlasFlow
  - Python
  - TensorFlow
  - filter_sequences_by_length.pl
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# classifier-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a sequence classifier (e.g., PlasFlow) on a held-out validation fragment dataset by re-scoring predictions at multiple probability thresholds and reporting accuracy alongside the unclassified fraction, quantifying the accuracy/coverage trade-off.

## When to use

Use when evaluating a plasmid/chromosome classifier's accuracy on known-label fragment data, or when deciding which probability threshold balances accuracy against the fraction of unclassified sequences.

## When NOT to use

- Input sequences are shorter than 1000 bp — k-mer coverage is insufficient and results are unreliable; filter by length first
- The goal is to recover plasmids from an assembly graph with paired reads — use PlasmidSpades or Recycler instead, which use the assembly graph rather than genome signatures
- Input is raw reads rather than assembled metagenomic contigs — PlasFlow is designed for assembled contigs

## Inputs

- Held-out validation FASTA of known-origin sequences (e.g., 61,221 RefSeq chromosome/plasmid fragments, 1-15.8 kb)
- Trained PlasFlow neural network models (TensorFlow, from github.com/smaegol/PlasFlow_models)
- Probability threshold settings to sweep (e.g., none, 0.7, 0.8)

## Outputs

- PlasFlow predictions TSV (--output) with per-sequence class probabilities
- Validation accuracy report CSV (e.g., validation_accuracy_report.csv) with accuracy and unclassified fraction per threshold
- Binned FASTA files for plasmid, chromosome, and unclassified sequences

## How to apply

Load a held-out validation FASTA of known-origin fragments (RefSeq chromosome and plasmid fragments of 1-15.8 kb, 61,221 sequences) and compute k-mer frequency features in batches with PlasFlow's --batch_size option. Run PlasFlow.py without threshold filtering to get baseline accuracy (~89.52%), then re-score predictions with --threshold 0.7 and --threshold 0.8, treating sequences whose class probability falls below the threshold as unclassified. At threshold 0.7 expect ~94.79% accuracy and at 0.8 expect ~96.41% accuracy with ~25.03% of sequences unclassified. Rationale: probability filtering excludes uncertain predictions, which lowers the number of false hits and increases overall accuracy, but sacrifices coverage. Bin sequences into plasmid, chromosome, and unclassified FASTA files using PlasFlow's output binning for downstream inspection.

## Related tools

- **PlasFlow** (Neural-network classifier whose predictions are re-scored at multiple probability thresholds to measure accuracy and unclassified fraction) — https://github.com/smaegol/PlasFlow
- **TensorFlow** (Runs the trained deep neural network models used by PlasFlow for classification)
- **scikit-learn** (Python dependency supporting model training/evaluation in the PlasFlow pipeline)
- **filter_sequences_by_length.pl** (Pre-filters input FASTA by sequence length threshold (e.g., 1000 bp) before classification) — https://github.com/smaegol/PlasFlow

## Examples

```
PlasFlow.py --input validation_fragments.fasta --output validation.plasflow_predictions.tsv --threshold 0.8
```

## Evaluation signals

- Baseline (no threshold) accuracy on the validation set matches ~89.52% within small numerical tolerance
- Threshold 0.7 accuracy matches ~94.79%; threshold 0.8 accuracy matches ~96.41%
- At threshold 0.8 the unclassified fraction matches ~25.03%
- Sum of classified + unclassified sequence counts equals the input fragment count (61,221)
- Binned plasmid/chromosome/unclassified FASTA files contain only sequences from the input validation FASTA with consistent lengths

## Limitations

- Raising the threshold trades accuracy for coverage — at threshold 0.8 about 25.03% of sequences remain unclassified
- PlasFlow does not perform well on short sequences because proper k-mer coverage is hard to obtain; results for short sequences are unreliable
- PlasFlow is no longer maintained (NOT MAINTAINED per README); models need regular updating with newly published genomic data to overcome dataset limitations
- Requires legacy Python 3.5 and TensorFlow 0.10.0rc0 dependencies

## Evidence

- [results] Raising the probability threshold to 0.8 increased plasmid classification accuracy on the validation dataset to 96.41%, while leaving 25.03% of sequences unclassified.: [evidence span withheld — anchor status: not_found]
- [discussion] We also introduced probability filtering that excluded uncertain predictions; thus, the number of false hits was significantly lower: "We also introduced probability filtering that excluded uncertain predictions; thus, the number of false hits was significantly lower"
- [methods] PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7: "PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7"
