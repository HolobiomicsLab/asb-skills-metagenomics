---
name: secondary-structure-constraint-integration
description: Use when you have top-L residue contact lists and predicted secondary
  structures from a deep-learning contact predictor and need to build and score 3D
  protein models.
when_to_use_negative:
- You lack predicted secondary structures and only have contacts — CONFOLD in this
  workflow expects both inputs
- The goal is refining an existing experimental or template-based structure rather
  than de novo folding from contact restraints
- The target protein has no usable MSA, since DeepECA contact predictions require
  an HHblits-generated MSA
edam_operation: http://edamontology.org/operation_0477
edam_topics:
- http://edamontology.org/topic_0736
- http://edamontology.org/topic_0082
tools:
- name: DeepECA
  role: Multi-task deep network producing the contact and secondary-structure predictions
    used as folding restraints
  repo: https://github.com/tomiilab/DeepECA
- name: CONFOLD
  role: Package script that reconstructs tertiary structures from the top 2L contacts
    plus predicted secondary structures
- name: TMscore
  role: Scores each reconstructed model against the native domain structure
- name: HHblits
  role: Generates the MSAs from which DeepECA predicts contacts and secondary structures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:a37601d37e248ad57904fbc30a645fac4e93ff1105f3565934af85b7591ab540
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CONFOLD
  - DeepECA
  - HHblits
  - TMscore
  edam_operation: http://edamontology.org/operation_0477
  edam_topics:
  - http://edamontology.org/topic_0736
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# secondary-structure-constraint-integration

## Summary

Reconstruct tertiary structure models by feeding predicted contacts plus predicted secondary structures from the DeepECA multi-task model into the CONFOLD package script, then quantify model quality with TMscore against native CASP domains.

## When to use

Use when you have top-L residue contact lists and predicted secondary structures from a deep-learning contact predictor and need to build and score 3D protein models.

## When NOT to use

- You lack predicted secondary structures and only have contacts — CONFOLD in this workflow expects both inputs
- The goal is refining an existing experimental or template-based structure rather than de novo folding from contact restraints
- The target protein has no usable MSA, since DeepECA contact predictions require an HHblits-generated MSA

## Inputs

- Predicted contact lists from ./contact_pred (mixed-range residue pairs with probabilities, output of 'python pipeline.py')
- Predicted secondary structures from ./ss_pred
- Native CASP domain structures (PDB) for TMscore evaluation
- Domain length L for top-2L contact selection

## Outputs

- Reconstructed 3D model per domain from CONFOLD
- Per-domain TMscore values versus native structures
- Summary statistics: average TMscore and fraction of domains with TMscore > 0.5

## How to apply

Run the DeepECA multi-task model via 'python pipeline.py', which stores predicted contact lists in ./contact_pred and predicted secondary structures in ./ss_pred. For each target domain (e.g. CASP11/CASP12 domains), select the top 2L mixed-range residue-pair contacts (L = domain length) and pass them together with the predicted secondary structures to a script in the CONFOLD package to reconstruct a 3D model. Rationale: adding secondary structure and solvent-accessibility predictions to the contact model raises accuracy, and CONFOLD converts contact-plus-SS restraints into tertiary folds. Assess each model by comparing it to the native domain with TMscore; a TMscore above 0.5 indicates a correct global fold. The approach can fail on mirror-image folds even at high contact accuracy, so inspect low-TMscore cases rather than trusting contact precision alone.

## Related tools

- **DeepECA** (Multi-task deep network producing the contact and secondary-structure predictions used as folding restraints) — https://github.com/tomiilab/DeepECA
- **CONFOLD** (Package script that reconstructs tertiary structures from the top 2L contacts plus predicted secondary structures)
- **TMscore** (Scores each reconstructed model against the native domain structure)
- **HHblits** (Generates the MSAs from which DeepECA predicts contacts and secondary structures)

## Examples

```
python pipeline.py  # contacts -> ./contact_pred, secondary structures -> ./ss_pred; then run CONFOLD script with top 2L contacts + predicted SS and score with TMscore
```

## Evaluation signals

- Average TMscore over CASP11 (105 domains) reproduces 0.472 and over CASP12 (55 domains) reproduces 0.402
- TMscore > 0.5 count matches 50/105 CASP11 domains and 18/55 CASP12 domains
- Best-case checks: T0811-D1 TMscore 0.818 (85.3% top-L contact accuracy) and T0920-D1 TMscore 0.848 (86.3%)
- Known failure reproduced: T0845-D1 shows 86.6% contact accuracy but mirror-image TMscore of only 0.276
- Sanity check that ./contact_pred and ./ss_pred directories exist and were freshly populated by 'python pipeline.py'

## Limitations

- Mirror-image failures can occur despite high contact accuracy (T0845-D1: 86.6% contacts, TMscore 0.276); more precise contact information would be necessary to solve such problems
- More sophisticated tertiary structure construction methods are acknowledged as necessary beyond the CONFOLD script
- DeepECA runs on TensorFlow r1.11.0 on Python3 (tested on Ubuntu 16.04), and pre-trained models/datasets must be downloaded separately from the linked Google Drive folder

## Evidence

- [discussion] CONFOLD folding with top 2L contacts: [evidence span withheld — anchor status: not_found]
- [intro] CONFOLD and TMscore evaluation: "we attempt to construct tertiary structures using the CONFOLD script [28]. We measure the quality of predicted structures in terms of the TMscore."
- [intro] TMscore thresholds on CASP benchmarks: "retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains"
- [abstract] Multi-task model rationale: "we expanded our model to a multi-task model to achieve higher accuracy by incorporating predictions of secondary structures and solvent-accessible surface areas."
- [readme] README prediction command and output locations: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
