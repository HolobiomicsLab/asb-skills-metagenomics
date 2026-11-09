---
name: contact-restraint-selection-top-2l
description: Use when you have ranked residue-pair contact predictions and secondary-structure
  predictions for a protein domain and need distance restraints for ab initio 3D model
  reconstruction and TMscore-based quality evaluation.
when_to_use_negative:
- The goal is contact prediction itself (MSA-based DNN contact prediction) rather
  than converting already-predicted contacts into 3D models
- Native structural templates are available and template-based modeling would be more
  appropriate than restraint-only reconstruction
- The contact list is unranked or lacks mixed-range residue-pair probabilities, so
  a top-2L selection cannot be performed
edam_operation: http://edamontology.org/operation_0476
edam_topics:
- http://edamontology.org/topic_0082
tools:
- name: DeepECA
  role: Multitask model whose predicted contacts (./contact_pred) and secondary structures
    (./ss_pred) supply the restraints; run via 'python pipeline.py'
  repo: https://github.com/tomiilab/DeepECA
- name: CONFOLD
  role: Package script that reconstructs the 3D model from the top 2L contact restraints
    plus predicted secondary structures
- name: TMscore
  role: Scores each reconstructed model against the native CASP domain structure to
    measure model quality
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:6c56ddc9c80cff8fd27e8faa061d652776f7a827577ed108559e41639ef40fb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CONFOLD
  - DeepECA
  - TMscore
  edam_operation: http://edamontology.org/operation_0476
  edam_topics:
  - http://edamontology.org/topic_0082
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# contact-restraint-selection-top-2l

## Summary

Select the top 2L mixed-range residue-pair contacts (L = domain length) from a DNN contact predictor's ranked list and combine them with predicted secondary structures to reconstruct tertiary protein structures with a CONFOLD package script, scoring the models with TMscore. This tests whether contacts plus secondary structures alone suffice to build usable 3D models.

## When to use

Use when you have ranked residue-pair contact predictions and secondary-structure predictions for a protein domain and need distance restraints for ab initio 3D model reconstruction and TMscore-based quality evaluation.

## When NOT to use

- The goal is contact prediction itself (MSA-based DNN contact prediction) rather than converting already-predicted contacts into 3D models
- Native structural templates are available and template-based modeling would be more appropriate than restraint-only reconstruction
- The contact list is unranked or lacks mixed-range residue-pair probabilities, so a top-2L selection cannot be performed

## Inputs

- Ranked predicted contact list per domain (mixed-range residue pairs, from ./contact_pred produced by the multitask model)
- Predicted secondary structures per domain (from ./ss_pred produced by the multitask model)
- Target domain length L (number of residues)
- Native CASP domain structures (PDB) for TMscore evaluation

## Outputs

- Top 2L contact distance restraints per domain
- Reconstructed 3D tertiary structure model per domain (from CONFOLD)
- Per-domain TMscore summary table with aggregate statistics (average TMscore, count of TMscore > 0.5 domains)

## How to apply

Run the multitask contact predictor (DeepECA) via 'python pipeline.py' to obtain predicted contact lists in ./contact_pred and predicted secondary structures in ./ss_pred. For each CASP domain of length L, select up to the top 2L mixed-range residue-pair contacts from the ranked contact list to serve as distance restraints. Feed these restraints plus the predicted secondary structures into a script from the CONFOLD package to reconstruct a 3D model per domain. Compare each reconstructed model to the native CASP domain structure using TMscore, then aggregate per-domain scores; in this framework the benchmark achieved average TMscore 0.472 on CASP11 (105 domains) and 0.402 on CASP12 (55 domains).

## Related tools

- **DeepECA** (Multitask model whose predicted contacts (./contact_pred) and secondary structures (./ss_pred) supply the restraints; run via 'python pipeline.py') — https://github.com/tomiilab/DeepECA
- **CONFOLD** (Package script that reconstructs the 3D model from the top 2L contact restraints plus predicted secondary structures)
- **TMscore** (Scores each reconstructed model against the native CASP domain structure to measure model quality)

## Examples

```
python pipeline.py
```

## Evaluation signals

- Per-domain restraint count equals 2L (or fewer when the contact list is shorter), with only mixed-range residue pairs selected
- Aggregate reproduction check: average TMscore of 0.472 on 105 CASP11 domains and 0.402 on 55 CASP12 domains
- TMscore > 0.5 counts reproduce 50/105 CASP11 and 18/55 CASP12 domains
- Best-model sanity checks: T0811-D1 (TMscore 0.818, 85.3% top-L contact accuracy) and T0920-D1 (TMscore 0.848, 86.3%)
- Known failure case reproduced: T0845-D1 reaches 86.6% contact accuracy yet only TMscore 0.276 (mirror-image model)

## Limitations

- High contact accuracy does not guarantee a correct fold: T0845-D1 reached 86.6% contact accuracy but TMscore 0.276 due to a mirror-image-like failure
- More precise contact information would be needed to resolve mirror-image-like failures, and more sophisticated tertiary structure construction methods are acknowledged as necessary
- Only up to the top 2L contacts are used, so restraint selection may discard informative pairs; models were built with a CONFOLD script rather than a state-of-the-art assembly method

## Evidence

- [discussion] we used a script in the CONFOLD package ... then we used (up to) the top 2 L contacts among them as inputs for the script: [evidence span withheld — anchor status: not_found]
- [intro] we attempt to construct tertiary structures using the CONFOLD script [28]. We measure the quality of predicted structures in terms of the TMscore.: "we attempt to construct tertiary structures using the CONFOLD script [28]. We measure the quality of predicted structures in terms of the TMscore."
- [intro] retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains and 18 out of 55 (33%) CASP12 domains.: "retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains and 18 out of 55 (33%) CASP12 domains."
- [readme] The results will be stored in ./contact_pred and ./ss_pred directory.: "The results will be stored in ./contact_pred and ./ss_pred directory."
- [discussion] More precise contact information would be necessary to solve such a mirror image-like problem: "More precise contact information would be necessary to solve such a mirror image-like problem"
