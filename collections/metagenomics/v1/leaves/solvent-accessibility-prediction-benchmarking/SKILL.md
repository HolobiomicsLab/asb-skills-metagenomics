---
name: solvent-accessibility-prediction-benchmarking
description: Use when evaluating ASA/solvent-accessibility predictors on CASP target
  domains and you need a head-to-head comparison of a multi-task DNN against RaptorX-Property
  and SCRATCH-1D.
when_to_use_negative:
- The task is 3D tertiary structure modeling rather than 1D per-residue property prediction
  — the article treats tertiary construction (CONFOLD) as a separate downstream step
- Only shallow MSAs are available and the goal is long-range contact accuracy specifically
  — the article notes improving MSA construction for shallow MSAs was left as future
  work
edam_operation: http://edamontology.org/operation_0303
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: DeepECA
  role: Multi-task DNN producing ASA (and secondary structure) predictions from HHblits
    MSAs; predictions stored via pipeline.py in ./ss_pred
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs (three iterations, E-value 0.001, 60% coverage, UniProt20_2016)
    that are the input feature source for the multitask model
- name: RaptorX-Property
  role: Baseline ASA prediction method for comparison on CASP11/CASP12 (accessed via
    web server)
- name: SCRATCH-1D
  role: Baseline ASA prediction method for accuracy comparison on CASP11/CASP12
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:e4ed98ede2343ee4753cefbfb2dc540b143f6479deb3f61767f693aa40b6c095
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - DeepECA
  - HHblits
  - RaptorX-Property
  - SCRATCH-1D
  edam_operation: http://edamontology.org/operation_0303
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Solvent-accessibility-prediction-benchmarking

## Summary

Benchmark solvent-accessible surface area (ASA) predictions from the DeepECA multi-task network against RaptorX-Property and SCRATCH-1D on CASP11 (105 domains) and CASP12 (55 domains), verifying that joint training with secondary structure and contact tasks yields top ASA accuracy.

## When to use

Use when evaluating ASA/solvent-accessibility predictors on CASP target domains and you need a head-to-head comparison of a multi-task DNN against RaptorX-Property and SCRATCH-1D.

## When NOT to use

- The task is 3D tertiary structure modeling rather than 1D per-residue property prediction — the article treats tertiary construction (CONFOLD) as a separate downstream step
- Only shallow MSAs are available and the goal is long-range contact accuracy specifically — the article notes improving MSA construction for shallow MSAs was left as future work

## Inputs

- FASTA files of target protein domains (placed in ./dataset/fasta)
- Pre-trained multitask models and datasets from the DeepECA Google Drive folder
- HHblits MSAs generated against UniProt20_2016 (3 iterations, E-value 0.001, 60% coverage)
- CASP11 (105 domains) and CASP12 (55 domains) test sets with DSSP-derived accessibility labels

## Outputs

- ASA prediction files in ./ss_pred
- Secondary-structure prediction files in ./ss_pred
- Contact prediction files in ./contact_pred
- Benchmark accuracy comparison table (ASA, Table 6) versus RaptorX-Property and SCRATCH-1D

## How to apply

Run the DeepECA multi-task model via pipeline.py so ASA predictions are written to ./ss_pred alongside secondary-structure outputs, after generating MSAs with HHblits (three iterations, E-value 0.001, 60% coverage, UniProt20_2016). The multitask network uses a shared 20-residual-block trunk with task-specific 10-block heads, trained with ADAM (learning rate 0.0005), batch normalization, one domain per batch, 200-residue cropping, and sampling of up to 30,000 MSA sequences. Evaluate the predictions on the CASP11 and CASP12 domain sets and compare ASA accuracy (Table 6) against RaptorX-Property and SCRATCH-1D, alongside secondary-structure accuracy (Tables 4-5). Ground-truth secondary structure and accessibility states come from DSSP, so predictions must cover residues for which DSSP assignments exist. Validation requires the multi-task model to achieve the highest ASA prediction accuracy among the three methods on both CASP11 and CASP12.

## Related tools

- **DeepECA** (Multi-task DNN producing ASA (and secondary structure) predictions from HHblits MSAs; predictions stored via pipeline.py in ./ss_pred) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs (three iterations, E-value 0.001, 60% coverage, UniProt20_2016) that are the input feature source for the multitask model)
- **RaptorX-Property** (Baseline ASA prediction method for comparison on CASP11/CASP12 (accessed via web server))
- **SCRATCH-1D** (Baseline ASA prediction method for accuracy comparison on CASP11/CASP12)

## Examples

```
python pipeline.py
```

## Evaluation signals

- The multitask model achieves the highest ASA prediction accuracy among DeepECA, RaptorX-Property, and SCRATCH-1D on both CASP11 (105 domains) and CASP12 (55 domains)
- Prediction files exist in ./ss_pred (and ./contact_pred) after running pipeline.py
- Multitask model also outperforms RaptorX-Property and SCRATCH-1D in secondary structure prediction accuracy (Tables 4-5)
- DSSP secondary structure states can be assigned for ≥80% of residues of training proteins used in the comparison
- Training converges with the stated configuration: ADAM, learning rate 0.0005, 200-residue cropping, up to 30,000 sampled MSA sequences, ~2-3 days on an NVIDIA TITAN GTX GPU

## Limitations

- GPU memory limitations constrained the network depth to 60 layers
- RaptorX-Property comparison results were missing for 3 of 105 CASP11 domains and 1 of 55 CASP12 domains due to server errors, since no local execution file was available
- Training the multitask network requires 2-3 days on an NVIDIA TITAN GTX GPU and TensorFlow r1.11.0 (tested on Ubuntu 16.04)

## Evidence

- [results] Multitask model outperforms baselines on ASA: [evidence span withheld — anchor status: not_found]
- [methods] Validation criteria for the benchmark: [evidence span withheld — anchor status: not_found]
- [readme] Prediction output locations from README: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
- [discussion] Training configuration: "We trained all these networks using the ADAM optimizer with the learning rate of 0.0005"
- [abstract] Multi-task motivation for higher accuracy: "we expanded our model to a multi-task model to achieve higher accuracy by incorporating predictions of secondary structures and solvent-accessible surface areas."
