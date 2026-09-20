---
name: msa-input-feature-preparation-for-dnn
description: Use when a target protein sequence (FASTA) must be converted into the
  DNN input feature tensor (MSA-derived covariance matrices) for DeepECA/DeepCov-style
  contact prediction.
when_to_use_negative:
- When the input is already a per-residue-pair feature set from other ECA programs
  (e.g. PSICOV, CCMpred, FreeContact outputs used by meta-predictors like MetaPSICOV)
  — this skill instead derives features directly from the MSA
- When a precision-matrix (not covariance-matrix) input is required, as in the ResPRE-style
  approach
- When shallow MSAs with insufficient evolutionary information are the only option
  — the paper notes improving MSA construction for shallow MSAs was left as future
  work
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DeepECA
  role: Target framework whose DNN consumes the covariance-tensor input features for
    end-to-end contact prediction
  repo: github.com/tomiilab/DeepECA
- name: HHblits
  role: Builds one MSA per target sequence (three iterations, E-value 0.001, '-cov
    60' against UniProt20_2016)
- name: PSICOV
  role: Source of the frequency formula used to compute the observed amino-acid single-site
    and pairwise frequencies from each MSA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:c6c4f2ee32a7e798f727adf4a2d4b24d6acad58075590217109a5b93589ab26e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepECA
  - HHblits
  - PSICOV
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# msa-input-feature-preparation-for-dnn

## Summary

Prepares the input features for DeepECA's end-to-end contact prediction by building one MSA per target protein with HHblits and computing the PSICOV single-site and pairwise frequencies, assembled into 441 L-by-L covariance matrices per protein of length L that feed the deep neural network directly.

## When to use

Use when a target protein sequence (FASTA) must be converted into the DNN input feature tensor (MSA-derived covariance matrices) for DeepECA/DeepCov-style contact prediction.

## When NOT to use

- When the input is already a per-residue-pair feature set from other ECA programs (e.g. PSICOV, CCMpred, FreeContact outputs used by meta-predictors like MetaPSICOV) — this skill instead derives features directly from the MSA
- When a precision-matrix (not covariance-matrix) input is required, as in the ResPRE-style approach
- When shallow MSAs with insufficient evolutionary information are the only option — the paper notes improving MSA construction for shallow MSAs was left as future work

## Inputs

- Target protein sequences in FASTA format (placed into ./dataset/fasta)
- UniProt20_2016 HHblits sequence library
- Locally installed HHblits (path configured in run_hhblits_local.py)

## Outputs

- One MSA per target protein (built with three HHblits iterations, E-value 0.001, '-cov 60')
- Per-protein amino-acid single-site and pairwise observed frequencies (PSICOV frequency formula)
- Covariance feature tensor of 441 L-by-L channels (L = protein length) per protein, ready for the DeepECA network

## How to apply

Place the target FASTA files into ./dataset/fasta and configure the hhblits path in run_hhblits_local.py. Run HHblits for each sequence with three iterations, an E-value threshold of 0.001 against the UniProt20_2016 library, and the '-cov 60' coverage option to build one MSA per protein. For each MSA, compute the observed amino-acid single-site and pairwise frequencies using the PSICOV frequency formula. Assemble, for every protein of length L, the full set of 441 L-by-L covariance matrices derived from those frequencies — this tensor (441 channels × L × L) is the end-to-end input to the DeepECA networks, replacing hand-crafted per-residue-pair features used by prior ECA methods.

## Related tools

- **DeepECA** (Target framework whose DNN consumes the covariance-tensor input features for end-to-end contact prediction) — github.com/tomiilab/DeepECA
- **HHblits** (Builds one MSA per target sequence (three iterations, E-value 0.001, '-cov 60' against UniProt20_2016))
- **PSICOV** (Source of the frequency formula used to compute the observed amino-acid single-site and pairwise frequencies from each MSA)

## Examples

```
python run_hhblits_local.py  # after editing the hhblits path and placing FASTA files in ./dataset/fasta
```

## Evaluation signals

- Each target protein yields a generated MSA (non-empty alignment from HHblits)
- Each covariance tensor has the expected channel count of 441
- Each covariance tensor's matrix dimensions equal the protein length L
- Tensor can be passed directly to the DeepECA networks without further feature engineering
- HHblits parameters match the protocol: 3 iterations, E-value threshold 0.001, '-cov 60' coverage option

## Limitations

- The '-cov 60' and strict E-value (0.001) settings produce clean MSAs; the paper deliberately generated noisy MSAs by setting E-values to 1 and 3 and removing the '-cov' option to test robustness
- Proteins with very shallow MSAs lack sufficient evolutionary information; the paper notes improving MSA construction and collecting wider sequence-space information as future work
- GPU memory limitations constrained the networks (60 layers), which bounds the practical input size

## Evidence

- [discussion] MSA construction using HHblits with three iterations, E-value 0.001, coverage 60%: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [intro] DeepCov uses covariance matrix from MSA as DNN input features: "The recently reported DeepCov [21], which is a conceptually similar method to ours uses a covariance matrix calculated from MSA for input features for DNN."
- [readme] HHblits setup required for MSA generation: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
- [discussion] E-value threshold 0.001 on UniProt20_2016 library: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] Noisy MSAs produced by relaxing HHblits parameters: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
