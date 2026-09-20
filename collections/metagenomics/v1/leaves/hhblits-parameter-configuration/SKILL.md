---
name: hhblits-parameter-configuration
description: Use when a target protein FASTA must be converted into a high-quality
  MSA as input to covariance-matrix-based contact prediction, and HHblits run parameters
  (iterations, E-value, coverage, database path) need to be set.
when_to_use_negative:
- When the goal is to stress-test model robustness against noisy MSAs — the article
  instead relaxes E-values to 1 and 3 and removes the '-cov' option to deliberately
  produce noisy MSAs
- When shallow MSAs are the target and the goal is adding sequential features rather
  than strict MSA construction — the article leaves improving MSA construction for
  shallow MSAs as future work
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: HHblits
  role: Generates one MSA per target protein using three iterations, E-value 0.001,
    '-cov 60' coverage, against UniProt20_2016
- name: DeepECA
  role: Downstream framework consuming the HHblits-built MSAs for covariance-matrix-based
    contact prediction
  repo: https://github.com/tomiilab/DeepECA
- name: PSICOV
  role: Frequency formula used on the generated MSAs to compute single-site and pairwise
    frequencies for the covariance features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:f5b4863cd93bbee66a2891c3eabf8fb4b6e74c65516ac7494d2e63652990dd37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  - PSICOV
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hhblits-parameter-configuration

## Summary

Configures HHblits to build one multiple sequence alignment (MSA) per target protein for DeepECA contact prediction, using three iterations, E-value threshold 0.001 against the UniProt20_2016 library, and 60% coverage ('-cov 60').

## When to use

Use when a target protein FASTA must be converted into a high-quality MSA as input to covariance-matrix-based contact prediction, and HHblits run parameters (iterations, E-value, coverage, database path) need to be set.

## When NOT to use

- When the goal is to stress-test model robustness against noisy MSAs — the article instead relaxes E-values to 1 and 3 and removes the '-cov' option to deliberately produce noisy MSAs
- When shallow MSAs are the target and the goal is adding sequential features rather than strict MSA construction — the article leaves improving MSA construction for shallow MSAs as future work

## Inputs

- Target protein sequences in FASTA format (placed in ./dataset/fasta)
- Local UniProt20_2016 HHblits sequence database
- Local HHblits installation with path set in run_hhblits_local.py

## Outputs

- One HHblits-generated MSA per target protein (input for PSICOV frequency and covariance-matrix computation)

## How to apply

Install HHblits locally and edit the hhblits path in run_hhblits_local.py so the script can invoke it. Run HHblits for each target sequence with three iterations against the UniProt20_2016 library, E-value threshold 0.001, and the '-cov 60' coverage option, producing one MSA per protein; place FASTA files into ./dataset/fasta first. The strict E-value and coverage settings yield deep, high-quality MSAs, which the DeepECA authors contrast with deliberately noisy MSAs produced by relaxing E-values to 1 and 3 and removing the '-cov' option for robustness testing. Confirm each target yields a non-empty MSA suitable for downstream PSICOV frequency-formula computation and assembly of the 441 L-by-L covariance matrices.

## Related tools

- **HHblits** (Generates one MSA per target protein using three iterations, E-value 0.001, '-cov 60' coverage, against UniProt20_2016)
- **DeepECA** (Downstream framework consuming the HHblits-built MSAs for covariance-matrix-based contact prediction) — https://github.com/tomiilab/DeepECA
- **PSICOV** (Frequency formula used on the generated MSAs to compute single-site and pairwise frequencies for the covariance features)

## Examples

```
python run_hhblits_local.py  # hhblits path configured in-script; run with 3 iterations, E-value 0.001, '-cov 60' on UniProt20_2016
```

## Evaluation signals

- Each target protein in ./dataset/fasta yields a generated, non-empty MSA
- The HHblits run used exactly three iterations with E-value threshold 0.001 on the UniProt20_2016 library
- The '-cov 60' coverage option was applied so aligned sequences cover at least 60% of the target
- Downstream covariance tensor has the expected 441 channels and dimensions equal to protein length L
- run_hhblits_local.py points to a working local hhblits executable (no path errors during execution)

## Limitations

- The strict E-value/coverage settings target deep MSAs; the authors note improving MSA construction and extracting information from shallow MSAs remains future work
- Training on the prepared data takes 2–3 days on an NVIDIA TITAN GTX GPU, and GPU memory limited the network to 60 layers, indirectly constraining the pipeline's scale
- The README was tested on Ubuntu 16.04 with TensorFlow r1.11.0; other environments are unverified

## Evidence

- [discussion] MSA construction using HHblits with three iterations, E-value 0.001, coverage 60%: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] E-value and database configuration: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] Coverage option and noisy-MSA contrast: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
- [readme] Local HHblits setup requirement: "You need to set up hhblits for generation of MSA. Please edit the path to hhblits in run_hhblits_local.py appropriately."
