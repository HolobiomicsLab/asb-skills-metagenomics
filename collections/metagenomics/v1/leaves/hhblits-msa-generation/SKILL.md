---
name: hhblits-msa-generation
description: Use when a target protein is given as a FASTA sequence and a downstream
  covariance-matrix-based contact predictor (e.g. DeepECA) requires one MSA per protein
  as input.
when_to_use_negative:
- The analysis already starts from a computed covariance matrix or precision matrix
  (e.g. ResPRE-style input) rather than raw sequences needing MSA construction
- The goal is to test model robustness to shallow or noisy MSAs — in that case intentionally
  use E-values 1 or 3 and omit the '-cov' option instead of the standard 0.001 / '-cov
  60' settings
- MSAs are already provided in the pre-packaged Google Drive datasets and only model
  training or prediction is needed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0160
tools:
- name: HHblits
  role: Generates one MSA per target protein from the input FASTA using three iterations,
    E-value 0.001 on the UniProt20_2016 library, and 60% coverage ('-cov 60')
- name: DeepECA
  role: Consumes the generated MSAs via run_hhblits_local.py / pipeline.py, converting
    them into covariance-matrix input features for contact prediction
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:b6dc6489b30310854a9b899ccf08bc7b01a20baad8a56a4b858230b068d64244
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
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# hhblits-msa-generation

## Summary

Generate a per-protein multiple sequence alignment (MSA) with HHblits as the input feature basis for DeepECA's end-to-end deep-learning contact prediction, which converts the MSA into 441 L-by-L covariance matrices.

## When to use

Use when a target protein is given as a FASTA sequence and a downstream covariance-matrix-based contact predictor (e.g. DeepECA) requires one MSA per protein as input.

## When NOT to use

- The analysis already starts from a computed covariance matrix or precision matrix (e.g. ResPRE-style input) rather than raw sequences needing MSA construction
- The goal is to test model robustness to shallow or noisy MSAs — in that case intentionally use E-values 1 or 3 and omit the '-cov' option instead of the standard 0.001 / '-cov 60' settings
- MSAs are already provided in the pre-packaged Google Drive datasets and only model training or prediction is needed

## Inputs

- Target protein sequences in FASTA format (placed in ./dataset/fasta)
- UniProt20_2016 HHblits sequence library
- Locally installed HHblits

## Outputs

- One HHblits MSA per target protein
- Covariance-matrix input tensors (441 L-by-L channels) derived from the MSAs for DeepECA

## How to apply

Place all target FASTA files into ./dataset/fasta and configure the local HHblits installation by editing the hhblits path in run_hhblits_local.py. Run HHblits with three iterations against the UniProt20_2016 library using an E-value threshold of 0.001 and the '-cov 60' coverage option so that one MSA is built per protein. The downstream DeepECA workflow (per the PSICOV frequency formula) then computes single-site and pairwise frequencies from each MSA to assemble 441 L-by-L covariance matrices of channel count 441 and dimensions equal to protein length L. For robustness experiments, deliberately degrade MSAs by setting E-values to 1 or 3 and dropping the '-cov' option.

## Related tools

- **HHblits** (Generates one MSA per target protein from the input FASTA using three iterations, E-value 0.001 on the UniProt20_2016 library, and 60% coverage ('-cov 60'))
- **DeepECA** (Consumes the generated MSAs via run_hhblits_local.py / pipeline.py, converting them into covariance-matrix input features for contact prediction) — https://github.com/tomiilab/DeepECA

## Examples

```
cp my_target.fasta dataset/fasta/ && python pipeline.py
```

## Evaluation signals

- Every target protein in ./dataset/fasta yields a generated MSA after the HHblits run
- The resulting covariance tensor for a protein of length L has the expected channel count of 441 and per-channel dimensions equal to L
- HHblits was invoked with exactly three iterations, E-value threshold 0.001, and the '-cov 60' coverage option against the UniProt20_2016 library
- The hhblits path in run_hhblits_local.py points to a working local HHblits installation (no execution errors)

## Limitations

- MSA construction quality is a known open problem: the authors left 'improving the MSA construction process and collecting sufficient evolutional information from wider sequence spaces' as future work
- Shallow MSAs can degrade contact prediction accuracy; adding sequence/MSA-derived features helps but does not fully resolve the issue
- Training with MSAs cropped to 200 residues and sampled to at most 30,000 sequences takes roughly 2–3 days on an NVIDIA TITAN GTX GPU
- GPU memory limits constrained the downstream network depth to 60 layers

## Evidence

- [discussion] MSA construction parameters: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] E-value threshold and library: "The threshold E-value was set to 0.001 on the UniProt20_2016 library"
- [intro] Noisy MSA variant for robustness testing: "We use HHblits and set E-values 1 and 3 and eliminate the “-cov” option to produce noisy MSAs"
- [readme] README setup instructions: "Please edit the path to hhblits in run_hhblits_local.py appropriately."
- [readme] FASTA placement for original targets: "When you want to use your original fasta file, put all your fasta files into ./dataset/fasta ."
