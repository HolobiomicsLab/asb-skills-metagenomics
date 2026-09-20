---
name: held-out-test-set-evaluation
description: Use when trained phenotype classifiers must be assessed on genomes that
  were held out from model training, to confirm cross-validation accuracy estimates
  (e.g. Traitar's 85.3%/86.7% macroaccuracy) generalize.
when_to_use_negative:
- The genomes being scored were part of the training data — accuracy would be optimistically
  biased and not comparable to independent-test-set estimates
- Phenotype labels are unavailable for the test genomes, so predictions cannot be
  scored against ground truth
- Input is raw nucleotide sequence needing gene prediction and annotation rather than
  already-computed Pfam profiles — use the full Traitar phenotype workflow instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: Traitar
  role: Runs phenotype prediction with the trained phypat and phypat+PGL classifiers
    on the Pfam profiles of the held-out genomes to produce phenotype calls for scoring
  repo: https://github.com/hzi-bifo/traitar
- name: Python 2.7
  role: Implementation language of Traitar and used to compute macroaccuracy and accuracy
    metrics from the prediction tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:1c0b8303f4ab3ab776590ef537d5f039e142d094f1e38bc6597d586b488cb452
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hzi-bifo/traitar
  tools:
  - Python 2.7
  - Traitar
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# held-out-test-set-evaluation

## Summary

Evaluate trained genotype-phenotype classifiers (Traitar phypat and phypat+PGL) on an independent test set of genomes excluded from model training, computing per-classifier macroaccuracy and overall sample-level accuracy against curated phenotype labels (GIDEON). This validates that reported cross-validation performance generalizes to unseen bacteria.

## When to use

Use when trained phenotype classifiers must be assessed on genomes that were held out from model training, to confirm cross-validation accuracy estimates (e.g. Traitar's 85.3%/86.7% macroaccuracy) generalize.

## When NOT to use

- The genomes being scored were part of the training data — accuracy would be optimistically biased and not comparable to independent-test-set estimates
- Phenotype labels are unavailable for the test genomes, so predictions cannot be scored against ground truth
- Input is raw nucleotide sequence needing gene prediction and annotation rather than already-computed Pfam profiles — use the full Traitar phenotype workflow instead

## Inputs

- Pfam (Pfam 27.0) protein-family profiles of 42 held-out GIDEON II bacterial genomes
- GIDEON phenotype labels for the held-out species (ground truth across 67 phenotypes)
- Trained phypat and phypat+PGL phenotype classification models

## Outputs

- Per-classifier phenotype predictions for the 42 held-out genomes (e.g. predictions tables for phypat and phypat+PGL)
- gideon2_accuracy.tsv with macroaccuracy and sample-level accuracy per classifier (expected: 85.3%/87.5% phypat; 86.7%/87.9% phypat+PGL)

## How to apply

Set aside genomes during training (here 42 GIDEON II species) so their phenotype labels are never seen by the learned models. Run Traitar's phenotype prediction module twice on the Pfam protein-family profiles of the held-out genomes: once with the phypat classifier (phyletic patterns only) and once with the phypat+PGL classifier, which additionally encodes ancestral protein family gains/losses inferred by maximum likelihood on the species phylogeny to account for horizontal gene transfer. Compare each classifier's calls against the GIDEON phenotype labels across the 67 predicted phenotypes, computing macroaccuracy (average over phenotypes) and overall sample-level accuracy per model (e.g. write both to gideon2_accuracy.tsv). Validation: reproduced values should match the reported 85.3%/87.5% (phypat) and 86.7%/87.9% (phypat+PGL) within ~0.1 percentage points for rounding.

## Related tools

- **Traitar** (Runs phenotype prediction with the trained phypat and phypat+PGL classifiers on the Pfam profiles of the held-out genomes to produce phenotype calls for scoring) — https://github.com/hzi-bifo/traitar
- **Python 2.7** (Implementation language of Traitar and used to compute macroaccuracy and accuracy metrics from the prediction tables)

## Examples

```
traitar phenotype <in_dir> <sample_file> from_genes <out_dir> -c 2  # then score predictions vs GIDEON labels for the 42 held-out genomes
```

## Evaluation signals

- Computed macroaccuracy matches reported 85.3% (phypat) and 86.7% (phypat+PGL) within ~0.1 percentage points
- Computed sample-level accuracy matches reported 87.5% (phypat) and 87.9% (phypat+PGL) within ~0.1 percentage points
- gideon2_accuracy.tsv contains one macroaccuracy and one accuracy value per classifier covering all 67 predicted phenotypes
- phypat+PGL macroaccuracy and accuracy are both higher than phypat, consistent with the reported improvement from ancestral gain/loss features
- Hold-out genomes (42 GIDEON II species) are confirmed absent from the training set used to fit the phenotype models

## Limitations

- Traitar genotype-phenotype inference uses only presence/absence of protein families; other omics data are not integrated and integration is proposed only as future work
- For incomplete genomes, absence of a phenotype prediction may simply reflect missing protein families in the input genome
- Accuracy estimates are specific to the GIDEON phenotype labels; Traitar's applicability beyond GIDEON bacteria was assessed separately on Bergey's Manual organisms

## Evidence

- [discussion] reliability of the performance estimates by phenotyping, with similar accuracy, an independent test data set with bacteria described in GIDEON: "reliability of the performance estimates by phenotyping, with similar accuracy, an independent test data set with bacteria described in GIDEON"
- [discussion] improves accuracy over phyletic-pattern-only prediction: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [intro] ancestral gains/losses via maximum-likelihood inference: "The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny"
- [discussion] Traitar predicts 67 phenotypes from Pfam profiles: "assigning 67 different phenotypes to bacteria on the basis of the protein family contents of their genomes."
- [methods] traitar requires Pfam 27.0 HMM models: "traitar requires the Pfam 27.0 HMM models. These are not distributed with this package but need to be downloaded"
