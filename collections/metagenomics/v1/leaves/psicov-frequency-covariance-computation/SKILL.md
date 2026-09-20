---
name: psicov-frequency-covariance-computation
description: Use when a target protein FASTA must be converted into the covariance-tensor
  feature representation (441 channels of LxL matrices) required as direct DNN input
  by DeepECA/DeepCov-style contact predictors.
when_to_use_negative:
- You need precision-matrix features rather than covariance features — ResPRE used
  a precision matrix instead, which is a different input representation
- You only need contact predictions and already possess pre-trained DeepECA models
  and computed features — run the prediction pipeline instead of recomputing covariances
- Proteins outside the supported length regime (training cropped sequences to 200
  residues; dataset filtering removed proteins >700 or <25 residues) may not match
  the trained feature representation
edam_operation: http://edamontology.org/operation_2476
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: DeepECA
  role: Consumes the computed 441-channel covariance tensor as the input feature for
    end-to-end DNN contact prediction
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Builds one MSA per target sequence (3 iterations, E-value 0.001, '-cov 60')
    from which frequencies are computed
- name: PSICOV
  role: Source of the frequency formula used to compute observed single-site and pairwise
    amino-acid frequencies from the MSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:165cdefb2b0196a3339732f00b2aba545fd2fa42438e85dab391527259fdb4ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  - PSICOV
  edam_operation: http://edamontology.org/operation_2476
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# psicov-frequency-covariance-computation

## Summary

Compute per-protein covariance-matrix input features for DeepECA-style end-to-end contact prediction: build one MSA per target with HHblits, extract single-site and pairwise amino-acid frequencies via the PSICOV frequency formula, and assemble 441 L-by-L covariance matrices per protein of length L.

## When to use

Use when a target protein FASTA must be converted into the covariance-tensor feature representation (441 channels of LxL matrices) required as direct DNN input by DeepECA/DeepCov-style contact predictors.

## When NOT to use

- You need precision-matrix features rather than covariance features — ResPRE used a precision matrix instead, which is a different input representation
- You only need contact predictions and already possess pre-trained DeepECA models and computed features — run the prediction pipeline instead of recomputing covariances
- Proteins outside the supported length regime (training cropped sequences to 200 residues; dataset filtering removed proteins >700 or <25 residues) may not match the trained feature representation

## Inputs

- Target protein sequence(s) in FASTA format (placed in ./dataset/fasta)
- Local HHblits installation and the UniProt20_2016 sequence library

## Outputs

- One MSA per target protein (HHblits, 3 iterations, E-value 0.001, -cov 60)
- Per-protein covariance tensor of 441 L-by-L matrices (L = protein length), ready to be fed to the DeepECA networks

## How to apply

Place the target FASTA files into ./dataset/fasta and configure the local HHblits path in run_hhblits_local.py. Run HHblits with three iterations, an E-value threshold of 0.001 against the UniProt20_2016 library, and the '-cov 60' coverage option to produce one MSA per protein. For each MSA, compute observed amino-acid single-site and pairwise frequencies using the PSICOV frequency formula, then assemble the full set of 441 L-by-L covariance matrices for every protein of length L. The rationale is that DeepECA, like DeepCov, feeds the covariance matrix calculated from the MSA directly to the deep neural network for end-to-end contact prediction.

## Related tools

- **DeepECA** (Consumes the computed 441-channel covariance tensor as the input feature for end-to-end DNN contact prediction) — https://github.com/tomiilab/DeepECA
- **HHblits** (Builds one MSA per target sequence (3 iterations, E-value 0.001, '-cov 60') from which frequencies are computed)
- **PSICOV** (Source of the frequency formula used to compute observed single-site and pairwise amino-acid frequencies from the MSA)

## Examples

```
python run_hhblits_local.py  # after placing FASTA files in ./dataset/fasta and setting the hhblits path
```

## Evaluation signals

- Each target protein yields a generated MSA (non-empty HHblits output)
- The covariance tensor has the expected channel count of 441
- Each of the 441 covariance matrices has dimensions equal to the protein length L
- Frequencies are computed with the PSICOV frequency formula from observed single-site and pairwise MSA counts
- HHblits parameters match the protocol: 3 iterations, E-value 0.001, UniProt20_2016 library, '-cov 60'

## Limitations

- DeepECA's authors note improving MSA construction and collecting sufficient evolutionary information for shallow MSAs is left as future work — shallow MSAs yield weaker features
- Noisy MSAs degrade prediction; the paper deliberately produced noisy MSAs (E-values 1 and 3, '-cov' option removed) to test robustness, so relaxed HHblits settings are inappropriate here
- GPU memory limitations constrained network depth, and training data preprocessing (cropping to 200 residues, sampling up to 30,000 MSA sequences) shapes the regime in which the covariance features were trained

## Evidence

- [intro] DeepCov: "The recently reported DeepCov [21], which is a conceptually similar method to ours uses a covariance matrix calculated from MSA for input features for DNN."
- [discussion] HHblits MSA protocol: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [readme] README setup: "When you want to use your original fasta file, put all your fasta files into ./dataset/fasta ."
- [discussion] UniProt library threshold: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
