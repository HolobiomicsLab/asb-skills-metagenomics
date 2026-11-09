---
name: accuracy-metrics-computation
description: Use when evaluating phenotype classifiers from cross-validation folds
  (e.g., Traitar phypat/phypat+PGL over the 67 GIDEON phenotypes) and reporting macroaccuracy
  alongside overall accuracy.
when_to_use_negative:
- The input is an independent test set rather than cross-validation folds — the article
  phenotyped an independent GIDEON test set separately, and these fold-aggregated
  metrics would not apply
- Predictions come from incomplete genomes or SAGs — the article notes phypat assigns
  more phenotypes to incomplete genomes at the price of more false positives, so fold-level
  accuracy metrics would be misleading without adjusting expectations
- Only overall accuracy is needed and phenotype imbalance matters — macroaccuracy
  over the 67 phenotypes is the headline metric and requires per-phenotype confusion
  counts, not just pooled accuracy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Traitar
  role: Provides the phypat and phypat+PGL classifiers whose cross-validation predictions
    are scored for overall accuracy and macroaccuracy
  repo: https://github.com/hzi-bifo/traitar
- name: Python
  role: Implementation language (Traitar is implemented in Python 2.7) used to aggregate
    fold predictions and compute the accuracy metrics
- name: scipy
  role: Dependency of Traitar supporting the model training/evaluation stack used
    in cross-validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:f14710bc10aa026bce1d6a7e50c80b56e86ef6a59de65008e11779d48ee0622a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - GNU parallel
  - Python
  - Traitar
  - hmmsearch
  - matplotlib
  - pip
  - prodigal
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# accuracy-metrics-computation

## Summary

Compute overall accuracy and macroaccuracy for bacterial phenotype classifiers (Traitar phypat and phypat+PGL) across 67 GIDEON phenotypes during 10-fold nested cross-validation on 234 GIDEON I genomes, so that reproduced performance can be compared directly to the published values.

## When to use

Use when evaluating phenotype classifiers from cross-validation folds (e.g., Traitar phypat/phypat+PGL over the 67 GIDEON phenotypes) and reporting macroaccuracy alongside overall accuracy.

## When NOT to use

- The input is an independent test set rather than cross-validation folds — the article phenotyped an independent GIDEON test set separately, and these fold-aggregated metrics would not apply
- Predictions come from incomplete genomes or SAGs — the article notes phypat assigns more phenotypes to incomplete genomes at the price of more false positives, so fold-level accuracy metrics would be misleading without adjusting expectations
- Only overall accuracy is needed and phenotype imbalance matters — macroaccuracy over the 67 phenotypes is the headline metric and requires per-phenotype confusion counts, not just pooled accuracy

## Inputs

- Per-fold phenotype predictions (binary/trait labels) for the 234 GIDEON I bacterial genomes from the Traitar phypat classifier
- Per-fold phenotype predictions from the Traitar phypat+PGL classifier (phyletic patterns plus ancestral gains/losses)
- Gold-standard phenotype labels for 67 phenotypes from the GIDEON (Global Infectious Disease and Epidemiology Online Network) microbiology section

## Outputs

- Metrics table (cross_validation_accuracy.tsv) with overall accuracy and macroaccuracy per classifier across the 67 phenotypes
- Per-phenotype accuracy values for phypat and phypat+PGL aggregated over the 10 cross-validation folds

## How to apply

Run 10-fold nested cross-validation with both Traitar classifiers: phypat, which uses Pfam phyletic patterns alone, and phypat+PGL, which adds ancestral protein family gains/losses inferred by maximum likelihood on the species phylogeny. Aggregate per-phenotype predictions across the folds for all 234 GIDEON I genomes, then compute overall accuracy (fraction of correct phenotype calls) and macroaccuracy (mean accuracy per phenotype over the 67 phenotypes). Report both metrics per classifier because phypat+PGL improves both individual-phenotype and overall performance relative to phyletic patterns only. Write the metrics to a table (e.g., cross_validation_accuracy.tsv) and judge correctness by agreement with the published values: overall accuracy 88.1% (phypat) and 89.8% (phypat+PGL), macroaccuracy 82.6% (phypat) and 85.5% (phypat+PGL), within a small tolerance.

## Related tools

- **Traitar** (Provides the phypat and phypat+PGL classifiers whose cross-validation predictions are scored for overall accuracy and macroaccuracy) — https://github.com/hzi-bifo/traitar
- **Python** (Implementation language (Traitar is implemented in Python 2.7) used to aggregate fold predictions and compute the accuracy metrics)
- **scipy** (Dependency of Traitar supporting the model training/evaluation stack used in cross-validation)

## Examples

```
traitar phenotype <in_dir> samples.txt from_nucleotides <out_dir> -c 2  # then aggregate phypat and phypat+PGL fold predictions into cross_validation_accuracy.tsv
```

## Evaluation signals

- Reproduced overall accuracy matches the reported 88.1% for phypat and 89.8% for phypat+PGL within a small tolerance
- Reproduced macroaccuracy over the 67 GIDEON phenotypes matches 82.6% for phypat and 85.5% for phypat+PGL within a small tolerance
- phypat+PGL outperforms phypat on both overall accuracy and macroaccuracy, mirroring the article's finding that ancestral gains/losses improve prediction
- Metrics cover all 67 phenotypes and all 10 nested cross-validation folds (234 GIDEON I genomes), not a subset
- Metrics table is written to cross_validation_accuracy.tsv with one row per classifier

## Limitations

- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome, depressing apparent accuracy
- Genotype-phenotype inference with Traitar takes into account only the presence and absence of protein families; other omics data are not integrated, capping achievable accuracy
- The plant biomass predictor prototype is not included in the released Traitar software, so that phenotype cannot be scored

## Evidence

- [discussion] In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy in phenotyping bacterial samples.: "In cross-validation experiments with phenotype data from the GIDEON database, we showed that the Traitar phypat classifier has high accuracy"
- [discussion] improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall.: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [intro] for training phenotype classification models on the protein family annotation of a large number of sequenced genomes of microbial isolates: "for training phenotype classification models on the protein family annotation of a large number of sequenced genomes"
- [discussion] assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes.: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
- [methods] pip install traitar-<version>.tar.gz --user: "pip install traitar-<version>.tar.gz --user"
