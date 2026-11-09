---
name: horizontal-gene-transfer-aware-classifier-evaluation
description: Use when comparing phenotype classifiers built from Pfam profiles where
  one model adds maximum-likelihood ancestral character-state inference on the species
  phylogeny, and you need held-out-set macroaccuracy/accuracy to quantify the HGT-aware
  benefit.
when_to_use_negative:
- Input is already a phenotype call table or feature matrix and no classifier retraining/comparison
  is needed
- The question concerns cross-validation on training species rather than evaluating
  on an independent test set not used for learning
- Only one classifier's performance is of interest — the skill requires a paired phypat
  vs. phypat+PGL comparison to quantify the HGT-aware gain
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: Traitar
  role: Runs both phenotype classifiers (phypat and phypat+PGL) on Pfam profiles of
    the held-out genomes and produces prediction tables used to compute macroaccuracy
    and accuracy
  repo: https://github.com/hzi-bifo/traitar
- name: Python 2.7
  role: Implementation language for Traitar; used for the accuracy/macroaccuracy computation
    script comparing predictions to GIDEON labels
  repo: https://github.com/hzi-bifo/traitar
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00101-16
    title: 'From Genomes to Phenotypes: Traitar, the Microbial Trait Analyzer'
  source_content_hash: sha256:73292c058827f318dc2cf67ed371b3235663b95aabc0861e94c4b3b92bff7deb
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
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# horizontal-gene-transfer-aware-classifier-evaluation

## Summary

Evaluates whether encoding ancestral protein family gains/losses (to capture horizontal gene transfer) on a species phylogeny improves phenotype classification, by benchmarking the phylogeny-aware phypat+PGL model against the phyletic-pattern-only phypat model on held-out genomes.

## When to use

Use when comparing phenotype classifiers built from Pfam profiles where one model adds maximum-likelihood ancestral character-state inference on the species phylogeny, and you need held-out-set macroaccuracy/accuracy to quantify the HGT-aware benefit.

## When NOT to use

- Input is already a phenotype call table or feature matrix and no classifier retraining/comparison is needed
- The question concerns cross-validation on training species rather than evaluating on an independent test set not used for learning
- Only one classifier's performance is of interest — the skill requires a paired phypat vs. phypat+PGL comparison to quantify the HGT-aware gain

## Inputs

- Pfam 27.0 protein family (Pfam-A.hmm) profiles of bacterial genomes (nucleotide FASTAs or gene FASTAs as input to Traitar)
- GIDEON phenotype labels for training and held-out species across the 67 phenotypes
- Species phylogeny used for maximum-likelihood ancestral protein family gain/loss inference
- List of held-out test-set genomes (e.g., 42 GIDEON II species excluded from model learning)

## Outputs

- Per-classifier phenotype prediction tables (phypat and phypat+PGL subfolders, e.g., predictions_majority-vote_combined.txt)
- Per-model macroaccuracy and accuracy values written per model (e.g., gideon2_accuracy.tsv)
- Heatmaps of phenotype predictions (heatmap_phypat.png, heatmap_phypat_ggl.png, heatmap_comb.png)

## How to apply

Train two phenotype models on the protein family (Pfam 27.0) annotation of genomes with GIDEON-derived phenotype labels: phypat, which uses only phyletic patterns, and phypat+PGL, which additionally encodes ancestral protein family gains/losses inferred by maximum-likelihood on the species phylogeny to account for horizontal gene transfer. Predict the 67 phenotypes for the independent test set (42 held-out GIDEON II species not used in cross-validation) with both classifiers. Compare each classifier's calls against the GIDEON phenotype labels across the 67 predicted phenotypes. Compute macroaccuracy (average over phenotypes) and sample-level accuracy per model — on the GIDEON II test set this yields 85.3%/87.5% for phypat and 86.7%/87.9% for phypat+PGL, confirming that phylogeny-aware modeling improves both individual-phenotype and overall performance.

## Related tools

- **Traitar** (Runs both phenotype classifiers (phypat and phypat+PGL) on Pfam profiles of the held-out genomes and produces prediction tables used to compute macroaccuracy and accuracy) — https://github.com/hzi-bifo/traitar
- **Python 2.7** (Implementation language for Traitar; used for the accuracy/macroaccuracy computation script comparing predictions to GIDEON labels) — https://github.com/hzi-bifo/traitar

## Examples

```
traitar phenotype <in_dir> samples.txt from_genes <out_dir> -c 2
```

## Evaluation signals

- Reproduced macroaccuracy on the 42-species GIDEON II independent test set is 85.3% for phypat and 86.7% for phypat+PGL (tolerance ~0.1 percentage points for rounding)
- Reproduced sample-level accuracy is 87.5% for phypat and 87.9% for phypat+PGL
- phypat+PGL accuracy should exceed phypat accuracy, both per individual phenotype and overall, consistent with the article's finding
- Predictions cover the full set of 67 phenotypes and are compared across the same 42 held-out species for both classifiers
- Output metrics file (e.g., gideon2_accuracy.tsv) contains both metrics (macroaccuracy and accuracy) for both models

## Limitations

- Genotype-phenotype inference with Traitar only takes into account the presence and absence of protein families; other omics data are not integrated
- For incomplete genomes, the absence of a phenotype prediction may be due to the absence of the relevant protein families from the input genomes
- The evaluation depends on GIDEON phenotype labels; accuracy is bounded by the quality of those labels and the 67-phenotype trait space

## Evidence

- [intro] We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events: "We investigated the effect of incorporating ancestral protein family gains and losses into the model inference on classification performance to allow consideration of horizontal gene transfer events"
- [intro] The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny: "The latter were determined by using maximum-likelihood inference of their ancestral character states on the species phylogeny"
- [discussion] reliability of the performance estimates by phenotyping, with similar accuracy, an independent test data set with bacteria described in GIDEON: "reliability of the performance estimates by phenotyping, with similar accuracy, an independent test data set with bacteria described in GIDEON"
- [discussion] improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall.: "improves the accuracy compared to prediction from phyletic patterns only, both for individual phenotypes and overall."
- [readme] A negative prediction is encoded as 0, a prediction made only by the pure phyletic classifier as 1, one made by the phylogeny-aware classifier by 2 and a prediction supported by both algorithms as 3.: [evidence span withheld — anchor status: external_doc]
