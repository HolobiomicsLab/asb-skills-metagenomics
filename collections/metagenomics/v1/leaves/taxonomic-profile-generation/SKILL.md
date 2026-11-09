---
name: taxonomic-profile-generation
description: Use when metagenomic sequencing reads (fastq/fasta) must be converted
  into taxonomic profiles (relative abundances and per-read assignments) for benchmarking
  or abundance estimation of gut microbiome taxa.
when_to_use_negative:
- Input is already a taxonomic abundance profile or feature table rather than raw
  sequencing reads
- Reads come from platforms whose lengths and error profiles differ strongly from
  the training data (e.g. long-read sequencing), since retraining would be required
- 'The goal is classifying taxa absent from the training repertoire: adding new species
  requires retraining the entire deep neural network, so alignment/k-mer tools or
  a retrained model are needed'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: DeepMicrobes
  role: Deep learning classifier whose predictions are converted into per-species/genus
    taxonomic profiles via the repository profiling pipeline
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Kraken
  role: k-mer/LCA-based comparison classifier run on the same mock-community reads
    to produce profiles for benchmarking
- name: Kaiju
  role: Protein-based comparison classifier run against a custom database of predicted
    proteins from the complete human gut bacterial repertoire
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: R
  role: Benchmarking scripts from the DeepMicrobes-data repository used to compute
    metrics and generate comparison figures/tables
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:3217aa1380652e01ad00647b68d2dc03e63619cb82f8da5121928c3efe00eda3
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# taxonomic-profile-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate per-species/genus abundance profiles and read-level taxonomic assignments from metagenome reads by running DeepMicrobes and comparison classifiers, then profiling their predictions. This skill underpins genus/species identification and abundance-estimation benchmarks on gut mock communities.

## When to use

Use when metagenomic sequencing reads (fastq/fasta) must be converted into taxonomic profiles (relative abundances and per-read assignments) for benchmarking or abundance estimation of gut microbiome taxa.

## When NOT to use

- Input is already a taxonomic abundance profile or feature table rather than raw sequencing reads
- Reads come from platforms whose lengths and error profiles differ strongly from the training data (e.g. long-read sequencing), since retraining would be required
- The goal is classifying taxa absent from the training repertoire: adding new species requires retraining the entire deep neural network, so alignment/k-mer tools or a retrained model are needed

## Inputs

- fastq/fasta metagenomic sequencing reads (e.g. ten simulated mock communities from HBC isolate genomes, ENA accessions ERP105624 and ERP012217)
- DeepMicrobes pre-trained species/genus models and 12-mer TFRecord vocabulary
- Ground-truth taxonomic profiles of the mock communities (e.g. Table S4 true genus profiles)
- Custom Kaiju database (predicted protein sequences, names.dmp, nodes.dmp, Kaiju index)

## Outputs

- Per-species/genus relative abundance profiles for each classifier on each mock community
- Read-level taxonomic assignments from every tool
- Community-level precision/recall at 0.01% and 0.0001% abundance cutoffs, L2 abundance distances, and read classification rates aggregated into a benchmark table (e.g. benchmark_results.csv)

## How to apply

Convert each fastq/fasta metagenome dataset to TFRecord format with the DeepMicrobes conversion utility (using the repository's 12-mer vocabulary, with complementary k-mers merged), then run DeepMicrobes prediction, submitting to GPUs as documented. Run comparison classifiers such as Kraken (k-mer/LCA) and Kaiju (protein-based, using the custom GitHub database) on the same reads, and generate taxonomic profiles with the DeepMicrobes profiling pipeline, choosing a confidence threshold per the repository guidance. Compute community-level precision and recall at 0.01% and 0.0001% abundance cutoffs, L2 distances to ground-truth abundance profiles, and per-tool read classification rates, aggregating metrics with the R scripts in DeepMicrobes-data. Validate against reported values, e.g. DeepMicrobes classifying on average 89.40% of reads in ten HBC mock communities versus Kaiju's 68.27%.

## Related tools

- **DeepMicrobes** (Deep learning classifier whose predictions are converted into per-species/genus taxonomic profiles via the repository profiling pipeline) — https://github.com/MicrobeLab/DeepMicrobes
- **Kraken** (k-mer/LCA-based comparison classifier run on the same mock-community reads to produce profiles for benchmarking)
- **Kaiju** (Protein-based comparison classifier run against a custom database of predicted proteins from the complete human gut bacterial repertoire) — https://github.com/MicrobeLab/DeepMicrobes
- **R** (Benchmarking scripts from the DeepMicrobes-data repository used to compute metrics and generate comparison figures/tables) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
convert_to_tfrecord.py --input mock_community.fastq --output mock_community.tfrecord && predict.py --input mock_community.tfrecord --model genus_model && generate_profile.py --predictions predictions.tsv --rank genus --output genus_profile.csv
```

## Evaluation signals

- DeepMicrobes' read classification rate on the ten HBC mock communities matches the reported 89.40% (versus Kaiju's 68.27%)
- Community-level precision and recall reproduce the Figure 3 values at both the 0.01% and 0.0001% abundance cutoffs
- Predicted genus abundance profiles have low L2 distance to the true genus profiles of the ten mock communities (Table S4)
- TFRecord conversion used the merged 12-mer vocabulary so complementary k-mers share embedding vectors
- Per-tool profiles match the abundance profiles deposited in DeepMicrobes-data (genus-level and species-level results)

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads with lengths and error profiles strongly different from training data
- The k-mer length (12 bp) was optimized for thousands of genomes; other scenarios may require trying different k-mer lengths (6-12 bp)
- Confidence threshold choice affects read-level precision/recall (see Table S13 and the confidence-threshold documentation)

## Evidence

- [results] results: [evidence span withheld — anchor status: not_found]
- [readme] readme-vocabulary: [evidence span withheld — anchor status: not_found]
