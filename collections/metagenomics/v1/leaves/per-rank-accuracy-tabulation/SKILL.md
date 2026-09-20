---
name: per-rank-accuracy-tabulation
description: Use when a taxonomic classifier's predictions on the 69,335-sequence
  Greengenes2 2024.09 test set need per-rank top-1 accuracy (and optionally F1/ECE)
  tabulated, including Seen/Unseen or similarity-bucket subsets.
when_to_use_negative:
- Input sequences or labels come from a reference database other than Greengenes2
  2024.09 (e.g. SILVA or RDP), where the paper's rank-level reference values do not
  apply
- Predictions and labels do not cover the same sequence set — per-rank accuracy requires
  a matched 69,335-sequence pairing
- The goal is runtime/speed benchmarking rather than accuracy tabulation, which requires
  separate wall-clock timing measurement
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3301
tools:
- name: DeepTaxa
  role: Classifier whose per-rank predictions on the Greengenes2 test set are tabulated;
    `deeptaxa predict` with a taxonomy file emits per-rank accuracy, F1, and ECE
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:2a7abed83b963871b12b7199e4080891eb4064f116142cfdb1213e02d235ef7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - Kraken 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-rank-accuracy-tabulation

## Summary

Compute top-1 classification accuracy at each taxonomic rank (domain through species) by comparing classifier predictions against Greengenes2 2024.09 held-out test-set labels, evaluating only up to the rank preceding the first missing rank in each lineage.

## When to use

Use when a taxonomic classifier's predictions on the 69,335-sequence Greengenes2 2024.09 test set need per-rank top-1 accuracy (and optionally F1/ECE) tabulated, including Seen/Unseen or similarity-bucket subsets.

## When NOT to use

- Input sequences or labels come from a reference database other than Greengenes2 2024.09 (e.g. SILVA or RDP), where the paper's rank-level reference values do not apply
- Predictions and labels do not cover the same sequence set — per-rank accuracy requires a matched 69,335-sequence pairing
- The goal is runtime/speed benchmarking rather than accuracy tabulation, which requires separate wall-clock timing measurement

## Inputs

- gg_2024_09_testing.tsv.gz taxonomy label file for the 69,335-sequence Greengenes2 2024.09 test set
- Classifier predictions (per-sequence lineage strings, e.g. d__;p__;c__;o__;f__;g__;s__) for the same test sequences
- Optional: training-set taxonomy vocabulary to define Seen vs Unseen species labels

## Outputs

- Per-rank top-1 accuracy table (domain, phylum, class, order, family, genus, species)
- Per-rank F1 (and optionally ECE) metrics, e.g. benchmark_species_accuracy.csv or metrics JSON per rank
- Optional Seen/Unseen or similarity-bucket per-rank accuracy breakdown

## How to apply

Load the Greengenes2 2024.09 test-set taxonomy labels (gg_2024_09_testing.tsv.gz) together with the classifier's predictions for the same 69,335 sequences. For each sequence, evaluate top-1 accuracy at each rank from domain to species, but only up to the rank immediately preceding the first missing ('g__'/'s__'-style) rank in the reference lineage, so sequences with unresolvable labels do not count as errors. Optionally partition sequences into Seen (species label present in the training vocabulary) versus Unseen (~1,300 test sequences with labels absent from training) subsets before tabulating. Report per-rank accuracy, F1, and optionally expected calibration error, then compare against published reference values (e.g. DeepTaxa 5-seed mean: 99.98% domain to 92.95% species; Unseen subset declining monotonically from 99.8% at domain to 48.9% at genus).

## Related tools

- **DeepTaxa** (Classifier whose per-rank predictions on the Greengenes2 test set are tabulated; `deeptaxa predict` with a taxonomy file emits per-rank accuracy, F1, and ECE) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_testing.tsv.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/evaluation
```

## Evaluation signals

- Per-rank accuracies for the DeepTaxa full-length v2 checkpoint match the 5-seed means: 99.98% (domain), 99.70% (phylum), 99.63% (class), 99.06% (order), 98.61% (family), 96.87% (genus), 92.95% (species), within a small tolerance
- Species-level accuracy for DeepTaxa reproduces the reported 92.96% (3-seed mean) on the 69,335-sequence test set
- Unseen-subset accuracies decline monotonically from 99.8% at domain to 48.9% at genus, with evaluation restricted to ranks preceding the first missing rank in each lineage
- Cross-seed standard deviation stays ≤0.0004 F1 (≤0.03 percentage points accuracy) at every rank, so recomputation across seeds should stay within that spread
- Every test sequence is scored exactly once per rank up to its lineage's first missing rank; no lineage is silently dropped or double-counted

## Limitations

- Evaluation is on a single stratified 80/20 train/test split with three (or five) training seeds rather than k-fold cross-validation, so split-specific variance is not captured
- Species labels absent from the training vocabulary cannot be correctly predicted; roughly 1,300 Unseen test sequences have degraded genus-level accuracy (48.9%)
- Results are specific to Greengenes2 2024.09; performance on SILVA, RDP, or independent environmental/clinical datasets has not been established

## Evidence

- [results] Evaluate only up to the rank preceding the first missing rank in each lineage: [evidence span withheld — anchor status: not_found]
- [methods] Seen vs Unseen partition rule: "We classified sequences into two categories: “Seen” (species label present in the training vocabulary) and “Unseen” (species label absent from the training vocabulary)."
- [methods] Dataset partition into training and test sets: "The dataset was partitioned into training (277 336 sequences, 80%) and test (69 335 sequences, 20%) sets using stratified sampling to preserve label distributions."
- [readme] Per-rank metrics in evaluation output: [evidence span withheld — anchor status: external_doc]
- [readme] Cross-seed reproducibility of per-rank metrics: [evidence span withheld — anchor status: external_doc]
