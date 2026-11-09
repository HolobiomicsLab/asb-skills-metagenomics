---
name: mock-community-species-benchmarking
description: Use when a target-detection tool must be quantitatively validated on
  a defined mock community, e.g. per-species read classification accuracy/F1 for 16S
  MinION reads under stratified 5-fold cross-validation.
when_to_use_negative:
- Your goal is abundance estimation of a target genome in a pure sample rather than
  per-read species classification — baseLess abundance mode, not read-detection benchmarking,
  is appropriate
- Your input is already basecalled FASTQ/FASTA sequence data — this skill evaluates
  detection directly on raw MinION electrical signal (fast5), which is the point of
  baseLess
- You need genome-level taxonomy profiling of a complex metagenome rather than targeted
  detection of a defined set of mock-community species
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: baseLess
  role: Compiles the read_detection k-mer model and runs inference on fast5 files,
    classifying each read as target when >66% of k-mers are detected
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: Faster drop-in replacement for conda to install baseLess from bioconda
- name: conda
  role: Alternative installer for baseLess from bioconda channels
- name: ont-fast5-api
  role: Splits multi-fast5 read files into single-read fast5 files before inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:ebb95490bb54e7a7298f20a588260185156f46984f072a33204f14fad2f67137
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - conda
  - mamba
  - ont-fast5-api
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mock-community-species-benchmarking

## Summary

Benchmark per-species read-detection accuracy of a sequence detector (baseLess read_detection mode) on a defined mock community (16S rRNA HM-278D, 21 species) using stratified 5-fold cross-validation, reporting per-species accuracy and F1 scores.

## When to use

Use when a target-detection tool must be quantitatively validated on a defined mock community, e.g. per-species read classification accuracy/F1 for 16S MinION reads under stratified 5-fold cross-validation.

## When NOT to use

- Your goal is abundance estimation of a target genome in a pure sample rather than per-read species classification — baseLess abundance mode, not read-detection benchmarking, is appropriate
- Your input is already basecalled FASTQ/FASTA sequence data — this skill evaluates detection directly on raw MinION electrical signal (fast5), which is the point of baseLess
- You need genome-level taxonomy profiling of a complex metagenome rather than targeted detection of a defined set of mock-community species

## Inputs

- MinION fast5 read files from a 21-species 16S rRNA mock community (HM-278D), organized per species and per CV fold
- Target FASTA containing 16S sequences of the community species (or pre-generated 16S k-mer models via update_16s_db)
- baseLess read_detection model (.h5)

## Outputs

- per_species_accuracy_f1.csv: per-species accuracy and F1 score table
- Per-fold read classification labels (target vs non-target) pooled across 5 folds

## How to apply

Install baseLess from bioconda via mamba (or conda) and fetch the pre-generated 16S k-mer models (<1MB) with 'baseLess update_16s_db', or compile a read-detection model with 'baseLess compile_model --model-type read_detection --target-fasta target.fasta --out-model your_model_name.h5'. For each fold of the stratified 5-fold cross-validation over the 21-species mock community (HM-278D) fast5 reads, run 'baseLess run_inference --fast5-in <fold read directory> --out-dir <output> --model your_model_name.h5 --inference-mode once', where a read is classified as target when it contains >66% (default threshold) of the total k-mers. Pool per-fold read classifications across all 5 folds, then compute per-species accuracy and F1 scores and write them to per_species_accuracy_f1.csv. The benchmark is accepted when per-species accuracy reproduces the reported result: >95% accuracy for most species, mean F1 near 0.54, with Helicobacter pylori as the known low-F1 exception.

## Related tools

- **baseLess** (Compiles the read_detection k-mer model and runs inference on fast5 files, classifying each read as target when >66% of k-mers are detected) — https://github.com/cvdelannoy/baseLess
- **mamba** (Faster drop-in replacement for conda to install baseLess from bioconda)
- **conda** (Alternative installer for baseLess from bioconda channels)
- **ont-fast5-api** (Splits multi-fast5 read files into single-read fast5 files before inference)

## Examples

```
baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5 --inference-mode once
```

## Evaluation signals

- Per-species accuracy exceeds 95% for most of the 21 mock-community species
- Mean F1 across species is near 0.54, with Helicobacter pylori as the only/known low-F1 exception
- All 5 CV folds complete and pooled classifications cover every read of the stratified splits
- Read-classification threshold applied consistently: a read is called target only when it contains >66% (default) of total k-mers
- Output table per_species_accuracy_f1.csv contains accuracy and F1 columns for each species

## Limitations

- Read detection mode has been tested for and attuned to 16S sequence; it is not necessarily restricted to 16S but was only validated there
- H. pylori is a known low-F1 exception even though accuracy is high (>95% for most species, average F1 of 0.54)
- The set of pre-generated k-mer networks is limited (<1MB, 16S-focused); generating new k-mer networks requires corrected training reads (guppy basecalling, single_to_multi_fast5 splitting, tombo resquiggle)

## Evidence

- [results] In read detection mode on the 16S mock community, baseLess consistently identified its target reads with more than 95% accuracy and an average F1 score of 0.54, with Helicobacter pylori as the only exception.: [evidence span withheld — anchor status: not_found]
- [methods] k-mer fraction threshold: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
- [methods] update_16s_db: "download pre-generated k-mer models into the default directory of the package (current size <1MB): baseLess update_16s_db"
- [readme] install via mamba/conda: "Install baseLess from conda, with the much faster [mamba](https://anaconda.org/conda-forge/mamba) drop-in replacement if you have it:"
- [readme] attuned to 16S: "Though not necessarily restricted to any sequence, this has been tested for and attuned to 16S sequence at the moment."
