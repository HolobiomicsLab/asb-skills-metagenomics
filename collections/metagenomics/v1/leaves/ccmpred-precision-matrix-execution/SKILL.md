---
name: ccmpred-precision-matrix-execution
description: 'Use when you need to benchmark contact-prediction runtime: you hold
  pre-computed MSAs for the 150 PSICOV proteins and must time CCMpred precision-matrix
  optimization against a DNN-based predictor on identical inputs.'
when_to_use_negative:
- Your goal is contact prediction accuracy benchmarking (e.g., TMscore of CONFOLD-built
  3D models on CASP11/CASP12 domains) rather than wall-clock runtime comparison on
  PSICOV.
- You cannot use the authors' pre-computed MSAs and pre-trained models — regenerated
  MSAs or models would make the timing comparison non-comparable to Table 13.
- You need to train DeepECA models from scratch (python multitask.py, 2–3 days on
  a TITAN GTX GPU) — that is a separate training workflow, not this timing-execution
  skill.
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0082
- http://edamontology.org/topic_3316
tools:
- name: CCMpred
  role: Pseudolikelihood ECA program whose precision-matrix optimization is executed
    and timed on the same 150 PSICOV MSAs (CPU and GPU) as the slower comparator in
    the runtime benchmark.
- name: DeepECA
  role: End-to-end DNN contact predictor whose inference pipeline (python pipeline.py)
    is timed as the faster method in the CPU/GPU runtime comparison.
  repo: https://github.com/tomiilab/DeepECA
- name: HHblits
  role: Generates the MSAs that both CCMpred and DeepECA consume as input; pre-computed
    MSAs are provided in the authors' dataset download.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-019-3190-x
    title: 'DeepECA: an end-to-end learning framework for protein contact prediction
      from a multiple sequence alignment'
  source_content_hash: sha256:27f76179ea8f1495d4a70716a74da500aa518134c56b7f74ec77ab2b151ee1f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tomiilab/DeepECA
  tools:
  - CCMpred
  - DeepECA
  - HHblits
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0082
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ccmpred-precision-matrix-execution

## Summary

Executes CCMpred, the fastest existing ECA (precision-matrix) method, on the 150-protein PSICOV dataset with pre-computed HHblits MSAs, on both CPU and GPU, and records wall-clock times to benchmark against DeepECA DNN inference (paper Table 13: DNN 10 min CPU / 2 min GPU vs CCMpred 58 min CPU / 5 min GPU).

## When to use

Use when you need to benchmark contact-prediction runtime: you hold pre-computed MSAs for the 150 PSICOV proteins and must time CCMpred precision-matrix optimization against a DNN-based predictor on identical inputs.

## When NOT to use

- Your goal is contact prediction accuracy benchmarking (e.g., TMscore of CONFOLD-built 3D models on CASP11/CASP12 domains) rather than wall-clock runtime comparison on PSICOV.
- You cannot use the authors' pre-computed MSAs and pre-trained models — regenerated MSAs or models would make the timing comparison non-comparable to Table 13.
- You need to train DeepECA models from scratch (python multitask.py, 2–3 days on a TITAN GTX GPU) — that is a separate training workflow, not this timing-execution skill.

## Inputs

- Pre-computed HHblits MSAs for the 150 proteins in the PSICOV dataset
- DeepECA pre-trained models and datasets from the authors' Google Drive folder
- FASTA files placed in ./dataset/fasta (for running on original targets)
- NVIDIA TITAN GTX GPU (for the GPU timing condition; CPU-only also supported)

## Outputs

- timing_results.csv with the four measured runtimes (DeepECA CPU/GPU, CCMpred CPU/GPU)
- CCMpred precision-matrix-derived contact predictions for the 150 PSICOV proteins
- DeepECA predictions stored in ./contact_pred and ./ss_pred directories

## How to apply

Download the authors' pre-trained models and the 150-protein PSICOV dataset with pre-computed MSAs from the Google Drive folder linked in the README. Run the DeepECA prediction pipeline (python pipeline.py) twice — once on the NVIDIA TITAN GTX GPU and once restricted to CPU — timing the DNN inference wall-clock for each. Then run CCMpred on the same 150 MSAs on CPU and on GPU, timing each complete precision-matrix optimization run. Tabulate the four measured runtimes (DeepECA CPU/GPU, CCMpred CPU/GPU) into timing_results.csv. Acceptance is qualitative reproduction of Table 13: DNN inference on the order of minutes completes much faster than CCMpred on both CPU (10 vs 58 min) and GPU (2 vs 5 min).

## Related tools

- **CCMpred** (Pseudolikelihood ECA program whose precision-matrix optimization is executed and timed on the same 150 PSICOV MSAs (CPU and GPU) as the slower comparator in the runtime benchmark.)
- **DeepECA** (End-to-end DNN contact predictor whose inference pipeline (python pipeline.py) is timed as the faster method in the CPU/GPU runtime comparison.) — https://github.com/tomiilab/DeepECA
- **HHblits** (Generates the MSAs that both CCMpred and DeepECA consume as input; pre-computed MSAs are provided in the authors' dataset download.)

## Examples

```
python pipeline.py
```

## Evaluation signals

- timing_results.csv contains exactly four rows (DeepECA CPU/GPU, CCMpred CPU/GPU) for all 150 PSICOV proteins.
- Reproduced ordering shows DeepECA DNN inference substantially faster than CCMpred on both hardware conditions (10 vs 58 min CPU; 2 vs 5 min GPU in Table 13).
- The same pre-computed MSAs are fed to both methods — any MSA regeneration invalidates the runtime comparison.
- GPU runs use the NVIDIA TITAN GTX GPU specified in the paper's setup.
- DeepECA predictions land in ./contact_pred and ./ss_pred directories as the README specifies.

## Limitations

- Absolute runtimes are hardware-specific (NVIDIA TITAN GTX); they transfer only qualitatively (ordering), not numerically, to other machines.
- The paper's DNN was limited to 60 layers by GPU memory, which is a constraint inherited by the timing comparison on GPU.
- The comparison isolates inference/optimization time — it does not include upstream HHblits MSA generation or downstream 3D model construction.

## Evidence

- [intro] CCMpred is one of the major pseudolikelihood ECA programs named by the article: "Major programs using this method are EVFold [5], plmDCA [11], GREMLIN [7], and CCMpred [13]."
- [readme] Pre-trained models and datasets must be downloaded before prediction runs: "Pre-trained models (for prediction) and datasets (for training) are needed.
Please download these files from the link below."
- [discussion] GPU memory constrained the network depth, a factor in the GPU timing condition: "We used 60 layers because of GPU memory limitations"
- [readme] The prediction pipeline stores contact and secondary-structure outputs in fixed directories: "The results will be stored in ./contact_pred and ./ss_pred directory."
