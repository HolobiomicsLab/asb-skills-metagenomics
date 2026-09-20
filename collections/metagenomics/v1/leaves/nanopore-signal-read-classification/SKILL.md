---
name: nanopore-signal-read-classification
description: Use when input is raw ONT fast5 signal data (amplified or metagenomic
  16S) and you need per-read target-species detection on inexpensive hardware without
  basecalling, e.g. MinION as a species detector.
when_to_use_negative:
- You need full basecalled sequences rather than target-detection-only output — baseLess
  reduces the MinION to a species detector and does not produce base calls.
- Samples are pure and you want genome-wide detection with larger genomes — abundance
  mode ('--model-type abundance') is the tested route, not read_detection.
- Your target is not tuned for 16S-style sequence context — read detection mode 'has
  been tested for and attuned to 16S sequence at the moment'.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: baseLess
  role: Compiles the read_detection k-mer model and runs neural-network inference
    directly on fast5 electrical signals, classifying reads as target when >66% of
    total k-mers are found
  repo: https://github.com/cvdelannoy/baseLess
- name: mamba
  role: Faster drop-in replacement for conda used to install baseLess from bioconda
- name: conda
  role: Alternative installer for baseLess from bioconda
- name: ont-fast5-api
  role: Splits multi-fast5 output into single-read fast5 files prior to inference
  repo: https://pypi.org/project/ont-fast5-api/
- name: guppy
  role: Basecaller used only when generating your own k-mer detection networks (not
    needed when using pre-generated 16S models)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad017
    title: 'baseLess: lightweight detection of sequences in raw MinION data'
  source_content_hash: sha256:5218ea230dc51ca5cb4bf9c74ce8822fe98e1d0c625823e2c37ffaf27f5f4c81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cvdelannoy/baseLess
  tools:
  - baseLess
  - conda
  - guppy
  - mamba
  - ont-fast5-api
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# nanopore-signal-read-classification

## Summary

Classify individual MinION reads as originating from a target species by detecting k-mers directly in the raw electrical signal with baseLess read_detection mode, avoiding basecalling entirely. Benchmarked on a 21-species 16S rRNA mock community (HM-278D) under stratified 5-fold cross-validation.

## When to use

Use when input is raw ONT fast5 signal data (amplified or metagenomic 16S) and you need per-read target-species detection on inexpensive hardware without basecalling, e.g. MinION as a species detector.

## When NOT to use

- You need full basecalled sequences rather than target-detection-only output — baseLess reduces the MinION to a species detector and does not produce base calls.
- Samples are pure and you want genome-wide detection with larger genomes — abundance mode ('--model-type abundance') is the tested route, not read_detection.
- Your target is not tuned for 16S-style sequence context — read detection mode 'has been tested for and attuned to 16S sequence at the moment'.

## Inputs

- Raw MinION fast5 signal files (one read per fast5, or multi-fast5 split with ont-fast5-api single_to_multi_fast5)
- Target species 16S sequences in FASTA (target.fasta)
- Pre-generated 16S k-mer models fetched via 'baseLess update_16s_db' (<1MB), or a compiled read_detection .h5 model
- Stratified 5-fold cross-validation splits of reads per species (for benchmarking)

## Outputs

- Per-read target/non-target classifications per fold from run_inference
- Pooled per-species accuracy and F1 scores (e.g. per_species_accuracy_f1.csv)
- In read detection mode, fast5 files containing the target are retained in the output directory

## How to apply

Install baseLess from bioconda (mamba install -c bioconda -c conda-forge -c cvdelannoy baseless, or conda as a drop-in replacement) and either fetch pre-generated 16S k-mer models (<1MB) with 'baseLess update_16s_db' or compile a read-detection model from a target FASTA via 'baseLess compile_model --model-type read_detection --target-fasta target.fasta --out-model your_model_name.h5'. For evaluation, split reads into stratified 5-fold cross-validation folds and for each fold run 'baseLess run_inference --fast5-in <fold dir> --out-dir <out> --model your_model_name.h5 --inference-mode once'. baseLess uses an array of small neural networks, each detecting a fixed-size subsequence (k-mer) of the target sequence directly from the electrical signal; a read is classified as target when it contains a minimum fraction of total k-mers (>66% by default). Pool per-fold read classifications across folds and compute per-species accuracy and F1 scores, e.g. written to per_species_accuracy_f1.csv.

## Related tools

- **baseLess** (Compiles the read_detection k-mer model and runs neural-network inference directly on fast5 electrical signals, classifying reads as target when >66% of total k-mers are found) — https://github.com/cvdelannoy/baseLess
- **mamba** (Faster drop-in replacement for conda used to install baseLess from bioconda)
- **conda** (Alternative installer for baseLess from bioconda)
- **ont-fast5-api** (Splits multi-fast5 output into single-read fast5 files prior to inference) — https://pypi.org/project/ont-fast5-api/
- **guppy** (Basecaller used only when generating your own k-mer detection networks (not needed when using pre-generated 16S models))

## Examples

```
baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5 --inference-mode once
```

## Evaluation signals

- Per-species read-detection accuracy reproduces the 16S mock community (HM-278D) benchmark: >95% accuracy for most species, with Helicobacter pylori as the known exception.
- Average F1 across species is near 0.54 under pooled stratified 5-fold cross-validation, with H. pylori as the low-F1 exception.
- In read detection mode, only fast5s containing the target are retained in the output directory.
- Inference terminates after analysing all reads when '--inference-mode once' is supplied rather than running indefinitely.
- The per-species table is complete: one accuracy and F1 row per species in the mock community.

## Limitations

- Read detection mode 'has been tested for and attuned to 16S sequence at the moment', so non-16S targets may not be supported out of the box.
- Generating your own k-mer networks requires basecalled reads with guppy, single-fast5 conversion, and tombo resquiggle correction against a reference genome, plus a good reference and large read set.
- baseLess runs indefinitely until manually stopped (ctrl-c) unless '--inference-mode once' is provided; in read detection mode reads are removed from the output flow unless '--copy-reads' is added.
- On low-memory hardware (2GB Jetson Nano), docker and baseLess must be memory-limited separately (e.g. -m 512m and --mem 256) and the GUI switched off.

## Evidence

- [abstract] BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal.: "BaseLess makes use of an array of small neural networks, each of which efficiently detects a fixed-size subsequence of the target sequence directly from the electrical signal."
- [methods] download pre-generated k-mer models into the default directory of the package (current size <1MB): baseLess update_16s_db: "download pre-generated k-mer models into the default directory of the package (current size <1MB): baseLess update_16s_db"
- [readme] If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species.: "If a read contains a minimum fraction of total k-mers (>66%, by default), it is considered to originate from the target species."
- [readme] baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5: "baseLess run_inference --fast5-in directory/of/fast5s --out-dir your/output/directory --model your_model_name.h5"
- [readme] In read detection mode, baseLess will only retain fast5s that contain your target. In abundance mode, no reads will be retained.: "In read detection mode, baseLess will only retain fast5s that contain your target. In abundance mode, no reads will be retained."
