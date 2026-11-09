---
name: casp-target-benchmark-comparison
description: Use when a trained contact or secondary-structure predictor must be compared
  against existing methods on fixed CASP11/CASP12 domain sets, with precision at top
  L/10-L and accuracy tables reproduced.
when_to_use_negative:
- Input proteins are not CASP target domains but a custom benchmark; the fixed CASP11/CASP12
  domain sets are required for comparability with published tables.
- The goal is training a new model rather than evaluating one — use the training workflow
  (multitask.py) instead.
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: DeepECA
  role: Trained multitask model whose contact, secondary structure, and ASA predictions
    are benchmarked on CASP11/CASP12 via pipeline.py
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs for each CASP target (three iterations, E-value 0.001,
    60% coverage, UniProt20_2016) used as input for all evaluated methods
- name: RaptorX-Property
  role: Baseline secondary structure and ASA predictor compared against on CASP11
    and CASP12 (Tables 4-6)
- name: SCRATCH-1D
  role: Baseline secondary structure and ASA predictor compared against on CASP11
    and CASP12 (Tables 4-6)
- name: PSICOV
  role: Existing ECA baseline for top-L/k contact precision comparison
- name: CCMpred
  role: Existing ECA baseline for top-L/k contact precision comparison
- name: DeepCov
  role: Covariance-based DNN baseline for contact precision comparison
- name: ResPRE
  role: Precision-matrix-based DNN baseline for contact precision comparison
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:b75943c51d739464f588582b107c4e376617a42921ef059d5c4c8919da00fe7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CCMpred
  - DeepCov
  - DeepECA
  - HHblits
  - PSICOV
  - RaptorX-Property
  - ResPRE
  - SCRATCH-1D
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# casp-target-benchmark-comparison

## Summary

Benchmark protein contact, secondary structure, and ASA prediction methods against published baselines (e.g. RaptorX-Property, SCRATCH-1D, PSICOV, CCMpred, DeepCov, ResPRE) on CASP11 (105 domains) and CASP12 (55 domains) targets, as done in the DeepECA study.

## When to use

Use when a trained contact or secondary-structure predictor must be compared against existing methods on fixed CASP11/CASP12 domain sets, with precision at top L/10-L and accuracy tables reproduced.

## When NOT to use

- Input proteins are not CASP target domains but a custom benchmark; the fixed CASP11/CASP12 domain sets are required for comparability with published tables.
- The goal is training a new model rather than evaluating one — use the training workflow (multitask.py) instead.

## Inputs

- CASP11 target domains (105 domains) from the CASP download area
- CASP12 target domains (55 domains) from the CASP download area
- MSAs generated with HHblits (three iterations, E-value 0.001, coverage 60%)
- FASTA files of target sequences placed in ./dataset/fasta
- Pre-trained models and datasets from the DeepECA Google Drive folder

## Outputs

- Contact prediction files in ./contact_pred
- Secondary structure and ASA prediction files in ./ss_pred
- Precision tables at top L/10, L/5, L/2, L per sequence-separation range (cf. Tables 11-12)
- Secondary-structure and ASA accuracy tables vs RaptorX-Property and SCRATCH-1D (cf. Tables 4-6)

## How to apply

Obtain the CASP11 and CASP12 target domain sets (105 and 55 domains) from the CASP download area and generate MSAs for each target with HHblits (three iterations, E-value 0.001, 60% coverage, UniProt20_2016 library). Run the trained model via pipeline.py so predictions land in ./contact_pred and ./ss_pred, then evaluate contact prediction precision TP/(TP+FP) at top L/10, L/5, L/2, and L predicted pairs separately for short, medium, and long sequence-separation ranges. For secondary structure and ASA, compute prediction accuracy on the same CASP11/CASP12 domains and compare against RaptorX-Property and SCRATCH-1D (Tables 4-6); the method is validated only if it outperforms these baselines. Note that RaptorX comparisons were computed on their web server, with 3 of 105 CASP11 and 1 of 55 CASP12 domains missing due to server errors.

## Related tools

- **DeepECA** (Trained multitask model whose contact, secondary structure, and ASA predictions are benchmarked on CASP11/CASP12 via pipeline.py) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs for each CASP target (three iterations, E-value 0.001, 60% coverage, UniProt20_2016) used as input for all evaluated methods)
- **RaptorX-Property** (Baseline secondary structure and ASA predictor compared against on CASP11 and CASP12 (Tables 4-6))
- **SCRATCH-1D** (Baseline secondary structure and ASA predictor compared against on CASP11 and CASP12 (Tables 4-6))
- **PSICOV** (Existing ECA baseline for top-L/k contact precision comparison)
- **CCMpred** (Existing ECA baseline for top-L/k contact precision comparison)
- **DeepCov** (Covariance-based DNN baseline for contact precision comparison)
- **ResPRE** (Precision-matrix-based DNN baseline for contact precision comparison)

## Examples

```
python pipeline.py  # predictions written to ./contact_pred and ./ss_pred for CASP benchmark evaluation
```

## Evaluation signals

- Contact precision values TP/(TP+FP) at top L/10, L/5, L/2, and L match the Baseline rows of Tables 11 and 12 on CASP11 (105 domains) and CASP12 (55 domains).
- The evaluated model outperforms PSICOV, CCMpred, DeepCov, and ResPRE at every distance range and prediction count on both CASP sets.
- Secondary-structure accuracy exceeds RaptorX-Property and SCRATCH-1D on CASP11 and CASP12 (Tables 4-5).
- ASA prediction achieves the highest accuracy among the three compared methods on CASP11 and CASP12 (Table 6).
- Prediction files are present in ./contact_pred and ./ss_pred after running pipeline.py.

## Limitations

- RaptorX-Contact/Property comparisons were run via web server (no local execution file) and differ in MSA construction; results were missing for 3 of 105 CASP11 domains and 1 of 55 CASP12 domains due to server errors.
- Comparability depends on using the same HHblits parameters (three iterations, E-value 0.001, 60% coverage, UniProt20_2016) across all methods.
- Training constraints (e.g. 60-layer depth due to GPU memory) bound achievable accuracy, so reproduction on weaker hardware may differ.

## Evidence

- [discussion] CASP11/CASP12 benchmark sets: "we used the CASP11 (105 domains) and CASP12 (55 domains) dataset [33, 34] obtained from the CASP download area"
- [intro] Comparison baselines: [evidence span withheld — anchor status: not_found]
- [discussion] MSA construction parameters: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [results] Evaluation metric: [evidence span withheld — anchor status: not_found]
- [readme] Prediction pipeline command: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
- [discussion] RaptorX server limitation: "for 3 out of 105 CASP11 domains and for 1 out of 55 CASP12 domains, the results were not retrieved because of server error"
