---
name: parameter-count-model-size-control
description: Use when training per-k-mer neural detectors from raw MinION squiggle
  reads and the resulting .h5 models must stay small enough to run in limited memory
  (e.g. baseLess with --mem 256 on a Jetson Nano).
when_to_use_negative:
- You only need to run detection/inference — pre-generated 16S k-mer networks are
  available via 'baseLess update_16s_db' (<1MB), so retraining is unnecessary.
- Your reads are not signal-level fast5 corrected with tombo (e.g. FASTQ-only input)
  — the training pipeline requires tombo-resquiggle corrected reads.
- You need full basecalling or whole-genome analysis; baseLess is explicitly a target-detection-only
  tool.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: baseLess
  role: Provides run_production_pipeline, which trains and exports the per-k-mer 1D
    CNN detection networks as .h5 files
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls raw MinION reads as a prerequisite for read correction and training
- name: ont-fast5-api
  role: single_to_multi_fast5 splits multi-fast5 output into single-read fast5 files
    required by the pipeline
  repo: https://pypi.org/project/ont-fast5-api/
- name: tombo
  role: tombo resquiggle corrects signal-level reads against a reference genome before
    training
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:209a793ac1f3a79778f9e62dd20efe32cb91aa39408b112849505f1c9bddcc1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# parameter-count-model-size-control

## Summary

Trains baseLess k-mer detection 1D CNNs with an explicit model-size penalty so that each per-k-mer network is small enough for single-board computers like the 2GB Jetson Nano. Hyperopt's tree-structured Parzen estimator minimizes L=(1-F1)+lambda*p_c/p_max (lambda=0.01) over 100 rounds, jointly maximizing F1 and shrinking parameter count.

## When to use

Use when training per-k-mer neural detectors from raw MinION squiggle reads and the resulting .h5 models must stay small enough to run in limited memory (e.g. baseLess with --mem 256 on a Jetson Nano).

## When NOT to use

- You only need to run detection/inference — pre-generated 16S k-mer networks are available via 'baseLess update_16s_db' (<1MB), so retraining is unnecessary.
- Your reads are not signal-level fast5 corrected with tombo (e.g. FASTQ-only input) — the training pipeline requires tombo-resquiggle corrected reads.
- You need full basecalling or whole-genome analysis; baseLess is explicitly a target-detection-only tool.

## Inputs

- Training reads: tombo-resquiggle corrected single-read fast5 files from guppy-basecalled MinION runs
- Test reads: held-out corrected fast5 directory for F1 evaluation
- k-mer list: plain txt file with one k-mer per line
- Reference genome FASTA used for tombo resquiggle correction

## Outputs

- Trained per-k-mer 1D CNN models exported as a .h5 file
- Parameter count and F1 score of the trained k-mer detectors on held-out test reads

## How to apply

First basecall raw reads with guppy and split multi-fast5 to single-read fast5 using ont-fast5-api's single_to_multi_fast5, then correct signals against a reference genome with 'tombo resquiggle'. Assemble one small 1D CNN per k-mer from a txt k-mer list (one k-mer per line) in TensorFlow 2.3; each network outputs the posterior probability that its target k-mer is present in a squiggle segment. Optimize hyperparameters with hyperopt's tree-structured Parzen estimator over 100 training/evaluation rounds, minimizing the loss L=(1-F1)+lambda*p_c/p_max with lambda=0.01, so F1 is maximized while the number of network parameters p_c (relative to a max p_max) is penalized. Launch via 'baseLess run_production_pipeline' with --training-reads, --test-reads, --kmer-list and --cores 8; export the trained model as a .h5 file and report its parameter count and F1 score on held-out test reads.

## Related tools

- **baseLess** (Provides run_production_pipeline, which trains and exports the per-k-mer 1D CNN detection networks as .h5 files) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls raw MinION reads as a prerequisite for read correction and training)
- **ont-fast5-api** (single_to_multi_fast5 splits multi-fast5 output into single-read fast5 files required by the pipeline) — https://pypi.org/project/ont-fast5-api/
- **tombo** (tombo resquiggle corrects signal-level reads against a reference genome before training) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list path/to/kmers.txt
```

## Evaluation signals

- The exported .h5 model file loads successfully and its parameter count can be checked.
- Trained k-mer detectors attain a reported F1 score on the held-out test reads.
- Hyperparameter optimization completes the full 100 hyperopt rounds with the loss (1-F1)+0.01*p_c/p_max decreasing/converging.
- Model size stays small enough to satisfy the low-memory deployment envelope (baseLess limited to 256MB on a 2GB Jetson Nano).
- Each k-mer in the provided txt k-mer list has a corresponding trained network in the output directory.

## Limitations

- Optimization takes significant time: k-mer models are generated as needed and this 'will take longer to run (typically <1 hour)' when trained on-the-fly.
- Training requires reads from a known species, a good reference genome, and a large read set; otherwise the pre-generated 16S k-mer networks should be used instead.
- Reads must be signal-level corrected with tombo prior to use for training.
- Deployment hardware is memory-constrained: on a 2GB Jetson Nano, baseLess must be limited to 256MB and the docker container to 512MB.

## Evidence

- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [methods] hyperopt TPE over 100 rounds with parameter-count-penalized loss: [evidence span withheld — anchor status: not_found]
- [readme] README production pipeline command: "baseLess run_production_pipeline \
  --training-reads path/to/training/reads \
  --test-reads path/to/test/reads \
  --out-dir path/to/networks/directory \
  --cores 8 \"
- [readme] README tombo correction step: "Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):
tombo resquiggle single/fast5/output/folder reference_genome.fasta"
