---
name: wall-clock-performance-tabulation
description: Use when comparing computational cost of two contact-prediction pipelines
  on the same pre-computed MSAs, needing measured runtimes per hardware configuration
  (CPU vs GPU) rather than accuracy metrics.
when_to_use_negative:
- The goal is prediction accuracy evaluation (precision, TMscore of CONFOLD-built
  models) rather than computational cost — use an accuracy benchmarking skill instead.
- The input MSAs are not shared between the compared methods (e.g. RaptorX-Contact
  server predictions use different MSAs), making wall-clock comparisons non-commensurable.
- No GPU or reproducible hardware environment is available and only a single hardware
  configuration can be timed, so the CPU/GPU ordering cannot be tested.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: DeepECA
  role: DNN-based contact prediction pipeline (pipeline.py) whose inference wall-clock
    time is measured on CPU and on NVIDIA TITAN GTX GPU
  repo: https://github.com/tomiilab/DeepECA
- name: CCMpred
  role: Fastest existing ECA baseline; precision-matrix optimization timed on the
    same 150 PSICOV MSAs on CPU and GPU
- name: HHblits
  role: Required by the DeepECA repository for MSA generation if MSAs must be rebuilt
    rather than reused
  repo: https://github.com/tomiilab/DeepECA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:d3248c95034bce7a9eb85cd28b9a6e37cf4f82ebfefeb3ab8c56702e6e4aeb15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CCMpred
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# wall-clock-performance-tabulation

## Summary

Tabulates wall-clock runtimes of DeepECA DNN inference versus CCMpred precision-matrix optimization on the 150-protein PSICOV dataset, on CPU and GPU, to benchmark end-to-end deep learning against the fastest existing ECA method.

## When to use

Use when comparing computational cost of two contact-prediction pipelines on the same pre-computed MSAs, needing measured runtimes per hardware configuration (CPU vs GPU) rather than accuracy metrics.

## When NOT to use

- The goal is prediction accuracy evaluation (precision, TMscore of CONFOLD-built models) rather than computational cost — use an accuracy benchmarking skill instead.
- The input MSAs are not shared between the compared methods (e.g. RaptorX-Contact server predictions use different MSAs), making wall-clock comparisons non-commensurable.
- No GPU or reproducible hardware environment is available and only a single hardware configuration can be timed, so the CPU/GPU ordering cannot be tested.

## Inputs

- 150-protein PSICOV dataset with pre-computed MSAs (Google Drive folder 1_Ys7dZ2P0uF52nvXZkkMeIv0rbqhUMPj)
- Pre-trained DeepECA models for prediction
- FASTA files placed in ./dataset/fasta
- CCMpred executable configured for CPU and GPU execution

## Outputs

- timing_results.csv with four measured runtimes (DeepECA CPU, DeepECA GPU, CCMpred CPU, CCMpred GPU)
- Qualitative runtime ordering per protein set (DNN-GPU < DNN-CPU < CCMpred-GPU < CCMpred-CPU)

## How to apply

Download the pre-trained DeepECA models and the 150-protein PSICOV dataset (with pre-computed MSAs) from the authors' Google Drive folder linked in the README. Run the DeepECA prediction pipeline (python pipeline.py, multitask model, TensorFlow r1.11.0) on the PSICOV proteins with an NVIDIA TITAN GTX GPU and record DNN inference wall-clock time, then re-run restricted to CPU. Run CCMpred on the same 150 MSAs on CPU and on GPU, timing each complete run. Tabulate the four runtimes into timing_results.csv and check that the paper's Table 13 ordering reproduces: DNN-GPU (2 min) < DNN-CPU (10 min) < CCMpred-GPU (5 min) < CCMpred-CPU (58 min). Acceptance is qualitative reproduction of the ordering — DNN inference completes much faster than CCMpred on both CPU and GPU.

## Related tools

- **DeepECA** (DNN-based contact prediction pipeline (pipeline.py) whose inference wall-clock time is measured on CPU and on NVIDIA TITAN GTX GPU) — https://github.com/tomiilab/DeepECA
- **CCMpred** (Fastest existing ECA baseline; precision-matrix optimization timed on the same 150 PSICOV MSAs on CPU and GPU)
- **HHblits** (Required by the DeepECA repository for MSA generation if MSAs must be rebuilt rather than reused) — https://github.com/tomiilab/DeepECA

## Examples

```
python pipeline.py  # DeepECA multitask prediction on PSICOV MSAs; record wall-clock time, then repeat with CUDA_VISIBLE_DEVICES='' for CPU
```

## Evaluation signals

- Runtimes reproduce Table 13 ordering: DNN-GPU < DNN-CPU < CCMpred-GPU < CCMpred-CPU.
- DeepECA DNN inference completes on the order of minutes (~10 min CPU, ~2 min GPU) versus CCMpred (~58 min CPU, ~5 min GPU) on 150 PSICOV proteins.
- All 150 PSICOV proteins are processed identically for both methods so timings are computed on the same MSA set.
- Both methods are timed end-to-end on each hardware configuration; the same dataset and pre-computed MSAs are reused from the Google Drive download.

## Limitations

- GPU memory limitations constrained the network depth to 60 layers, so deeper architectures may not be timeable on the same hardware.
- The comparison is on the 150-protein PSICOV dataset only; timing does not transfer to CASP11/CASP12 domains or other datasets.
- Acceptance is qualitative (ordering, not exact times), and RaptorX-Contact cannot be included because no local execution file was available and its MSAs differ.

## Evidence

- [readme] Please download these files from the link below.: "Pre-trained models (for prediction) and datasets (for training) are needed. Please download these files from the link below."
- [readme] python pipeline.py invocation: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
- [results] Table 13 runtimes: [evidence span withheld — anchor status: not_found]
- [discussion] TITAN GTX GPU hardware: "It takes 2 or 3 days with NVIDIA TITAN GTX GPU."
