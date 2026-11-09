---
name: deep-neural-network-contact-prediction-training
description: Use when reproducing or extending the DeepECA Feature Added Model for
  long-range contact prediction from MSAs, particularly to test whether sequential
  features improve accuracy when homologous sequence depth is small.
when_to_use_negative:
- The goal is contact prediction from a single sequence or from non-MSA-derived covariance
  features (e.g. PDB structures directly) — this skill requires a multiple sequence
  alignment as input
- The task is MSA construction quality improvement itself; the article leaves improving
  MSA construction and collecting evolutionary information for shallow MSAs as future
  work
- GPU memory cannot accommodate the 60-layer network, which the authors chose because
  of GPU memory limitations
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3474
tools:
- name: DeepECA
  role: Framework providing the Feature Added Model, multitask.py training script,
    and pre-trained models/datasets
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates MSAs for training and test proteins (run_hhblits_local.py; three
    iterations, E-value 0.001, 60% coverage)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:9c4faf4fd04666d15a98fe2241fd6d307793622c4ead79736e91f71c615e0bf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# deep-neural-network-contact-prediction-training

## Summary

Train the Feature Added deep neural network from DeepECA to predict protein residue-residue contacts directly from a multiple sequence alignment, augmenting weighted-MSA covariance features with sequential features (one-hot query, PSSM, entropy, mutual information, sequence separations) to boost long-range contact accuracy, especially for shallow MSAs.

## When to use

Use when reproducing or extending the DeepECA Feature Added Model for long-range contact prediction from MSAs, particularly to test whether sequential features improve accuracy when homologous sequence depth is small.

## When NOT to use

- The goal is contact prediction from a single sequence or from non-MSA-derived covariance features (e.g. PDB structures directly) — this skill requires a multiple sequence alignment as input
- The task is MSA construction quality improvement itself; the article leaves improving MSA construction and collecting evolutionary information for shallow MSAs as future work
- GPU memory cannot accommodate the 60-layer network, which the authors chose because of GPU memory limitations

## Inputs

- Protein chain sequences and MSAs (training dataset from the Google Drive folder)
- FASTA query sequences (placed in ./dataset/fasta for prediction)
- HHblits-generated MSAs (three iterations, E-value 0.001, 60% coverage)
- Five per-protein features: one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, sequence separations
- CASP11 (105 domains) and CASP12 (55 domains) test datasets

## Outputs

- Trained Feature Added Model checkpoint (via multitask.py)
- Long-range contact prediction accuracies at top L/5, L/2, and L, binned by MSA depth (accuracy_comparison.csv)
- Contact probability matrices for residue pairs

## How to apply

Download the training datasets (protein chains and MSAs) from the DeepECA Google Drive folder, and generate missing MSAs with HHblits using three iterations, E-value 0.001, and 60% coverage. For each protein compute five input features: one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, and sequence separations. Train the network by running 'python multitask.py' with the ADAM optimizer at learning rate 0.0005, batch normalization, and one domain per batch, cropping 200 residues per sequence and sampling up to 30,000 MSA sequences when deeper. Evaluate long-range contact accuracy (Cβ atoms within 8 Å, Cα for glycine) at top L/5, L/2, and L, binning proteins by the number of sequences in the MSA, on CASP11 (105 domains) and CASP12 (55 domains) test sets.

## Related tools

- **DeepECA** (Framework providing the Feature Added Model, multitask.py training script, and pre-trained models/datasets) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates MSAs for training and test proteins (run_hhblits_local.py; three iterations, E-value 0.001, 60% coverage))

## Examples

```
python multitask.py
```

## Evaluation signals

- accuracy_comparison.csv shows the Feature Added Model matching or exceeding the Weighted MSA Model at long-range top L/5, L/2, and L on both CASP11 and CASP12
- Gains concentrate in the shallow-MSA bins (few homologous sequences), reproducing the reported Fig 4 pattern
- Contacts are defined with Cβ atoms within 8 Å (Cα for glycine) and evaluation is stratified by the number of sequences in the MSA
- Training log confirms ADAM optimizer with learning rate 0.0005, batch normalization, one domain per batch, 200-residue crops, and 30,000-sequence MSA sampling cap
- Full training runs in roughly 2–3 days on an NVIDIA TITAN GTX GPU as stated in the README

## Limitations

- Network depth was constrained to 60 layers because of GPU memory limitations
- Training is slow: 2–3 days on an NVIDIA TITAN GTX GPU
- Requires TensorFlow r1.11.0 and Python3; tested only on Ubuntu 16.04
- Gains depend on the number of sequences in the MSA; benefit is greatest for shallow MSAs and may be minimal for very deep MSAs
- Residue pairs with missing Cβ coordinates are marked NaN and excluded from loss calculation

## Evidence

- [discussion] Training hyperparameters: "We trained all these networks using the ADAM optimizer with the learning rate of 0.0005"
- [discussion] Cropping and MSA sampling: "we crop 200 residues from all protein sequences. When the number of sequences in MSAs is greater than 30,000, we sample 30,000 sequences"
- [abstract] Shallow-MSA sequential feature benefit: "for shallow MSAs, adding a few sequential features was useful to increase the prediction accuracy of long-range contacts in our model."
- [discussion] MSA construction parameters: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [readme] Training command and runtime: "If you want to train models originally, set up the database and run the following command.
It takes 2 or 3 days with NVIDIA TITAN GTX GPU.
`python multitask.py`"
