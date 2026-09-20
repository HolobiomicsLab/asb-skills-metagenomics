---
name: noisy-msa-robustness-testing
description: Use when assessing whether a contact-prediction model is robust to noisy
  MSA input (e.g. metagenomic contamination), or when reproducing DeepECA's noise-reduction
  claim via E-value 1/3 HHblits MSAs.
when_to_use_negative:
- Input MSAs are already high-quality (standard HHblits filtering, E-value 0.001,
  coverage 60%) and the goal is only baseline accuracy rather than noise robustness
- The task is improving accuracy on shallow MSAs via added sequential features — the
  paper treats that as a separate mechanism from noise reduction
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0166
- http://edamontology.org/topic_0080
tools:
- name: HHblits
  role: Generates both clean and deliberately noisy MSAs (E-values 1 and 3, '-cov'
    option removed) for the robustness test; path configured in run_hhblits_local.py
- name: DeepECA
  role: Weighted MSA Model used to predict contacts from the noisy MSAs and learn
    per-sequence weights that down-weight noise
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:ee395553d83c99955ff411e45ff38795833268f44764fa0d28b50d02626d9089
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0166
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Noisy-MSA robustness testing

## Summary

Generate deliberately degraded multiple sequence alignments with HHblits (E-values 1 and 3, coverage filter removed) and evaluate whether a contact-prediction model trained with learned per-sequence weighting (DeepECA Weighted MSA Model) retains accuracy and automatically down-weights the noise. This tests the paper's claim that supervised sequence weighting reduces MSA noise.

## When to use

Use when assessing whether a contact-prediction model is robust to noisy MSA input (e.g. metagenomic contamination), or when reproducing DeepECA's noise-reduction claim via E-value 1/3 HHblits MSAs.

## When NOT to use

- Input MSAs are already high-quality (standard HHblits filtering, E-value 0.001, coverage 60%) and the goal is only baseline accuracy rather than noise robustness
- The task is improving accuracy on shallow MSAs via added sequential features — the paper treats that as a separate mechanism from noise reduction

## Inputs

- Protein sequence files in FASTA format (placed in ./dataset/fasta)
- Trained Weighted MSA Model (pre-trained models from the Google Drive folder or locally trained via multitask.py)
- Clean reference MSAs (HHblits, three iterations, E-value 0.001, coverage 60%)
- Noisy MSAs generated with HHblits at E-values 1 and 3 with the '-cov' option removed

## Outputs

- Contact predictions for each noisy-MSA condition stored in ./contact_pred
- Per-condition contact prediction accuracy values comparable to Table 1 of the paper
- Visualizations of learned per-sequence weight distributions showing down-weighting of noisy sequences

## How to apply

Set up HHblits and edit the HHblits path in run_hhblits_local.py, then produce noisy MSAs by setting E-values 1 and 3 and eliminating the '-cov' option, which relaxes standard MSA quality filtering and injects noise sequences. Run contact prediction with the Weighted MSA Model on these noisy MSAs alongside the clean (E-value 0.001, coverage 60%, three iterations) MSAs. Compare contact-prediction accuracy between clean and noisy conditions (Table 1 in the paper) to verify the model retains accuracy. Additionally, visualize the learned per-sequence weight distributions of the trained MLP module to confirm the model assigns low weights to the noisy sequences introduced by the relaxed HHblits settings.

## Related tools

- **HHblits** (Generates both clean and deliberately noisy MSAs (E-values 1 and 3, '-cov' option removed) for the robustness test; path configured in run_hhblits_local.py)
- **DeepECA** (Weighted MSA Model used to predict contacts from the noisy MSAs and learn per-sequence weights that down-weight noise) — https://github.com/tomiilab/DeepECA

## Examples

```
python run_hhblits_local.py  # E-values 1 and 3, '-cov' option removed
```

## Evaluation signals

- Weighted MSA Model accuracy on E-value 1 and E-value 3 noisy MSAs is retained relative to clean-MSA accuracy (Table 1 of the paper)
- Learned per-sequence weight distributions of the MLP module visibly down-weight sequences that entered via the relaxed HHblits settings
- MSA generation used E-values 1 and 3 with the '-cov' option removed, matching the paper's noise-injection protocol
- Noisy-MSA predictions are written to ./contact_pred and can be evaluated with the same distance-bin / prediction-count accuracy scheme as clean runs

## Limitations

- Noisy MSAs are synthetic degradations (relaxed E-value, removed coverage filter); they approximate but do not fully reproduce real metagenomic contamination
- Robustness relies on the learned weighting module; an unweighted baseline model is expected to degrade more on noisy MSAs
- Improving MSA construction and extracting sufficient evolutionary information from shallow MSAs was left as future work, so robustness does not guarantee accuracy on very shallow inputs

## Evidence

- [intro] Noisy MSA generation protocol: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
- [abstract] Noise-reduction rationale: "our model can provide a means to reduce noise automatically"
- [readme] HHblits setup requirement: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
- [discussion] Clean MSA baseline parameters: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
