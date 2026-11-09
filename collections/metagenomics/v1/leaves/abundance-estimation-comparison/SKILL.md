---
name: abundance-estimation-comparison
description: Use when comparing a metagenomic read classifier's genus-level sensitivity
  and abundance profiles against multiple baselines on mock communities with known
  ground-truth abundances.
when_to_use_negative:
- Input reads come from a real clinical/environmental metagenome with no ground-truth
  community composition, so precision/recall against a true profile cannot be computed
- Reads have strongly different lengths and error profiles (e.g. long-read sequencing),
  where the authors state retraining might be required
- The target organisms are novel uncultured species absent from the training repertoire,
  since adding new species requires retraining the entire deep neural network
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: DeepMicrobes
  role: Deep learning classifier under benchmark; converts reads to TFRecord, predicts
    on GPUs, and generates taxonomic profiles
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kaiju
  role: Protein-based baseline classifier run on a custom database of predicted protein
    sequences from the complete bacterial repertoire of the human gut
- name: Kraken
  role: k-mer/LCA-based baseline classifier for genus-level read comparison
- name: R
  role: Aggregates precision/recall, L2 distances, and classification rates into the
    final benchmark tables and figures
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:f97dc2e85910a6af872b459117fb4f15b28626297aa7a6206a38c032eadac3b0
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# abundance-estimation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark DeepMicrobes' genus-level read classification and abundance estimation against eight state-of-the-art taxonomic classifiers (Kraken, Kraken 2, Centrifuge, CLARK, CLARK-S, Kaiju, DIAMOND-MEGAN, BLAST-MEGAN) on ten HBC mock communities with known ground-truth genus profiles.

## When to use

Use when comparing a metagenomic read classifier's genus-level sensitivity and abundance profiles against multiple baselines on mock communities with known ground-truth abundances.

## When NOT to use

- Input reads come from a real clinical/environmental metagenome with no ground-truth community composition, so precision/recall against a true profile cannot be computed
- Reads have strongly different lengths and error profiles (e.g. long-read sequencing), where the authors state retraining might be required
- The target organisms are novel uncultured species absent from the training repertoire, since adding new species requires retraining the entire deep neural network

## Inputs

- fastq/fasta reads of ten HBC mock communities (ENA ERP105624, ERP012217)
- Pre-trained DeepMicrobes genus model (Embed + LSTM + Attention, 12-mer vocabulary)
- True genus profiles of the ten mock communities (Table S4)
- Command lines for the eight benchmarked classifiers from the DeepMicrobes-data repository

## Outputs

- Per-tool genus-level taxonomic profiles (relative abundances and read-level assignments) for each mock community
- Community-level precision and recall at 0.01% and 0.0001% abundance cutoffs
- L2 distances between predicted and ground-truth abundance profiles
- Per-tool read classification rates (aggregated into benchmark_results.csv)

## How to apply

Download the sequencing reads for the ten mock communities built from human gastrointestinal bacterial isolates (ENA ERP105624 and ERP012217) and convert each fastq/fasta dataset to TFRecord using the DeepMicrobes conversion utility with the merged 12-mer vocabulary. Run DeepMicrobes prediction (the Embed + LSTM + Attention genus model, submitted to GPUs per the repository GPU documentation) alongside the eight comparison tools using the exact command lines provided in the DeepMicrobes-data repository. Generate per-tool taxonomic profiles (genus-level abundances and read-level assignments) with the DeepMicrobes profiling pipeline. Compute community-level precision and recall at 0.01% and 0.0001% abundance cutoffs, L2 distances between predicted and ground-truth abundance profiles (ground truth in Table S4), and per-tool read classification rates. Aggregate with the R benchmarking scripts from DeepMicrobes-data and check DeepMicrobes' average read classification rate (89.40%) against the most sensitive baseline, Kaiju (68.27%).

## Related tools

- **DeepMicrobes** (Deep learning classifier under benchmark; converts reads to TFRecord, predicts on GPUs, and generates taxonomic profiles) — https://github.com/MicrobeLab/DeepMicrobes
- **Kaiju** (Protein-based baseline classifier run on a custom database of predicted protein sequences from the complete bacterial repertoire of the human gut)
- **Kraken** (k-mer/LCA-based baseline classifier for genus-level read comparison)
- **R** (Aggregates precision/recall, L2 distances, and classification rates into the final benchmark tables and figures) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Evaluation signals

- DeepMicrobes' average read classification rate on the ten mock communities matches the reported 89.40%, versus Kaiju's 68.27%
- Precision/recall curves reproduce the Figure 3 values at both the 0.01% and 0.0001% abundance cutoffs
- Generated abundance profiles match the genus-level profiles released in the DeepMicrobes-data repository for the mock communities
- L2 distance between predicted and Table S4 ground-truth genus profiles falls within the published comparison for each tool
- All eight baseline tools are run with the exact command lines provided in the DeepMicrobes-data Scripts directory

## Limitations

- Adding new species requires retraining the entire deep neural network
- The k-mer length (12 bp) is optimized for thousands of genomes; other scenarios should try different k-mer lengths (e.g. 6-12 bp)
- Retraining might be required for reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)

## Evidence

- [abstract] advantage over baselines in genus identification: "We show the advantage of DeepMicrobes over state-of-the-art tools in species and genus identification and comparable accuracy in abundance estimation."
- [readme] benchmark data provenance: [evidence span withheld — anchor status: external_doc]
- [supplementary] abundance profiles released for the ten mock communities: "The sequences of benchmark datasets, the abundance profiles of different taxonomic classification tools on the ten mock communities"
- [readme] baseline command lines available in repository: [evidence span withheld — anchor status: external_doc]
- [other] reproduction acceptance criterion from the task card: [evidence span withheld — anchor status: not_found]
