---
name: hhblits-msa-construction
description: Use when a DNN contact-prediction or multitask pipeline (e.g. DeepECA)
  requires input MSAs for training or test proteins, or when constructing deliberately
  noisy MSAs to test robustness to metagenomic sequences.
when_to_use_negative:
- When pre-computed MSAs or pre-trained models and datasets from the DeepECA Google
  Drive folder are already available and MSA regeneration is unnecessary
- 'When the goal is shallow-MSA accuracy improvement: the article notes that for shallow
  MSAs it is useful to add features derived from the sequence and MSA, and improving
  MSA construction itself is left as future work'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: HHblits
  role: Generates the MSAs used as DNN input features, with three iterations, E-value
    0.001, 60% coverage on UniProt20_2016; also produces noisy MSAs (E-values 1 and
    3, no '-cov') for robustness tests
- name: DeepECA
  role: Downstream framework that consumes the HHblits-generated MSAs for end-to-end
    contact and secondary-structure prediction; run_hhblits_local.py orchestrates
    MSA generation
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:6dc6da0f2d3598aea9fcd4e889192a6d721a5f74515a2b67598d14d6e91932d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hhblits-msa-construction

## Summary

Generate multiple sequence alignments (MSAs) for protein contact and secondary-structure prediction with HHblits, following the DeepECA protocol: three iterations against the UniProt20_2016 library at E-value 0.001 with 60% coverage, plus controlled noisy-MSA variants for robustness testing.

## When to use

Use when a DNN contact-prediction or multitask pipeline (e.g. DeepECA) requires input MSAs for training or test proteins, or when constructing deliberately noisy MSAs to test robustness to metagenomic sequences.

## When NOT to use

- When pre-computed MSAs or pre-trained models and datasets from the DeepECA Google Drive folder are already available and MSA regeneration is unnecessary
- When the goal is shallow-MSA accuracy improvement: the article notes that for shallow MSAs it is useful to add features derived from the sequence and MSA, and improving MSA construction itself is left as future work

## Inputs

- Protein FASTA sequences (training set, CASP11/CASP12 target domains, or user-provided FASTA files placed in ./dataset/fasta)
- UniProt20_2016 HHblits sequence library/database
- run_hhblits_local.py with a correctly configured local hhblits path

## Outputs

- Per-protein MSA files for standard pipeline use (three HHblits iterations, E-value 0.001, 60% coverage)
- Noisy MSA variants (HHblits E-values 1 and 3, '-cov' option removed) for robustness evaluation

## How to apply

Set up HHblits locally and edit the path to hhblits in run_hhblits_local.py. For standard MSAs, run HHblits with three iterations, threshold E-value 0.001 on the UniProt20_2016 library, and 60% coverage to produce alignments for each training or test protein (e.g. CASP11/CASP12 domains). For the MSA-noise robustness test, generate noisy MSAs by setting E-values to 1 and 3 and removing the '-cov' option, which admits more remote and low-coverage sequences. Feed the resulting MSAs to DeepECA training/prediction (multitask.py / pipeline.py), cropping 200 residues and sampling up to 30,000 MSA sequences per domain during training.

## Related tools

- **HHblits** (Generates the MSAs used as DNN input features, with three iterations, E-value 0.001, 60% coverage on UniProt20_2016; also produces noisy MSAs (E-values 1 and 3, no '-cov') for robustness tests)
- **DeepECA** (Downstream framework that consumes the HHblits-generated MSAs for end-to-end contact and secondary-structure prediction; run_hhblits_local.py orchestrates MSA generation) — https://github.com/tomiilab/DeepECA

## Examples

```
python run_hhblits_local.py  # after editing the hhblits path; then python multitask.py (training) or python pipeline.py (prediction)
```

## Evaluation signals

- run_hhblits_local.py contains a valid local path to the hhblits executable before running MSA generation
- Standard MSAs are produced with exactly three HHblits iterations at E-value 0.001, 60% coverage, against the UniProt20_2016 library
- Noisy-MSA variants use E-values 1 and 3 with the '-cov' option removed, and show degraded-but-retained downstream accuracy (Tables 1, 2, 3 in the article)
- MSAs used at training time are cropped to 200 residues and sampled to at most 30,000 sequences per domain
- Prediction runs (python pipeline.py) complete with contact and secondary-structure files present in ./contact_pred and ./ss_pred

## Limitations

- Noisy MSAs intentionally degrade quality; they are only appropriate for robustness experiments, not for maximizing accuracy
- Contact prediction accuracy depends on MSA accuracy (Fox et al.), so poor or shallow MSAs limit final performance; improving MSA construction for shallow MSAs is explicitly left as future work
- Training with these MSAs is expensive: approximately 2-3 days on an NVIDIA TITAN GTX GPU

## Evidence

- [discussion] MSA construction using HHblits with three iterations, E-value 0.001, coverage 60%: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] Threshold E-value 0.001 on UniProt20_2016 library: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] Noisy MSA generation settings for robustness testing: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
- [readme] HHblits setup step in repository README: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
- [discussion] Cropping and sampling parameters applied to generated MSAs during training: "we crop 200 residues from all protein sequences. When the number of sequences in MSAs is greater than 30,000, we sample 30,000 sequences"
