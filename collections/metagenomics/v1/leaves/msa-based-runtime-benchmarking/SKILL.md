---
name: msa-based-runtime-benchmarking
description: Use when comparing inference wall-clock time of a DNN contact-prediction
  pipeline against a classical ECA baseline (e.g., CCMpred precision-matrix optimization)
  over the same pre-computed MSAs on matched CPU and GPU hardware.
when_to_use_negative:
- The goal is to compare contact-prediction accuracy (e.g., CASP contact metrics)
  rather than wall-clock runtime — accuracy benchmarking uses different protocols
  and datasets.
- Input proteins lack pre-computed MSAs and you do not have HHblits set up; the timing
  comparison assumes identical MSAs for both methods.
- No GPU is available and you need an exact GPU/CPU comparison rather than the CPU-only
  pair of measurements.
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0081
tools:
- name: DeepECA
  role: DNN-based end-to-end contact predictor whose inference runtime (CPU and GPU)
    is benchmarked; invoked via python pipeline.py
  repo: https://github.com/tomiilab/DeepECA
- name: CCMpred
  role: Fastest existing ECA baseline; precision-matrix optimization timed on the
    same 150 PSICOV MSAs on CPU and GPU
- name: HHblits
  role: Generates MSAs (via run_hhblits_local.py with a configured path) if new fasta
    inputs are used, ensuring identical MSA inputs for both methods
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:64161df5000f4757275ad6c7e6a8817d399f6706aab00af8816b8c0979341dd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CCMpred
  - DeepECA
  - HHblits
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# msa-based-runtime-benchmarking

## Summary

Benchmark wall-clock runtime of a DNN-based end-to-end contact predictor (DeepECA) against the fastest existing ECA method (CCMpred) on an identical MSA-backed protein set, with and without GPU acceleration, to establish a qualitative runtime ordering.

## When to use

Use when comparing inference wall-clock time of a DNN contact-prediction pipeline against a classical ECA baseline (e.g., CCMpred precision-matrix optimization) over the same pre-computed MSAs on matched CPU and GPU hardware.

## When NOT to use

- The goal is to compare contact-prediction accuracy (e.g., CASP contact metrics) rather than wall-clock runtime — accuracy benchmarking uses different protocols and datasets.
- Input proteins lack pre-computed MSAs and you do not have HHblits set up; the timing comparison assumes identical MSAs for both methods.
- No GPU is available and you need an exact GPU/CPU comparison rather than the CPU-only pair of measurements.

## Inputs

- 150-protein PSICOV dataset with pre-computed MSAs (from the authors' Google Drive folder)
- Pre-trained DeepECA multitask model files
- FASTA files for target proteins (placed in ./dataset/fasta)
- CCMpred executable run on the same 150 MSAs

## Outputs

- timing_results.csv with four measured runtimes (DeepECA CPU/GPU, CCMpred CPU/GPU)
- DeepECA contact predictions in ./contact_pred and secondary-structure predictions in ./ss_pred
- Qualitative runtime ordering comparison against Table 13

## How to apply

Download the pre-trained DeepECA models and the 150-protein PSICOV dataset with pre-computed MSAs from the authors' Google Drive folder, then run the DeepECA prediction pipeline (python pipeline.py) once restricted to CPU and once on the NVIDIA TITAN GTX GPU, recording DNN inference wall-clock time for each. Run CCMpred on the same 150 MSAs on CPU and on GPU, timing each complete run, so all four configurations consume identical inputs. Tabulate the four measured runtimes (DeepECA CPU/GPU, CCMpred CPU/GPU) into timing_results.csv. The acceptance criterion is qualitative: reproduce the paper's Table 13 ordering DNN-GPU (2 min) < DNN-CPU (10 min) < CCMpred-GPU (5 min) < CCMpred-CPU (58 min), i.e., DNN inference completes much faster than CCMpred on both CPU and GPU.

## Related tools

- **DeepECA** (DNN-based end-to-end contact predictor whose inference runtime (CPU and GPU) is benchmarked; invoked via python pipeline.py) — https://github.com/tomiilab/DeepECA
- **CCMpred** (Fastest existing ECA baseline; precision-matrix optimization timed on the same 150 PSICOV MSAs on CPU and GPU)
- **HHblits** (Generates MSAs (via run_hhblits_local.py with a configured path) if new fasta inputs are used, ensuring identical MSA inputs for both methods) — https://github.com/tomiilab/DeepECA

## Examples

```
python pipeline.py
```

## Evaluation signals

- timing_results.csv contains exactly four rows/configurations: DeepECA-CPU, DeepECA-GPU, CCMpred-CPU, CCMpred-GPU, all timed on the same 150 MSAs.
- The measured ordering reproduces Table 13: DNN-GPU < DNN-CPU < CCMpred-GPU < CCMpred-CPU.
- DeepECA DNN inference completes on the order of minutes (reported: 2 min GPU, 10 min CPU) versus CCMpred (5 min GPU, 58 min CPU).
- Both methods were given identical MSA inputs (same 150-protein PSICOV set), not different MSA pipelines.
- Timing is wall-clock of the complete run (DNN inference vs CCMpred precision-matrix optimization), not model training time.

## Limitations

- The acceptance criterion is qualitative (ordering), not exact timing reproduction — absolute times will vary with hardware and software versions.
- GPU memory limitations constrained the DeepECA network depth to 60 layers, and training (not inference) takes 2-3 days on a NVIDIA TITAN GTX GPU — only inference is benchmarked here.
- The pipeline was tested on Ubuntu 16.04 with TensorFlow r1.11.0, numpy, and pandas; other environments may require additional setup.
- RaptorX-Contact-style server-only tools cannot be timed locally, so this skill only covers methods with local executables (e.g., DeepECA, CCMpred).

## Evidence

- [methods] Pre-trained models (for prediction) and datasets (for training) are needed.: "Pre-trained models (for prediction) and datasets (for training) are needed."
- [readme] If you want to train models originally, set up the database and run the following command. It takes 2 or 3 days with NVIDIA TITAN GTX GPU.: "If you want to train models originally, set up the database and run the following command. It takes 2 or 3 days with NVIDIA TITAN GTX GPU."
- [readme] To predict by using the multitask model, simply run the following command.: "To predict by using the multitask model, simply run the following command."
- [discussion] We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations: "We prepared MSAs for proteins in both our original and test datasets using HHblits [35] with three iterations"
