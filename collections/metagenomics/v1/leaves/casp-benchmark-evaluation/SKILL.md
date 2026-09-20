---
name: casp-benchmark-evaluation
description: Use when a contact prediction model must be benchmarked on standard CASP
  target domains, or when accuracy gains (e.g., from features, ensembling, or MSA
  depth) must be quantified at top L/5, L/2, and L on held-out CASP sets.
when_to_use_negative:
- The goal is comparison on a non-CASP benchmark (e.g., the 150-protein PSICOV dataset),
  where the CASP11/CASP12 domain sets and their accuracy tables do not apply
- Inputs are already a final contact list — this skill covers benchmark scoring of
  probability outputs, not re-scoring of published contact rankings without native
  structures
- RaptorX-Contact-style server-only predictors where no local execution file is available
  and MSAs differ from those used for the evaluated methods
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0736
tools:
- name: DeepECA
  role: Multitask deep neural network whose contact probability outputs (via pipeline.py)
    are the predictions evaluated against CASP benchmarks
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs for test targets (three iterations, E-value 0.001, 60%
    coverage), whose depth determines the MSA-depth bins used in evaluation
- name: CONFOLD
  role: Reconstructs 3D models from top 2L contacts plus predicted secondary structures
    for TMscore-based structure-level evaluation
- name: TMscore
  role: Scores each CONFOLD-reconstructed model against the native CASP domain structure
    to produce per-domain TMscores
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:b87a5fa8bfea7710c9206c0ea370b050e4d90fcfce99385b034cc1f5f457259e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CONFOLD
  - DeepECA
  - HHblits
  - TMscore
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0736
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# casp-benchmark-evaluation

## Summary

Evaluate protein contact prediction models against the CASP11 (105 domains) and CASP12 (55 domains) benchmark sets, scoring top L/10, L/5, L/2, and L accuracy per range category and binning proteins by MSA depth; optionally extend to tertiary-structure quality via TMscore on CONFOLD-reconstructed models.

## When to use

Use when a contact prediction model must be benchmarked on standard CASP target domains, or when accuracy gains (e.g., from features, ensembling, or MSA depth) must be quantified at top L/5, L/2, and L on held-out CASP sets.

## When NOT to use

- The goal is comparison on a non-CASP benchmark (e.g., the 150-protein PSICOV dataset), where the CASP11/CASP12 domain sets and their accuracy tables do not apply
- Inputs are already a final contact list — this skill covers benchmark scoring of probability outputs, not re-scoring of published contact rankings without native structures
- RaptorX-Contact-style server-only predictors where no local execution file is available and MSAs differ from those used for the evaluated methods

## Inputs

- CASP11 target protein domains (105 domains) with native structures
- CASP12 target protein domains (55 domains) with native structures
- Target FASTA files placed in ./dataset/fasta
- Pre-trained DeepECA models downloaded from the Google Drive deposit
- Predicted contact probability lists in ./contact_pred
- Predicted secondary structures in ./ss_pred

## Outputs

- Per-target and aggregate contact prediction accuracy at top L/10, L/5, L/2, and L, per range category (short/medium/long)
- Accuracy stratified by MSA depth bins (e.g., accuracy_comparison.csv showing Feature Added vs Weighted MSA Model)
- Ensemble vs single-model accuracy tables (Tables 8-9) and fold-count comparison (Table 10)
- Per-domain TMscore summary table for CONFOLD-reconstructed 3D models

## How to apply

Obtain the CASP11 (105 domains) and CASP12 (55 domains) test sets from the CASP download area, and place target FASTA files in ./dataset/fasta. Run the prediction pipeline (python pipeline.py) with the multitask model to emit raw contact probabilities into ./contact_pred (and secondary structures into ./ss_pred); for ensembles, average residue-pair probabilities across cross-validation folds (5- or 10-fold). Define a contact as a Cβ–Cβ distance < 8 Å (Cα for glycine) and compute accuracy at top L/10, L/5, L/2, and L for short/medium/long-range categories, binning proteins by the number of homologous sequences in the MSA to expose shallow-MSA behavior. For structure-level assessment, feed the top 2L mixed-range contacts plus predicted secondary structures to CONFOLD and score each 3D model against the native domain with TMscore. Correct application is confirmed when aggregate statistics match reported values (e.g., average TMscore 0.472 on CASP11, 0.402 on CASP12) and per-domain counts (TMscore > 0.5 for 50/105 CASP11 and 18/55 CASP12 domains) are reproduced.

## Related tools

- **DeepECA** (Multitask deep neural network whose contact probability outputs (via pipeline.py) are the predictions evaluated against CASP benchmarks) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs for test targets (three iterations, E-value 0.001, 60% coverage), whose depth determines the MSA-depth bins used in evaluation)
- **CONFOLD** (Reconstructs 3D models from top 2L contacts plus predicted secondary structures for TMscore-based structure-level evaluation)
- **TMscore** (Scores each CONFOLD-reconstructed model against the native CASP domain structure to produce per-domain TMscores)

## Examples

```
python pipeline.py
```

## Evaluation signals

- Contact accuracy at long-range top L/5, L/2, and L must match or exceed baseline values (e.g., Feature Added Model ≥ Weighted MSA Model, especially for shallow MSAs)
- Ensemble accuracy exceeds single-model accuracy in almost all categories on both CASP sets, except Medium top L/10 on CASP12
- 10-fold ensemble yields the highest accuracy in almost all prediction categories compared with 3-, 5-, and 7-fold ensembles
- Average TMscore reproduces 0.472 on CASP11 (105 domains) and 0.402 on CASP12 (55 domains)
- TMscore > 0.5 count reproduces 50/105 CASP11 and 18/55 CASP12 domains, with known extremes: T0811-D1 (TMscore 0.818), T0920-D1 (TMscore 0.848), and T0845-D1 mirror-image failure (86.6% contact accuracy but TMscore 0.276)

## Limitations

- RaptorX-Contact comparison results are missing for 3 of 105 CASP11 domains and 1 of 55 CASP12 domains because of server errors, and its predictions rely on a web server with differing MSAs
- Mirror-image tertiary structure failures can occur despite high contact accuracy (T0845-D1: 86.6% contact accuracy but TMscore 0.276); more precise contact information would be needed to resolve such cases
- Improving MSA construction and collecting sufficient evolutionary information for shallow MSAs is acknowledged as future work, so shallow-MSA gains are bounded by MSA quality

## Evidence

- [discussion] CASP datasets from download area: "we used the CASP11 (105 domains) and CASP12 (55 domains) dataset [33, 34] obtained from the CASP download area"
- [discussion] Contact definition: "we defined a contact if the distance of Cβ atoms of the residue pair was less than 8 Å"
- [intro] TMscore benchmark counts: "retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains"
- [readme] Pipeline output directories: "The results will be stored in ./contact_pred and ./ss_pred directory."
- [discussion] Top 2L contacts for structure reconstruction: "we used (up to) the top 2 L contacts among them as inputs for the script"
