---
name: simulated-metagenomic-read-generation
description: Use when benchmarking or ablation-testing a taxonomic classifier (e.g.
  k-mer embedding size, architecture variants) because labeled real reads with known
  species-of-origin are unavailable; reads must be simulated from genomes with known
  taxonomy.
when_to_use_negative:
- Input is real metagenomic sequencing data (e.g. iHMP/IBD fastq) for classification
  or profiling — classify it directly instead of simulating
- Task is abundance estimation on real mock communities where isolate-based mock communities
  already exist
- Benchmarking alignment/k-mer tools like Kraken or Kaiju on data whose simulation
  assumptions would trivially favor one tool
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Provides the fastq/fasta-to-TFRecord conversion utility and the training/prediction
    routines that consume the simulated reads for benchmarking
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: Ships the benchmark simulated read datasets (reads simulated from 3,269 gut-derived
    MAGs and species absent from reference databases) and the k-mer vocabulary
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:efa15d07e2c98a8866b9b5b1cf2609be932ea442da9e1de99205b35eaf712808
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
  - DeepMicrobes-data
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulated-metagenomic-read-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate in silico sequencing reads from labeled reference genomes (e.g. the 3,269 gut-derived MAGs or the 2,505-species human gut bacterial repertoire) so that DeepMicrobes and other taxonomic classifiers can be benchmarked with known ground-truth labels per read.

## When to use

Use when benchmarking or ablation-testing a taxonomic classifier (e.g. k-mer embedding size, architecture variants) because labeled real reads with known species-of-origin are unavailable; reads must be simulated from genomes with known taxonomy.

## When NOT to use

- Input is real metagenomic sequencing data (e.g. iHMP/IBD fastq) for classification or profiling — classify it directly instead of simulating
- Task is abundance estimation on real mock communities where isolate-based mock communities already exist
- Benchmarking alignment/k-mer tools like Kraken or Kaiju on data whose simulation assumptions would trivially favor one tool

## Inputs

- Labeled reference genome collections in FASTA (e.g. 2,505 gut species genomes or 3,269 high-quality MAGs from Table S2)
- Sequencing platform error profiles for fixed-length read simulation
- k-mer vocabulary files used in TFRecord conversion (12-mers for pre-trained DeepMicrobes models)
- Desired coverage/read counts per genome

## Outputs

- Simulated fastq/fasta read files with species labels embedded in sequence IDs
- TFRecord files produced by the DeepMicrobes conversion utility
- Read-level precision and recall tables per model variant (e.g. Tables S9–S12 style outputs)

## How to apply

Simulate reads from the labeled reference genomes (DeepMicrobes-data ships 'Reads simulated from 3,269 gut-derived MAGs', with MAGs excluded from training per Table S2, and mock communities from gastrointestinal bacterial isolates). Choose fixed-length reads simulated under error profiles of specific sequencing platforms (Illumina/PacBio/ONT style, per Tables S11/S12) or variable-length reads (Tables S9/S10) depending on the benchmark. Convert the resulting fastq/fasta reads to TFRecord using the DeepMicrobes conversion utility, matching the k-mer vocabulary of the model being tested (pre-trained DeepMicrobes models use 12-mers, with complementary k-mers merged into the same embedding vector). Train/evaluate the model on these TFRecords and compute read-level precision and recall against the simulated ground-truth species labels. For ablations such as the k-mer length study, keep the simulation identical across K in {8,...,12} so precision/recall differences are attributable to the embedding, not the data.

## Related tools

- **DeepMicrobes** (Provides the fastq/fasta-to-TFRecord conversion utility and the training/prediction routines that consume the simulated reads for benchmarking) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Ships the benchmark simulated read datasets (reads simulated from 3,269 gut-derived MAGs and species absent from reference databases) and the k-mer vocabulary) — https://github.com/MicrobeLab/DeepMicrobes-data

## Evaluation signals

- Simulated reads come only from MAGs excluded from training (per Table S2), so precision/recall reflect generalization rather than memorization
- TFRecord conversion completes without vocabulary mismatch (e.g. 'merged' k-mers used so complementary k-mers share embeddings for 12-mer models)
- Read-level precision and recall are computed per condition and assemble into comparison tables like Tables S9–S12
- In the k-mer ablation, the 12-mer embedding must markedly outperform 8- to 11-mer variants, reproducing Supplementary Figures S7/S11
- Simulated-read benchmarks on species absent from reference databases should show classifier false positives consistent with Table S6-style controls

## Limitations

- Simulated reads are only as realistic as the error profiles and length models used; fixed-length simulation per platform (Tables S11/S12) and variable-length simulation (Tables S9/S10) can yield different results
- DeepMicrobes' k-mer length is optimized for thousands of genomes; for other data volumes, 'should try different k-mer lengths (e.g. 6-12 bp)'
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different from the simulated/training distribution
- Species not in the reference/training set require dedicated simulation (reads from species absent from reference databases) to expose false positives

## Evidence

- [readme] Reads simulated from 3,269 gut-derived MAGs: [evidence span withheld — anchor status: external_doc]
- [readme] Mock communities generated from gastrointestinal bacterial isolates: [evidence span withheld — anchor status: external_doc]
- [readme] TFRecord conversion and 12-mer vocabulary: [evidence span withheld — anchor status: external_doc]
- [supplementary] 12-mer ablation result: [evidence span withheld — anchor status: external_doc]
- [discussion] k-mer length choice caveat: [evidence span withheld — anchor status: not_found]
