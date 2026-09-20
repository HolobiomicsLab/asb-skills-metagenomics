---
name: tensorflow-1d-cnn-model-training
description: Use when generating custom k-mer detection networks for baseLess from
  corrected raw signal reads — i.e., you have training/test fast5 reads for a known
  species and a k-mer list, and no pre-generated network exists.
when_to_use_negative:
- A pre-generated k-mer detection network for your target 16S k-mers already exists
  (use 'baseLess update_16s_db' instead of retraining)
- You need full basecalling/sequence reconstruction rather than target-detection-only
  analysis on raw signal
- You lack a good reference genome for tombo resquiggle or a large read set of a known
  species to train on
edam_operation: http://edamontology.org/operation_3983
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: baseLess
  role: Provides 'run_production_pipeline' to orchestrate per-k-mer 1D CNN training
    and hyperparameter optimization over the corrected training/test reads
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls the raw MinION reads as a prerequisite for read correction
- name: ont-fast5-api
  role: Splits multi-fast5 outputs into single-read fast5 files required by the pipeline
- name: tombo
  role: Corrects signal-level reads against the reference genome via 'tombo resquiggle'
    to produce training labels/inputs
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:9b8d86fc4b87ab3f554289f289c68ba490f2ecbe2fc4723e121ba99720189ced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_3983
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tensorflow-1d-cnn-model-training

## Summary

Train per-k-mer 1D CNN detectors in TensorFlow whose hyperparameters are optimized with hyperopt's tree-structured Parzen estimator, minimizing loss L=(1-F1)+lambda*p_c/p_max (lambda=0.01) to jointly maximize F1 and shrink network size. Each network outputs the posterior probability that its target k-mer is present in a squiggle segment of raw MinION signal.

## When to use

Use when generating custom k-mer detection networks for baseLess from corrected raw signal reads — i.e., you have training/test fast5 reads for a known species and a k-mer list, and no pre-generated network exists.

## When NOT to use

- A pre-generated k-mer detection network for your target 16S k-mers already exists (use 'baseLess update_16s_db' instead of retraining)
- You need full basecalling/sequence reconstruction rather than target-detection-only analysis on raw signal
- You lack a good reference genome for tombo resquiggle or a large read set of a known species to train on

## Inputs

- Raw MinION reads (fast5), basecalled with guppy
- Single-read fast5 files (split from multi-fast5 via ont-fast5-api)
- tombo-resquiggle corrected signal reads aligned to a reference genome (FASTA)
- Training reads directory and held-out test reads directory
- k-mer list (txt file, one k-mer per line)

## Outputs

- Trained 1D CNN k-mer detection networks exported as .h5 files (one per k-mer)
- Per-network parameter counts and F1 scores on the held-out test reads

## How to apply

First prepare signal-level inputs: basecall raw reads with guppy, split multi-fast5 files into single-read fast5 with ont-fast5-api's single_to_multi_fast5, then correct reads against a reference genome using 'tombo resquiggle'. Assemble one small 1D CNN per k-mer from the provided k-mer list (txt, one k-mer per line) in TensorFlow 2.3. Optimize each network's hyperparameters with hyperopt's tree-structured Parzen estimator over 100 training/evaluation rounds, minimizing loss L=(1-F1)+lambda*p_c/p_max with lambda=0.01, so the objective rewards F1 on held-out reads while penalizing parameter count (p_c/p_max) — this is why the resulting networks stay small enough for single-board computers like the 2GB Jetson Nano. Run training via 'baseLess run_production_pipeline' with --training-reads, --test-reads, --kmer-list and --cores 8, writing the trained networks to the output directory. Export each k-mer detector as a .h5 model and record its parameter count and test F1 score.

## Related tools

- **baseLess** (Provides 'run_production_pipeline' to orchestrate per-k-mer 1D CNN training and hyperparameter optimization over the corrected training/test reads) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls the raw MinION reads as a prerequisite for read correction)
- **ont-fast5-api** (Splits multi-fast5 outputs into single-read fast5 files required by the pipeline)
- **tombo** (Corrects signal-level reads against the reference genome via 'tombo resquiggle' to produce training labels/inputs) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list kmers.txt
```

## Evaluation signals

- The exported .h5 model file loads successfully (e.g., in Keras/TensorFlow)
- Each trained k-mer detector's parameter count is checkable and stays small relative to p_max, reflecting the size penalty in the loss
- The trained k-mer detectors attain a reported F1 score on the held-out test reads
- One network is produced for every k-mer in the provided k-mer list (one k-mer per line)
- Hyperparameter optimization ran over 100 training/evaluation rounds with hyperopt's TPE, minimizing (1-F1)+0.01*p_c/p_max

## Limitations

- Retraining requires a large read set of a known species and a good reference genome; pre-generated networks only cover k-mers salient for 16S detection
- Reads must be corrected with tombo resquiggle prior to usage for training
- Hyperparameter search is capped at 100 rounds, and the lambda=0.01 size penalty trades some F1 for network compactness

## Evidence

- [readme] README: custom network generation workflow: "if you have a large read set of a known species, a good reference genome, and you want to generate k-mer networks yourself, you can do so as follows."
- [readme] README: baseLess run_production_pipeline invocation: "baseLess run_production_pipeline \
  --training-reads path/to/training/reads \
  --test-reads path/to/test/reads \
  --out-dir path/to/networks/directory"
- [readme] README: tombo resquiggle correction step: "tombo resquiggle single/fast5/output/folder reference_genome.fasta"
- [intro] Article: composite of smaller k-mer networks (intro): "The neural network it uses is a composite of many smaller networks, each detecting a different k-mer, thus reconfiguration of baseLess to detect a different sequence amounts to combining different"
- [results] Source card: hyperopt TPE loss design (task_007): [evidence span withheld — anchor status: not_found]
