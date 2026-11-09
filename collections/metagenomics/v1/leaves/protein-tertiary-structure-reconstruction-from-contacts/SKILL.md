---
name: protein-tertiary-structure-reconstruction-from-contacts
description: Use when you already have a per-domain list of predicted residue-pair
  contacts and predicted secondary structures and need to fold them into 3D models
  whose quality is measured by TMscore.
when_to_use_negative:
- The goal is contact or secondary-structure prediction itself — this skill starts
  from those predictions and folds structures, it does not produce the ./contact_pred
  and ./ss_pred inputs.
- Contact-map analysis or top-L accuracy benchmarking is the end goal — high contact
  accuracy does not guarantee model quality (e.g., T0845-D1 reached 86.6% contact
  accuracy but TMscore 0.276).
- No predicted secondary structures are available and CONFOLD must be run with contacts
  alone, deviating from the article's input protocol.
edam_operation: http://edamontology.org/operation_0474
tools:
- name: DeepECA
  role: Multitask model that produces the ./contact_pred and ./ss_pred inputs consumed
    by this skill (run via 'python pipeline.py').
  repo: https://github.com/tomiilab/DeepECA
- name: CONFOLD
  role: Package script that reconstructs the 3D model from the top 2L contacts plus
    the predicted secondary structures.
- name: TMscore
  role: Scoring tool that measures the quality of each reconstructed structure against
    the native CASP domain.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:e57cadd1abdaa17592d16b241420de0577bf6307a1186ea0c1129fa68de03528
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CONFOLD
  - DeepECA
  - TMscore
  edam_operation: http://edamontology.org/operation_0474
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# protein-tertiary-structure-reconstruction-from-contacts

## Summary

Reconstructs protein tertiary structures from predicted residue contacts and secondary structures using the CONFOLD script, then quantifies model quality with TMscore against native CASP domain structures. It bridges contact-prediction output (e.g., from DeepECA's multitask model) to an assessable 3D model.

## When to use

Use when you already have a per-domain list of predicted residue-pair contacts and predicted secondary structures and need to fold them into 3D models whose quality is measured by TMscore.

## When NOT to use

- The goal is contact or secondary-structure prediction itself — this skill starts from those predictions and folds structures, it does not produce the ./contact_pred and ./ss_pred inputs.
- Contact-map analysis or top-L accuracy benchmarking is the end goal — high contact accuracy does not guarantee model quality (e.g., T0845-D1 reached 86.6% contact accuracy but TMscore 0.276).
- No predicted secondary structures are available and CONFOLD must be run with contacts alone, deviating from the article's input protocol.

## Inputs

- Predicted residue-pair contact list per domain (./contact_pred output from the multitask model)
- Predicted secondary structure per domain (./ss_pred output from the multitask model)
- Domain length L (e.g., from the FASTA sequence of each CASP domain)
- Native CASP domain structures (CASP11 and CASP12 download-area sets) for TMscore comparison

## Outputs

- Reconstructed 3D model per CASP domain (CONFOLD output)
- Per-domain TMscore comparing each model to its native domain
- Summary table of TMscore statistics (average, and count of domains with TMscore > 0.5)

## How to apply

Collect the predicted contact lists from ./contact_pred and the predicted secondary structures from ./ss_pred (both produced by DeepECA's multitask model via 'python pipeline.py'). For each CASP domain, select (up to) the top 2L mixed-range residue-pair contacts, where L is the domain length, and use them as distance restraints together with the predicted secondary structures as inputs to a script in the CONFOLD package. Run CONFOLD to reconstruct one 3D model per domain, then compare each model to its corresponding native CASP domain structure using TMscore. Aggregate the per-domain TMscores into a summary table and validate the run against the reported statistics: average TMscore 0.472 on CASP11 (105 domains) and 0.402 on CASP12 (55 domains), with TMscore > 0.5 for 50/105 CASP11 and 18/55 CASP12 domains.

## Related tools

- **DeepECA** (Multitask model that produces the ./contact_pred and ./ss_pred inputs consumed by this skill (run via 'python pipeline.py').) — https://github.com/tomiilab/DeepECA
- **CONFOLD** (Package script that reconstructs the 3D model from the top 2L contacts plus the predicted secondary structures.)
- **TMscore** (Scoring tool that measures the quality of each reconstructed structure against the native CASP domain.)

## Examples

```
python pipeline.py
```

## Evaluation signals

- Computed average TMscores match the reported 0.472 (CASP11, 105 domains) and 0.402 (CASP12, 55 domains).
- Counts of domains with TMscore > 0.5 reproduce 50/105 (CASP11) and 18/55 (CASP12).
- Best models reproduce T0811-D1 (TMscore 0.818, 85.3% top-L accuracy) and T0920-D1 (TMscore 0.848, 86.3% top-L accuracy).
- The known mirror-image failure case reproduces: T0845-D1 has 86.6% contact accuracy but TMscore 0.276.
- Exactly the top 2L mixed-range contacts (L = domain length) were used as restraints — verify the restraint count per domain.

## Limitations

- Mirror-image failure: T0845-D1 built a mirror-image-like structure despite high (86.6%) contact accuracy, giving TMscore 0.276; more precise contact information would be needed to resolve this.
- The article acknowledges that more sophisticated tertiary structure construction methods are necessary beyond CONFOLD with top 2L contacts.
- Quality depends on restraint quality — only (up to) the top 2L contacts are used, so noisy or sparse contact predictions directly degrade models.

## Evidence

- [intro] Tertiary structure prediction from contacts using a CONFOLD package script with top 2L contacts: "we attempt to construct tertiary structures using the CONFOLD script [28]. We measure the quality of predicted structures in terms of the TMscore."
- [intro] retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains: "retrieve a TMscore [25] greater than 0.5 for 50 out of 105 (48%) CASP11 domains"
- [discussion] More precise contact information is needed to resolve mirror image-like tertiary structure failures (e.g. T0845-D1): "More precise contact information would be necessary to solve such a mirror image-like problem"
- [readme] Prediction command and output directories from the repository README: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
