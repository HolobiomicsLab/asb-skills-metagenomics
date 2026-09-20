---
name: msa-derived-feature-engineering
description: Use when building DNN input features for protein contact prediction from
  MSAs, particularly when MSAs are shallow (few homologous sequences) and raw coevolution
  signal alone underperforms on long-range contacts.
when_to_use_negative:
- The goal is de novo contact prediction without a DNN using existing ECA methods
  such as PSICOV, plmDCA, GREMLIN, or CCMpred, which consume covariance/precision
  matrices rather than engineered feature stacks
- The MSA is deep and the Weighted MSA Model already achieves target accuracy, so
  the added sequential features provide no expected benefit
- The task is tertiary structure construction from already-predicted contacts (use
  CONFOLD), not feature engineering from the MSA
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: DeepECA
  role: Framework whose Feature Added Model consumes the engineered MSA/sequence features;
    trained via multitask.py and evaluated for long-range contact accuracy
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs from which entropy, mutual information, and PSSM features
    are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:78de18bb02f29648c94ded737369141439dce53e73ab6cbdbc091115ee4d6588
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# msa-derived-feature-engineering

## Summary

Engineer five per-protein input features (one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, and sequence separations) from a multiple sequence alignment so a deep network (DeepECA Feature Added Model) can improve long-range contact prediction, especially for shallow MSAs.

## When to use

Use when building DNN input features for protein contact prediction from MSAs, particularly when MSAs are shallow (few homologous sequences) and raw coevolution signal alone underperforms on long-range contacts.

## When NOT to use

- The goal is de novo contact prediction without a DNN using existing ECA methods such as PSICOV, plmDCA, GREMLIN, or CCMpred, which consume covariance/precision matrices rather than engineered feature stacks
- The MSA is deep and the Weighted MSA Model already achieves target accuracy, so the added sequential features provide no expected benefit
- The task is tertiary structure construction from already-predicted contacts (use CONFOLD), not feature engineering from the MSA

## Inputs

- Multiple sequence alignments (MSAs) per protein, generated with HHblits (three iterations, E-value 0.001, 60% coverage) or downloaded from the provided Google Drive folder
- Protein query sequences in FASTA format
- Training datasets of protein chains and MSAs (Google Drive folder)
- CASP11 (105 domains) and CASP12 (55 domains) test sets

## Outputs

- Five per-protein feature tensors: one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, and sequence separations
- Trained Feature Added Model (via python multitask.py)
- Long-range contact predictions at top L/5, L/2, and L with accuracies binned by MSA depth (accuracy_comparison.csv)

## How to apply

Generate MSAs with HHblits using three iterations, E-value 0.001, and 60% coverage where MSAs are not supplied. For each protein compute five features: one-hot encoding of the query sequence, the position-specific scoring matrix (PSSM), weighted-MSA entropy, mutual information, and residue sequence separations. Concatenate these as inputs to the Feature Added Model and train with the ADAM optimizer (learning rate 0.0005, batch normalization, one domain per batch), cropping 200 residues per sequence and sampling up to 30,000 MSA sequences per protein. The rationale is that for shallow MSAs, adding features derived from the sequence itself and the MSA increases long-range contact prediction accuracy over the Weighted MSA Model alone.

## Related tools

- **DeepECA** (Framework whose Feature Added Model consumes the engineered MSA/sequence features; trained via multitask.py and evaluated for long-range contact accuracy) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs from which entropy, mutual information, and PSSM features are computed)

## Examples

```
python multitask.py
```

## Evaluation signals

- The accuracy_comparison.csv must show the Feature Added Model matching or exceeding the Weighted MSA Model on long-range contacts at top L/5, L/2, and L
- Accuracy gains should be largest for proteins whose MSAs contain only a small number of homologous sequences (shallow-MSA bins), reproducing the reported Fig 4 gains
- Contacts must be defined as Cβ atom pairs (Cα for glycine) within 8 Å, with residue pairs lacking Cβ coordinates marked NaN and excluded from the loss
- Training must use the documented settings: ADAM optimizer, learning rate 0.0005, batch normalization, one domain per batch, 200-residue crops, and ≤30,000 sampled MSA sequences per protein
- Results must replicate on both independent test sets: CASP11 (105 domains) and CASP12 (55 domains)

## Limitations

- GPU memory limitation constrained the network depth to 60 layers
- Improving MSA construction and collecting sufficient evolutionary information for shallow MSAs is left as future work, so gains still depend on MSA depth
- Training takes 2–3 days on an NVIDIA TITAN GTX GPU and requires TensorFlow r1.11.0 on Ubuntu 16.04

## Evidence

- [abstract] for shallow MSAs, adding a few sequential features was useful to increase the prediction accuracy of long-range contacts in our model.: "for shallow MSAs, adding a few sequential features was useful to increase the prediction accuracy of long-range contacts in our model."
- [discussion] We trained all these networks using the ADAM optimizer with the learning rate of 0.0005: "We trained all these networks using the ADAM optimizer with the learning rate of 0.0005"
- [discussion] we crop 200 residues from all protein sequences. When the number of sequences in MSAs is greater than 30,000, we sample 30,000 sequences: "we crop 200 residues from all protein sequences. When the number of sequences in MSAs is greater than 30,000, we sample 30,000 sequences"
- [discussion] We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [readme] Please download these files from the link below.: "Pre-trained models (for prediction) and datasets (for training) are needed. Please download these files from the link below."
