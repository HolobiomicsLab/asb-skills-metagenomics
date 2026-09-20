---
name: cross-validation-fold-count-comparison
description: Use when evaluating whether combining models from k-fold cross-validation
  splits raises protein contact prediction accuracy over a single model, and when
  comparing accuracy across fold counts (e.g., 3/5/7/10) on CASP-style benchmark sets.
when_to_use_negative:
- You only need a single-model prediction without benchmarking ensemble gains — running
  pipeline.py once suffices
- The model/MSA input is not one of the DeepECA multitask cross-validation models;
  the averaging result is specific to models trained on split datasets of the same
  architecture
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0082
tools:
- name: DeepECA
  role: Multitask deep neural network whose cross-validation-trained models are averaged
    to form the ensemble; run via pipeline.py for prediction
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:87a631cea7e005a251f011f26386cab0023053bd4f01e16202030f979f5c5fa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-validation-fold-count-comparison

## Summary

Reproduce DeepECA's ensemble-averaging protocol: average residue-pair contact probabilities over models trained on cross-validation splits of the training set, and quantify how the number of folds (3-, 5-, 7-, 10-fold) affects top-L/L/2/L/5/L/10 contact accuracy on CASP11 and CASP12 targets.

## When to use

Use when evaluating whether combining models from k-fold cross-validation splits raises protein contact prediction accuracy over a single model, and when comparing accuracy across fold counts (e.g., 3/5/7/10) on CASP-style benchmark sets.

## When NOT to use

- You only need a single-model prediction without benchmarking ensemble gains — running pipeline.py once suffices
- The model/MSA input is not one of the DeepECA multitask cross-validation models; the averaging result is specific to models trained on split datasets of the same architecture

## Inputs

- Pre-trained cross-validation DeepECA models (Google Drive deposit linked in the paper's methods)
- CASP11 target FASTA files (105 domains)
- CASP12 target FASTA files (55 domains)
- Raw per-model contact probability predictions for each target
- Ground-truth contact maps (Cβ/Cα distance < 8 Å) for CASP11/CASP12 targets

## Outputs

- Ensemble-averaged contact probability predictions per target (in ./contact_pred)
- Top-L, L/2, L/5, and L/10 contact accuracy per range category for ensemble vs single models (Tables 8-9)
- Accuracy comparison across 3-, 5-, 7-, and 10-fold ensembles (Table 10)

## How to apply

Download the pre-trained cross-validation models and test datasets from the Google Drive deposit linked in the DeepECA methods, and place CASP11 (105 domains) and CASP12 (55 domains) target FASTA files in ./dataset/fasta. Run the prediction pipeline with the multitask model (python pipeline.py) so raw contact probability outputs land in ./contact_pred. Form the ensemble prediction for each target by simply averaging the predicted residue-pair probabilities across the 5 cross-validation models, then evaluate top-L, L/2, L/5, and L/10 contact accuracy per range category (contact defined as Cβ atoms within 8 Å, Cα for glycine) comparing ensemble vs single models (Tables 8-9). Repeat with the 10-fold ensemble and compare fold counts (Table 10); expect ensemble accuracy to exceed single-model accuracy in almost all categories on both CASP sets (except Medium top L/10 on CASP12), with the 10-fold ensemble highest in almost all categories but the difference among fold counts small.

## Related tools

- **DeepECA** (Multitask deep neural network whose cross-validation-trained models are averaged to form the ensemble; run via pipeline.py for prediction) — https://github.com/tomiilab/DeepECA

## Examples

```
python pipeline.py
```

## Evaluation signals

- Ensemble top-L/L/2/L/5/L/10 accuracy exceeds single-model accuracy in almost all range categories on both CASP11 and CASP12, with the single exception of Medium top L/10 on CASP12
- The 10-fold ensemble shows the highest accuracy in almost all categories, but differences among fold counts are small
- Raw outputs for each of the 5 (or 10) models exist per target before averaging, and the averaged matrix is a valid probability surface per residue pair
- Contact definition matches the paper: Cβ (Cα for glycine) atom distance < 8 Å; accuracy is reported per range category (short/medium/long)
- Number of evaluated targets matches the benchmark sizes: 105 CASP11 domains and 55 CASP12 domains

## Limitations

- The accuracy gain does not hold in every category: Medium top L/10 on CASP12 was not improved by ensemble averaging
- Difference in accuracy among fold counts is small, so fold count is not a strong lever for improvement
- Pre-trained models and datasets must be fetched from the external Google Drive deposit; if unavailable, retraining the networks takes 2-3 days on an NVIDIA TITAN GTX GPU
- Training used 60 layers because of GPU memory limitations, constraining the network depth available to the fold models

## Evidence

- [discussion] Ensemble averaging over five (up to ten) models trained on split datasets: "Our final prediction result for each residue pair was obtained simply by averaging these predicted probabilities"
- [abstract] Ensemble averaging of models raised accuracy: "Moreover, we demonstrated that ensemble averaging of our models can raise accuracy."
- [readme] Prediction run command and output locations: "The results will be stored in ./contact_pred and ./ss_pred directory."
- [methods] Pre-trained model deposit requirement: "Pre-trained models (for prediction) and datasets (for training) are needed."
- [discussion] CASP11/CASP12 benchmark datasets: "we used the CASP11 (105 domains) and CASP12 (55 domains) dataset [33, 34] obtained from the CASP download area"
