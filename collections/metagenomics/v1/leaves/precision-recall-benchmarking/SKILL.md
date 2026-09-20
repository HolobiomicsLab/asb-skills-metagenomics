---
name: precision-recall-benchmarking
description: Use when you have sequencing reads from communities with known ground-truth
  taxonomic profiles (e.g. ten HBC mock communities) and must quantify which classifier's
  read-level assignments and abundance estimates are most precise and sensitive.
when_to_use_negative:
- Input reads come from real environmental or host samples with no ground-truth abundance
  profile — precision/recall cannot be computed without a known truth set
- The task is training or architecture selection for DeepMicrobes itself (k-mer length
  ablation, hyperparameter search) rather than head-to-head tool benchmarking
- The data are long reads or non-gut error profiles; the paper notes retraining might
  be required for strongly different read lengths and error profiles
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: DeepMicrobes
  role: Deep-learning classifier under test; TFRecord conversion, prediction, and
    taxonomic profile generation on mock communities
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA-based comparison classifier run on the same mock-community reads
- name: Kaiju
  role: Protein-based comparison classifier; run with a custom database of predicted
    protein sequences from the human gut bacterial repertoire
- name: R
  role: Aggregation of metrics and figure generation for benchmarking
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:ebd8992c541e3bb51f27ee8f0348b23ae118a2e6693e44b1414d9152f584d48c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MicrobeLab/DeepMicrobes
  repo_url: https://github.com/MicrobeLab/DeepMicrobes
  tools:
  - DeepMicrobes
  - Kaiju
  - Kraken
  - R
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# precision-recall-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark taxonomic classifiers (DeepMicrobes, Kraken, Kraken 2, Centrifuge, CLARK, CLARK-S, Kaiju, DIAMOND-MEGAN, BLAST-MEGAN) on mock communities by computing read-level precision/recall, genus/species profiles, L2 abundance distances, and per-tool read classification rates against ground-truth profiles.

## When to use

Use when you have sequencing reads from communities with known ground-truth taxonomic profiles (e.g. ten HBC mock communities) and must quantify which classifier's read-level assignments and abundance estimates are most precise and sensitive.

## When NOT to use

- Input reads come from real environmental or host samples with no ground-truth abundance profile — precision/recall cannot be computed without a known truth set
- The task is training or architecture selection for DeepMicrobes itself (k-mer length ablation, hyperparameter search) rather than head-to-head tool benchmarking
- The data are long reads or non-gut error profiles; the paper notes retraining might be required for strongly different read lengths and error profiles

## Inputs

- fastq/fasta sequencing reads from the ten mock communities generated from gastrointestinal bacterial isolates (ENA ERP105624, ERP012217)
- TFRecord-converted metagenome datasets for the DeepMicrobes pipeline
- True genus profiles of the ten mock communities (Table S4) and true relative abundances (Table S5)
- Output profiles and read-level assignments from each classifier (Kraken, Kraken 2, Centrifuge, CLARK, CLARK-S, Kaiju, DIAMOND-MEGAN, BLAST-MEGAN, DeepMicrobes)

## Outputs

- benchmark_results.csv with per-tool genus-level precision/recall at 0.01% and 0.0001% abundance cutoffs
- Per-tool read classification rates (fraction of reads assigned)
- L2 distances between predicted and ground-truth abundance profiles
- Benchmark figures rendered by the R scripts from DeepMicrobes-data/scripts

## How to apply

Download the ten mock communities built from HBC gastrointestinal isolate genomes (ENA ERP105624 and ERP012217) and convert each fastq/fasta dataset to TFRecord with the DeepMicrobes conversion utility before running prediction. Run every classifier on the same reads, then use the DeepMicrobes profiling pipeline to produce per-species/genus abundance profiles and read-level assignments for each tool. Compute community-level precision and recall at 0.01% and 0.0001% abundance cutoffs, L2 distances between predicted and true abundance profiles, and each tool's read classification rate. Aggregate the metrics with the R benchmarking scripts from MicrobeLab/DeepMicrobes-data (which also contains Table S4 true genus profiles and Table S13 precision/recall under different confidence scores) and write the comparison to benchmark_results.csv. Validate against the paper's numbers: DeepMicrobes' genus model classified on average 89.40% of reads versus Kaiju's 68.27%.

## Related tools

- **DeepMicrobes** (Deep-learning classifier under test; TFRecord conversion, prediction, and taxonomic profile generation on mock communities) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA-based comparison classifier run on the same mock-community reads)
- **Kaiju** (Protein-based comparison classifier; run with a custom database of predicted protein sequences from the human gut bacterial repertoire)
- **R** (Aggregation of metrics and figure generation for benchmarking) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- DeepMicrobes' genus-model read classification rate on the ten mock communities matches the reported 89.40% (versus Kaiju's 68.27%)
- Precision/recall curves reproduce the Figure 3 values at both the 0.01% and 0.0001% abundance cutoffs
- Genus- and species-level profiles per tool on the mock communities match the abundance profiles distributed in DeepMicrobes-data
- Read-level precision/recall on the 3,269 held-out MAGs correspond to Supplementary Tables S9 and S10
- Confidence-threshold sensitivity is reported, comparable to Supplementary Table S13 (precision/recall under different confidence scores)

## Limitations

- Adding new species to DeepMicrobes requires retraining the entire deep neural network, so benchmark results are tied to the trained repertoire of 2,505 gut species
- The 12-mer length used is optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (e.g. 6-12 bp)
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)

## Evidence

- [supplementary] Read-level precision of different model architectures on variable-length reads: [evidence span withheld — anchor status: external_doc]
