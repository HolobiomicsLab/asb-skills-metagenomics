---
name: kmer-signal-detection-network-design
description: Use when you have raw MinION fast5 reads, a reference genome, and a k-mer
  list, and need to generate custom k-mer detection networks for a sequence absent
  from baseLess's pre-generated 16S set.
when_to_use_negative:
- Your k-mers are covered by baseLess's pre-generated 16S k-mer network set (fetchable
  via 'baseLess update_16s_db', <1MB)
- You already have basecalled FASTQ data and want standard sequence analysis rather
  than signal-level k-mer detection
- You do not have a known-species read set plus a good reference genome, which the
  README states are prerequisites for generating k-mer networks yourself
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3168
tools:
- name: baseLess
  role: Runs the production pipeline ('baseLess run_production_pipeline') that trains
    the per-k-mer CNNs with hyperopt optimization and exports .h5 models
  repo: https://github.com/cvdelannoy/baseLess
- name: guppy
  role: Basecalls raw MinION reads as the first preparation step before network training
- name: ont-fast5-api
  role: Splits multi-fast5 files into single-read fast5 files via single_to_multi_fast5
  repo: https://pypi.org/project/ont-fast5-api/
- name: tombo
  role: Corrects signal-level reads against a reference genome via 'tombo resquiggle'
    to provide training labels
  repo: https://anaconda.org/bioconda/ont-tombo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:ed12c5e7cda4d26e59bd5955b475761ba36e5a2f018776aec111c5bbc010bc99
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - guppy
  - ont-fast5-api
  - tombo
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmer-signal-detection-network-design

## Summary

Design and train small 1D CNN k-mer detection networks for baseLess, each outputting the posterior probability that its target k-mer is present in a raw MinION squiggle segment, with hyperparameters optimized by hyperopt's tree-structured Parzen estimator over 100 rounds.

## When to use

Use when you have raw MinION fast5 reads, a reference genome, and a k-mer list, and need to generate custom k-mer detection networks for a sequence absent from baseLess's pre-generated 16S set.

## When NOT to use

- Your k-mers are covered by baseLess's pre-generated 16S k-mer network set (fetchable via 'baseLess update_16s_db', <1MB)
- You already have basecalled FASTQ data and want standard sequence analysis rather than signal-level k-mer detection
- You do not have a known-species read set plus a good reference genome, which the README states are prerequisites for generating k-mer networks yourself

## Inputs

- Raw MinION reads (fast5, possibly multi-fast5)
- Reference genome FASTA for tombo resquiggle
- K-mer list (txt file, one k-mer per line)
- Training reads directory (tombo-corrected)
- Test reads directory (tombo-corrected)

## Outputs

- Trained k-mer detection networks exported as .h5 files in the output directory
- Parameter count and F1 score per trained k-mer detector

## How to apply

Basecall raw reads with guppy and split multi-fast5 files into single-read fast5 files using ont-fast5-api's single_to_multi_fast5, then correct signal-level reads against a reference genome with 'tombo resquiggle'. Assemble one small 1D CNN per k-mer from the provided k-mer list (txt, one k-mer per line) in TensorFlow 2.3, and optimize each network's hyperparameters with hyperopt's tree-structured Parzen estimator over 100 training/evaluation rounds, minimizing the loss L=(1-F1)+lambda*p_c/p_max with lambda=0.01 to jointly maximize F1 and shrink parameter count. Run training via 'baseLess run_production_pipeline' with --training-reads, --test-reads, --kmer-list and --cores 8, writing networks to the output directory. Export each trained detector as a .h5 file; these networks are the building blocks that baseLess later combines for abundance or read_detection models.

## Related tools

- **baseLess** (Runs the production pipeline ('baseLess run_production_pipeline') that trains the per-k-mer CNNs with hyperopt optimization and exports .h5 models) — https://github.com/cvdelannoy/baseLess
- **guppy** (Basecalls raw MinION reads as the first preparation step before network training)
- **ont-fast5-api** (Splits multi-fast5 files into single-read fast5 files via single_to_multi_fast5) — https://pypi.org/project/ont-fast5-api/
- **tombo** (Corrects signal-level reads against a reference genome via 'tombo resquiggle' to provide training labels) — https://anaconda.org/bioconda/ont-tombo

## Examples

```
baseLess run_production_pipeline --training-reads path/to/training/reads --test-reads path/to/test/reads --out-dir path/to/networks/directory --cores 8 --kmer-list kmers.txt
```

## Evaluation signals

- The exported .h5 model file loads successfully as a TensorFlow/Keras model
- The trained k-mer detector's parameter count is checkable and reflects the size penalty lambda*p_c/p_max with lambda=0.01
- Each k-mer detector attains a reported F1 score on the held-out test reads after 100 hyperopt rounds
- Corrected training reads exist (tombo resquiggle completed) and single-read fast5 files were produced by single_to_multi_fast5
- Networks are written to the --out-dir directory as one small network per k-mer in the provided k-mer list

## Limitations

- Hyperparameter optimization requires 100 training/evaluation rounds per k-mer; the README notes k-mer model generation 'will take longer to run (typically <1 hour)' when training on the fly
- Generating k-mer networks yourself requires 'a large read set of a known species, a good reference genome'
- baseLess is tuned to and tested for 16S sequence detection; read_detection mode thresholds (e.g. >66% k-mer fraction) are attuned to 16S
- Deployment constraints: designed for low-powered hardware such as the 2GB Jetson Nano with baseLess limited to 256MB memory

## Evidence

- [full_text] individual k-mer detectors are 1D CNNs in Tensorflow 2.3 whose hyperparameters are tuned over 100 training/evaluation rounds with hyperopt's tree-structured Parzen estimator: [evidence span withheld — anchor status: not_found]
- [full_text] minimizing loss L=(1-F1)+lambda*p_c/p_max (lambda=0.01) to jointly maximize F1 and shrink network size: [evidence span withheld — anchor status: not_found]
- [methods] Basecall your reads with guppy if you haven't yet.: "Basecall your reads with guppy if you haven't yet. If reads were output as multi-fast5, split them using from the ont-fast5-api"
- [readme] Correct your reads with tombo: tombo resquiggle single/fast5/output/folder reference_genome.fasta: "Correct your reads with [tombo](https://anaconda.org/bioconda/ont-tombo):
``ntombo resquiggle single/fast5/output/folder reference_genome.fasta
"
- [readme] if you have a large read set of a known species, a good reference genome, and you want to generate k-mer networks yourself, you can do so as follows: "however if you have a large read set of a known species, a good reference genome, and you want to generate k-mer networks yourself, you can do so as follows"
- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
