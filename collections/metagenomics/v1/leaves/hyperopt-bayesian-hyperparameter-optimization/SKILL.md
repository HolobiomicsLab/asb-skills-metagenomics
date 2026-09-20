---
name: hyperopt-bayesian-hyperparameter-optimization
description: Use when training small neural networks (e.g., per-k-mer 1D CNNs on MinION
  squiggle segments) and you need Bayesian hyperparameter search that trades off predictive
  F1 against model size rather than grid/random search.
when_to_use_negative:
- A pre-generated set of k-mer detection networks already covers your target (e.g.,
  salient 16S k-mers via 'baseLess update_16s_db') — training from scratch is unnecessary
- You need basecalled sequences rather than target-detection-only posterior probabilities;
  use a basecaller such as guppy instead
- You lack reads of a known species plus a good reference genome, the prerequisites
  the README states for generating k-mer networks yourself
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: baseLess
  role: Provides run_production_pipeline, which hosts the hyperopt-driven hyperparameter
    search and training of per-k-mer CNNs
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls raw reads prior to network training data preparation
- name: ont-fast5-api
  role: Splits multi-fast5 files into single-read fast5 files via single_to_multi_fast5
    before correction
- name: tombo
  role: Corrects signal-level reads against a reference genome via resquiggle to label
    squiggle segments
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:c8ac61db770f326bbde9aaa6da2ac0964b90826689f31e445fcf9981bc77638f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hyperopt-bayesian-hyperparameter-optimization

## Summary

Tunes the hyperparameters of individual k-mer detection 1D CNNs (TensorFlow 2.3) in the baseLess pipeline using hyperopt's tree-structured Parzen estimator, jointly maximizing F1 and shrinking network size over 100 training/evaluation rounds.

## When to use

Use when training small neural networks (e.g., per-k-mer 1D CNNs on MinION squiggle segments) and you need Bayesian hyperparameter search that trades off predictive F1 against model size rather than grid/random search.

## When NOT to use

- A pre-generated set of k-mer detection networks already covers your target (e.g., salient 16S k-mers via 'baseLess update_16s_db') — training from scratch is unnecessary
- You need basecalled sequences rather than target-detection-only posterior probabilities; use a basecaller such as guppy instead
- You lack reads of a known species plus a good reference genome, the prerequisites the README states for generating k-mer networks yourself

## Inputs

- Corrected signal-level reads (single-read fast5, tombo resquiggle output) split into training and test sets
- K-mer list as txt file, one k-mer per line
- Reference genome FASTA used for tombo resquiggle correction

## Outputs

- One trained k-mer detection 1D CNN per k-mer, exported as .h5 files in the output directory
- Parameter count and F1 score per trained k-mer detector on held-out test reads

## How to apply

Prepare signal-level reads by basecalling with guppy, splitting multi-fast5 to single-read fast5 with ont-fast5-api, and correcting signals with 'tombo resquiggle' against a reference genome. Assemble one small 1D CNN per k-mer from the provided k-mer list (txt, one k-mer per line) in TensorFlow, each outputting the posterior probability that its target k-mer is present in a squiggle segment. Run hyperopt's tree-structured Parzen estimator over 100 training/evaluation rounds, minimizing the loss L=(1-F1)+lambda*p_c/p_max with lambda=0.01, where p_c is the candidate network's parameter count and p_max a reference size — this penalizes parameter count so the search favors lightweight networks that still reach high F1. Drive the whole procedure via 'baseLess run_production_pipeline' with --training-reads, --test-reads, --kmer-list and --cores 8, writing the selected networks to the output directory as .h5 files. Judge success by checking that the .h5 model loads, its parameter count is inspectable, and the trained detectors attain a reported F1 on the held-out test reads.

## Related tools

- **baseLess** (Provides run_production_pipeline, which hosts the hyperopt-driven hyperparameter search and training of per-k-mer CNNs) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls raw reads prior to network training data preparation)
- **ont-fast5-api** (Splits multi-fast5 files into single-read fast5 files via single_to_multi_fast5 before correction)
- **tombo** (Corrects signal-level reads against a reference genome via resquiggle to label squiggle segments) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list kmers.txt
```

## Evaluation signals

- The exported .h5 model file for each k-mer detector loads successfully in TensorFlow/Keras
- The parameter count of each trained network is checkable and reduced by the p_c/p_max penalty term
- Trained k-mer detectors attain an F1 score on held-out test reads as reported by the pipeline
- The hyperopt search completes the full 100 training/evaluation rounds with the loss (1-F1)+0.01*p_c/p_max
- Each k-mer in the input k-mer list yields a corresponding trained network in the output directory

## Limitations

- The optimization objective couples F1 with a parameter-count penalty (lambda=0.01), so maximally accurate but larger networks may be disfavored
- Training on-the-fly k-mer models requires reads corrected with tombo and takes longer to run (typically <1 hour per the README)
- baseLess runs on memory-constrained hardware (e.g., 2GB Jetson Nano with baseLess limited to 256MB), constraining network size

## Evidence

- [other] individual k-mer detectors are 1D CNNs in Tensorflow 2.3 whose hyperparameters are tuned over 100 training/evaluation rounds with hyperopt's tree-structured Parzen estimator: [evidence span withheld — anchor status: not_found]
- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the"
- [readme] Correct your reads with tombo ... baseLess run_production_pipeline --training-reads ... --kmer-list: "tombo resquiggle single/fast5/output/folder reference_genome.fasta"
- [readme] README production pipeline invocation: "baseLess run_production_pipeline \
  --training-reads path/to/training/reads \
  --test-reads path/to/test/reads \
  --out-dir path/to/networks/directory"
