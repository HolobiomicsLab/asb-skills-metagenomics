---
name: f1-score-model-evaluation
description: Use when training or tuning baseLess 1D CNN k-mer detectors on raw MinION
  squiggle signal and you need a held-out-test-read F1 score, or when comparing hyperparameter
  candidates under a size-penalized loss.
when_to_use_negative:
- Reads are already basecalled FASTQ and a full sequence-level classifier (not raw-signal
  k-mer detection) is required — F1 evaluation here applies only to baseLess k-mer
  detectors on squiggle segments
- The task is inference with a pre-trained model (baseLess run_inference); no hyperparameter
  search or per-round F1 evaluation is involved
- The k-mer networks were generated outside baseLess and no labeled test-read squiggle
  segments exist to compute a meaningful F1
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: baseLess
  role: Trains and evaluates the individual 1D CNN k-mer detectors via 'run_production_pipeline',
    scoring candidates with the F1-based size-penalized loss
  repo: https://github.com/cvdelannoy/baseLess
- name: tombo
  role: resquiggle corrects signal-level reads against a reference genome, producing
    the labeled squiggle segments on which F1 is computed
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:d9730e8654f88919eca8fe8efd4420234b72dc46368542f5d6c4aa441344b7a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - tombo
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# f1-score-model-evaluation

## Summary

Evaluate baseLess k-mer detection CNNs on held-out test reads using the F1 score, and jointly score candidate architectures with the loss L=(1-F1)+lambda*p_c/p_max (lambda=0.01) so that both classification quality and network size are optimized.

## When to use

Use when training or tuning baseLess 1D CNN k-mer detectors on raw MinION squiggle signal and you need a held-out-test-read F1 score, or when comparing hyperparameter candidates under a size-penalized loss.

## When NOT to use

- Reads are already basecalled FASTQ and a full sequence-level classifier (not raw-signal k-mer detection) is required — F1 evaluation here applies only to baseLess k-mer detectors on squiggle segments
- The task is inference with a pre-trained model (baseLess run_inference); no hyperparameter search or per-round F1 evaluation is involved
- The k-mer networks were generated outside baseLess and no labeled test-read squiggle segments exist to compute a meaningful F1

## Inputs

- Trained or candidate 1D CNN k-mer detection networks (.h5)
- Test reads: tombo-resquiggled, single-read fast5 files (squiggle segments with k-mer labels)
- Hyperparameter configurations sampled by hyperopt's TPE over 100 rounds

## Outputs

- F1 score per k-mer detection network on held-out test reads
- Loss value L=(1-F1)+lambda*p_c/p_max per candidate (lambda=0.01)
- Parameter count of each candidate network (for the size penalty and reporting)

## How to apply

After running 'baseLess run_production_pipeline' with --training-reads, --test-reads and --kmer-list, evaluate each candidate k-mer detection network (a TensorFlow 1D CNN that outputs the posterior probability of its target k-mer being present in a squiggle segment) on the held-out test reads and compute the F1 score. During hyperopt's 100 training/evaluation rounds with the tree-structured Parzen estimator, minimize the loss L=(1-F1)+lambda*p_c/p_max with lambda=0.01, where p_c/p_max penalizes the number of network parameters — this jointly maximizes F1 and shrinks the network so it can run on low-powered hardware like the 2GB Jetson Nano. Report the final .h5 model's parameter count together with its F1 score on the test reads as validation of the trained detector.

## Related tools

- **baseLess** (Trains and evaluates the individual 1D CNN k-mer detectors via 'run_production_pipeline', scoring candidates with the F1-based size-penalized loss) — https://github.com/cvdelannoy/baseLess
- **tombo** (resquiggle corrects signal-level reads against a reference genome, producing the labeled squiggle segments on which F1 is computed) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list kmer_list.txt
```

## Evaluation signals

- The exported .h5 model file loads successfully and its parameter count is checkable
- The trained k-mer detector attains an F1 score on the held-out test reads as reported by the pipeline
- Across hyperopt rounds the loss L=(1-F1)+0.01*p_c/p_max decreases, indicating simultaneous F1 gain and parameter-count reduction
- Networks remain small enough to run under the low-memory targets (baseLess limited to 256MB on a 2GB Jetson Nano)

## Limitations

- The loss trades F1 against network size (lambda=0.01), so maximal per-k-mer F1 is not pursued in isolation
- F1 is measured on k-mer presence in squiggle segments of resquiggled reads, not on end-to-end species classification; downstream read_detection still applies a >66% k-mer fraction threshold
- Validated k-mer networks are attuned to 16S sequence; other targets require generating networks from sufficient reads of a known species plus a good reference genome

## Evidence

- [full_text] hyperopt TPE loss and F1 objective: [evidence span withheld — anchor status: not_found]
- [full_text] network output definition: [evidence span withheld — anchor status: not_found]
- [readme] composite design rationale: "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer"
- [methods] read_detection threshold downstream of k-mer scoring: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
- [readme] production pipeline invocation: "baseLess run_production_pipeline \
  --training-reads path/to/training/reads \
  --test-reads path/to/test/reads"
