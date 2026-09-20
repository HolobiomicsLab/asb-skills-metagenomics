---
name: classifier-precision-recall-evaluation
description: Use when multiple taxonomic classification model architectures must be
  compared on short shotgun metagenomic reads to select the one with the read-level
  precision and recall before deploying it for profiling.
when_to_use_negative:
- The goal is abundance estimation or genus/species profiling on real metagenomes
  with mock communities rather than architecture selection on held-out genomes
- Reads come from long-read platforms with strongly different lengths and error profiles
  — the article notes retraining would be required for such data
- New species must be added to an existing trained model — the framework requires
  retraining the entire deep neural network
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Framework used to convert reads to TFRecord, train each DNN architecture variant,
    and run prediction to compute read-level precision and recall
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: 'Source of benchmark datasets: reads simulated from 3269 gut-derived MAGs,
    training genomes, vocabulary, and Tables S9/S10 precision-recall results'
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:4bf853face995b9035c61b6db101617154a6f2383849408eddea867eb6c4b3ae
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# classifier-precision-recall-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark deep-learning taxonomic classifiers by measuring read-level precision and recall on variable-length reads simulated from held-out genomes, as done in DeepMicrobes to compare one-hot CNN, hybrid, seq2species, embedding, LSTM, and self-attention architectures and select the best (Embed + LSTM + Attention).

## When to use

Use when multiple taxonomic classification model architectures must be compared on short shotgun metagenomic reads to select the one with the best read-level precision and recall before deploying it for profiling.

## When NOT to use

- The goal is abundance estimation or genus/species profiling on real metagenomes with mock communities rather than architecture selection on held-out genomes
- Reads come from long-read platforms with strongly different lengths and error profiles — the article notes retraining would be required for such data
- New species must be added to an existing trained model — the framework requires retraining the entire deep neural network

## Inputs

- Simulated training reads from 2505 human gut reference genomes (FASTQ/FASTA, labels in sequence IDs)
- Simulated variable-length test reads from 3269 high-quality gut-derived MAGs excluded from training (Table S2)
- TFRecord-converted training/test sequence files (12-mer merged vocabulary)
- Trained model checkpoints for each DNN architecture variant

## Outputs

- Read-level precision per architecture (Table S9-style aggregate, e.g. mean = 0.942 for Embed + LSTM + Attention)
- Read-level recall per architecture (Table S10-style aggregate, e.g. mean = 0.428 for Embed + LSTM + Attention)
- Architecture-selection decision identifying the best model (Embed + LSTM + Attention)

## How to apply

Obtain simulated training reads from the 2505 human gut reference genomes and held-out test reads simulated from the 3269 gut-derived MAGs (Table S2) that were excluded from training, both available from the MicrobeLab/DeepMicrobes-data repository. Convert FASTQ/FASTA sequence files to TFRecord format using the DeepMicrobes tfrecord conversion utility (using 12-mer vocabulary with complementary k-mers merged) before training. Train each candidate architecture — one-hot CNN, hybrid, seq2species, k-mer embedding baseline, BiLSTM, and embedding + BiLSTM + self-attention — on the TFRecord data, submitting jobs to GPUs per the DeepMicrobes GPU guide, using the hyperparameter search space documented in Table S8. Run prediction on the held-out MAG reads with each trained model and aggregate read-level precision (Table S9) and recall (Table S10) per architecture. The embedding-based recurrent self-attention model should attain mean read-level precision of 0.942 and recall of 0.428, exceeding the second-best LSTM variant, which justifies its selection as the DeepMicrobes architecture.

## Related tools

- **DeepMicrobes** (Framework used to convert reads to TFRecord, train each DNN architecture variant, and run prediction to compute read-level precision and recall) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Source of benchmark datasets: reads simulated from 3269 gut-derived MAGs, training genomes, vocabulary, and Tables S9/S10 precision-recall results) — https://github.com/MicrobeLab/DeepMicrobes-data

## Evaluation signals

- The embedding + BiLSTM + self-attention model attains mean read-level precision of 0.942 and recall of 0.428 on variable-length reads from the 3269 held-out MAGs, matching Figure 2 / Tables S9–S10
- The evaluation MAGs (3269, Table S2) are disjoint from the 2505 training genomes (Table S1) — no leakage between train and test sets
- Input sequences were converted to TFRecord with the merged complementary 12-mer vocabulary before training
- Each architecture's precision and recall are aggregated over the same held-out reads, enabling a like-for-like ranking where LSTM surpasses one-hot CNN and embedding outperforms one-hot inputs

## Limitations

- Adding new species requires retraining the entire deep neural network, so precision/recall benchmarks do not transfer to newly discovered organisms
- The k-mer length (12-mers) was optimized for the data volume of thousands of genomes; other scenarios require trying different k-mer lengths (6–12 bp)
- Retraining might be required to adapt to reads with strongly different lengths and error profiles (e.g. long-read sequencing)

## Evidence

- [results] The embedding-based recurrent self-attention model achieves higher read-level precision (mean = 0.942) and recall (mean = 0.428) than the second-best LSTM variant: [evidence span withheld — anchor status: not_found]
- [readme] Table S9: read-level precision of architectures on variable-length MAG reads: [evidence span withheld — anchor status: external_doc]
- [readme] Table S2: metadata of the held-out benchmark MAGs: [evidence span withheld — anchor status: external_doc]
- [discussion] k-mer embedding and LSTM findings supporting the architecture comparison: "We found that k-mer embedding significantly boosts model performance."
