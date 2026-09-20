---
name: protein-contact-accuracy-evaluation-by-msa-depth
description: Use when comparing contact-prediction models on CASP target domains and
  you need to know whether accuracy gains at top L/5, L/2, and L depend on how shallow
  (few homologous sequences) the input MSA is.
when_to_use_negative:
- The goal is a single aggregate accuracy number with no need to resolve gains as
  a function of MSA depth (the binning is the point of this skill)
- Evaluating prediction of non-long-range contacts or metrics other than top L/5,
  L/2, L contact accuracy (this skill targets long-range contacts under the 8 Å Cβ
  definition)
- The model under test does not consume MSA-derived inputs, so depth-stratified gains
  cannot be interpreted
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_2275
- http://edamontology.org/topic_0623
tools:
- name: DeepECA
  role: Framework whose Feature Added and Weighted MSA models are trained (python
    multitask.py) and whose long-range contact predictions are evaluated by MSA depth
  repo: github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs whose depth (number of homologous sequences) is the stratification
    variable; run with three iterations, E-value 0.001, 60% coverage via run_hhblits_local.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:f3e738fc33b60150d407cd5ef9759b139fbcc1a3dc868583930e38e7c847b50c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_2275
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# protein-contact-accuracy-evaluation-by-msa-depth

## Summary

Evaluates whether adding sequential features (one-hot query, PSSM, weighted-MSA entropy, mutual information, sequence separations) to the Weighted MSA Model improves long-range contact prediction accuracy, stratifying results by MSA depth (number of homologous sequences) on CASP11 and CASP12 test sets.

## When to use

Use when comparing contact-prediction models on CASP target domains and you need to know whether accuracy gains at top L/5, L/2, and L depend on how shallow (few homologous sequences) the input MSA is.

## When NOT to use

- The goal is a single aggregate accuracy number with no need to resolve gains as a function of MSA depth (the binning is the point of this skill)
- Evaluating prediction of non-long-range contacts or metrics other than top L/5, L/2, L contact accuracy (this skill targets long-range contacts under the 8 Å Cβ definition)
- The model under test does not consume MSA-derived inputs, so depth-stratified gains cannot be interpreted

## Inputs

- Protein chains with associated MSAs (training datasets from the DeepECA Google Drive folder)
- CASP11 test set (105 domains)
- CASP12 test set (55 domains)
- FASTA files of query sequences (./dataset/fasta)
- Five per-protein input features: one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, sequence separations

## Outputs

- Long-range contact prediction accuracy at top L/5, L/2, and L per MSA-depth bin
- Accuracy comparison table (accuracy_comparison.csv) contrasting Feature Added Model vs Weighted MSA Model
- Fig-4-style gain profiles showing depth-dependent accuracy improvements

## How to apply

Generate MSAs with HHblits (three iterations, E-value 0.001, 60% coverage) for each protein, and compute the five input features: one-hot query sequence, PSSM, weighted-MSA entropy, mutual information, and sequence separations. Train the Feature Added Model with 'python multitask.py' using the ADAM optimizer (learning rate 0.0005, batch normalization, one domain per batch), cropping 200 residues per sequence and sampling up to 30,000 MSA sequences per protein. Define contacts as Cβ atom pairs within 8 Å (Cα for glycine), evaluate long-range contact prediction accuracy at top L/5, L/2, and L, and bin proteins by the number of homologous sequences in their MSA. Repeat the evaluation on the CASP11 (105 domains) and CASP12 (55 domains) test sets and record an accuracy comparison showing the Feature Added Model matching or exceeding the Weighted MSA Model, with the largest gains for shallow MSAs.

## Related tools

- **DeepECA** (Framework whose Feature Added and Weighted MSA models are trained (python multitask.py) and whose long-range contact predictions are evaluated by MSA depth) — github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs whose depth (number of homologous sequences) is the stratification variable; run with three iterations, E-value 0.001, 60% coverage via run_hhblits_local.py)

## Examples

```
python multitask.py
```

## Evaluation signals

- accuracy_comparison.csv shows the Feature Added Model matching or exceeding the Weighted MSA Model at long-range top L/5, L/2, and L
- Largest gains appear in bins with few homologous sequences (shallow MSAs), reproducing the reported Fig 4 pattern
- Contact definition verified: Cβ atoms (Cα for glycine) within 8 Å
- Evaluation reproduced on both CASP11 (105 domains) and CASP12 (55 domains) test sets
- Training used ADAM with learning rate 0.0005, 200-residue crops, and ≤30,000 sampled MSA sequences per protein

## Limitations

- For shallow MSAs, adding features is only noted as 'useful' to improve accuracy; improving MSA construction and collecting sufficient evolutionary information from wider sequence spaces is left as future work
- GPU memory limitations constrained the network to 60 layers
- Training the original model takes 2–3 days on an NVIDIA TITAN GTX GPU and requires downloading pre-trained models and datasets plus a local HHblits setup

## Evidence

- [abstract] shallow MSA gains: "for shallow MSAs, adding a few sequential features was useful to increase the prediction accuracy of long-range contacts in our model."
- [discussion] training protocol: "We trained all these networks using the ADAM optimizer with the learning rate of 0.0005"
- [discussion] crop and sampling: "we crop 200 residues from all protein sequences. When the number of sequences in MSAs is greater than 30,000, we sample 30,000 sequences"
- [discussion] MSA construction: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
- [discussion] contact definition: "we defined a contact if the distance of Cβ atoms of the residue pair was less than 8 Å"
- [readme] training command: "If you want to train models originally, set up the database and run the following command."
