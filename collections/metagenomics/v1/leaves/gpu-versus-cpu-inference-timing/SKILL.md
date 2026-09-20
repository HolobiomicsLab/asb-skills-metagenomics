---
name: gpu-versus-cpu-inference-timing
description: Use when comparing wall-clock inference/prediction runtime of a DNN contact
  predictor against an ECA baseline (CCMpred) across CPU and GPU on a fixed, pre-computed-MSA
  protein set such as the 150 PSICOV proteins.
when_to_use_negative:
- The goal is accuracy evaluation (contact precision, TMscore) rather than runtime
  benchmarking — this skill times execution only and does not judge prediction quality.
- MSAs have not been pre-computed and HHblits database construction time would dominate
  or confound the timing comparison.
- No CUDA-capable GPU is available and only the CPU-vs-CPU comparison is of interest
  — the GPU arm of the benchmark cannot be executed.
edam_operation: http://edamontology.org/operation_0252
tools:
- name: DeepECA
  role: DNN contact predictor whose inference pipeline (python pipeline.py) is timed
    on CPU and on NVIDIA TITAN GTX GPU
  repo: https://github.com/tomiilab/DeepECA
- name: CCMpred
  role: Fastest existing ECA baseline; run on the same 150 MSAs on CPU and GPU to
    time precision-matrix optimization
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:2572cb00b7e1c382bddf486e377fbebeb6be350ffd4e2ff13be529d622b5043c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CCMpred
  - DeepECA
  edam_operation: http://edamontology.org/operation_0252
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gpu-versus-cpu-inference-timing

## Summary

Benchmarks wall-clock time of DeepECA DNN contact-prediction inference against CCMpred precision-matrix optimization on the 150-protein PSICOV dataset, on both CPU and NVIDIA TITAN GTX GPU, to reproduce the paper's Table 13 runtime ordering.

## When to use

Use when comparing wall-clock inference/prediction runtime of a DNN contact predictor against an ECA baseline (CCMpred) across CPU and GPU on a fixed, pre-computed-MSA protein set such as the 150 PSICOV proteins.

## When NOT to use

- The goal is accuracy evaluation (contact precision, TMscore) rather than runtime benchmarking — this skill times execution only and does not judge prediction quality.
- MSAs have not been pre-computed and HHblits database construction time would dominate or confound the timing comparison.
- No CUDA-capable GPU is available and only the CPU-vs-CPU comparison is of interest — the GPU arm of the benchmark cannot be executed.

## Inputs

- 150-protein PSICOV dataset with pre-computed MSAs (Google Drive folder from the DeepECA authors)
- Pre-trained DeepECA multitask models (Google Drive download)
- FASTA files for the PSICOV target proteins (./dataset/fasta)
- CCMpred executable and its required MSA inputs

## Outputs

- timing_results.csv with four wall-clock runtimes (DeepECA CPU, DeepECA GPU, CCMpred CPU, CCMpred GPU)
- Contact predictions in ./contact_pred and secondary-structure predictions in ./ss_pred from the DeepECA pipeline runs

## How to apply

Download the pre-trained DeepECA models and the 150-protein PSICOV dataset (with pre-computed MSAs) from the authors' Google Drive folder so no HHblits MSA generation is re-timed. Run the DeepECA pipeline (python pipeline.py, TensorFlow r1.11.0) once restricted to CPU and once on the NVIDIA TITAN GTX GPU, recording DNN inference wall-clock time per full run. Then run CCMpred on the same 150 MSAs on CPU and on GPU, timing each complete precision-matrix optimization run. Tabulate the four measurements (DeepECA CPU/GPU, CCMpred CPU/GPU) in timing_results.csv. The rationale is that DNN inference is a single forward pass while CCMpred optimizes a precision matrix per protein, so the acceptance criterion is qualitative reproduction of the ordering DNN-GPU < DNN-CPU < CCMpred-GPU < CCMpred-CPU (Table 13: 2 min vs 10 min for DeepECA, 5 min vs 58 min for CCMpred).

## Related tools

- **DeepECA** (DNN contact predictor whose inference pipeline (python pipeline.py) is timed on CPU and on NVIDIA TITAN GTX GPU) — https://github.com/tomiilab/DeepECA
- **CCMpred** (Fastest existing ECA baseline; run on the same 150 MSAs on CPU and GPU to time precision-matrix optimization)

## Examples

```
python pipeline.py
```

## Evaluation signals

- timing_results.csv contains exactly four rows/entries: DeepECA CPU, DeepECA GPU, CCMpred CPU, CCMpred GPU, all measured on the same 150 PSICOV proteins.
- The qualitative ordering DNN-GPU < DNN-CPU < CCMpred-GPU < CCMpred-CPU is reproduced (paper values: 2 min, 10 min, 5 min, 58 min).
- DeepECA DNN inference completes on the order of minutes, substantially faster than CCMpred's per-protein precision-matrix optimization on both hardware configurations.
- The same pre-computed MSAs are reused for both methods, so no MSA-generation time (HHblits) is included in any measured runtime.
- GPU runtimes are recorded on an NVIDIA TITAN GTX-class device, matching the hardware stated in the paper's setup.

## Limitations

- The acceptance criterion is qualitative (ordering reproduction), not exact numerical agreement; absolute times depend on hardware and software versions.
- The DeepECA models were constrained to 60 layers because of GPU memory limitations, which affects both accuracy and inference cost on smaller GPUs.
- The comparison covers inference only — retraining DeepECA takes 2 or 3 days on an NVIDIA TITAN GTX GPU and is explicitly out of scope for this timing skill.
- Results are specific to the Ubuntu 16.04 / Python3 / numpy / pandas / TensorFlow r1.11.0 environment the authors tested.

## Evidence

- [other] Table 13 reports 10 min (CPU) and 2 min (GPU) for the DNN versus 58 min (CPU) and 5 min (GPU) for CCMpred: [evidence span withheld — anchor status: not_found]
- [methods] Downloading pre-trained models and datasets: "Please download these files from the link below."
- [readme] python pipeline.py prediction command: "To predict by using the multitask model, simply run the following command. The results will be stored in ./contact_pred and ./ss_pred directory."
- [readme] training time on TITAN GTX GPU: "If you want to train models originally, set up the database and run the following command. It takes 2 or 3 days with NVIDIA TITAN GTX GPU."
- [discussion] GPU memory constrained depth: "We used 60 layers because of GPU memory limitations"
