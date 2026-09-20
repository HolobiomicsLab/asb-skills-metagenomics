---
name: deep-neural-network-training-on-genomes
description: Use when labeled whole-genome sequences (FASTA with training labels in
  sequence IDs) must be turned into a DNN read classifier and architecture variants
  (one-hot CNN, hybrid, LSTM, self-attention) need to be compared by read-level precision
  and recall.
when_to_use_negative:
- The input reads come from long-read platforms or have error profiles strongly different
  from the training data — retraining is required in that case per the discussion.
- The goal is to add new species to an already-trained classifier without retraining
  — the current framework requires retraining the entire deep neural network.
- A reference-free or alignment-free abundance-only estimate is needed without species-level
  read classification (e.g. genus-level profiling with a chosen confidence threshold
  may suffice).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3697
tools:
- name: DeepMicrobes
  role: Deep learning framework used to convert sequences to TFRecord, train each
    DNN architecture variant on GPUs, and predict held-out MAG reads
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: Provides benchmark training genomes, the 3269 MAG test reads, k-mer vocabulary,
    supplementary tables, and benchmark command lines
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
- name: R
  role: R scripts used to generate benchmarking figures from aggregated precision/recall
    results
  repo: https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:2a2195518ea0afe32870ae21b0b5489206732512068c63c930fac73abe0ba640
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
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# deep-neural-network-training-on-genomes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains deep neural network architectures (e.g. k-mer embedding + BiLSTM + self-attention) on labeled genome sequences converted to TFRecord, to classify short shotgun metagenomic reads at species rank, as done in DeepMicrobes with the 2505-genome human gut bacterial repertoire.

## When to use

Use when labeled whole-genome sequences (FASTA with training labels in sequence IDs) must be turned into a DNN read classifier and architecture variants (one-hot CNN, hybrid, LSTM, self-attention) need to be compared by read-level precision and recall.

## When NOT to use

- The input reads come from long-read platforms or have error profiles strongly different from the training data — retraining is required in that case per the discussion.
- The goal is to add new species to an already-trained classifier without retraining — the current framework requires retraining the entire deep neural network.
- A reference-free or alignment-free abundance-only estimate is needed without species-level read classification (e.g. genus-level profiling with a chosen confidence threshold may suffice).

## Inputs

- Labeled genome sequences (FASTA) for the 2505 human gut bacterial repertoire, with training labels encoded in sequence IDs
- Simulated/real sequencing reads in FASTQ or FASTA format
- k-mer vocabulary file for TFRecord conversion (12-mers, complementary k-mers merged)

## Outputs

- Trained DeepMicrobes DNN model files (one per architecture variant)
- TFRecord-formatted training and test datasets
- Per-architecture read-level precision and recall tables (cf. Tables S9-S12)

## How to apply

First convert labeled FASTQ/FASTA training sequences to TFRecord using the DeepMicrobes tfrecord utility, merging complementary 12-mers so they share an embedding vector. Train each candidate architecture (one-hot CNN, hybrid, seq2species, embedding baseline, LSTM, embedding + BiLSTM + self-attention) on the TFRecord data from the 2505 human gut reference genomes, submitting jobs to GPUs per the DeepMicrobes GPU guide and adopting hyperparameters from the paper's search space (Table S8). Run prediction with each trained model on held-out variable-length reads simulated from 3269 gut-derived MAGs (Table S2). Aggregate read-level precision and recall per architecture; the embedding + BiLSTM + self-attention model is expected to win with mean precision 0.942 and mean recall 0.428, matching the Figure 2 architecture-selection result. Rationale: k-mer embedding significantly boosts model performance, and LSTM surpasses CNN because order and context of oligonucleotides carry taxonomic signal.

## Related tools

- **DeepMicrobes** (Deep learning framework used to convert sequences to TFRecord, train each DNN architecture variant on GPUs, and predict held-out MAG reads) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Provides benchmark training genomes, the 3269 MAG test reads, k-mer vocabulary, supplementary tables, and benchmark command lines) — https://github.com/MicrobeLab/DeepMicrobes-data
- **R** (R scripts used to generate benchmarking figures from aggregated precision/recall results) — https://github.com/MicrobeLab/DeepMicrobes-data/tree/master/scripts

## Examples

```
python DeepMicrobes/scripts/tfrecord.py --input training_reads.fastq --output train.tfrecord --type fastq --vocabulary vocabulary_merged_12mers.txt
```

## Evaluation signals

- The embedding + BiLSTM + self-attention model attains mean read-level precision = 0.942 and mean recall = 0.428 on held-out MAG reads, beating the second-best LSTM variant (Figure 2 reproduction check).
- Precision/recall values per architecture match Supplementary Tables S9 and S10 for variable-length reads simulated from the 3269 training-excluded MAGs.
- TFRecord conversion succeeds only with the 'merged' 12-mer vocabulary so complementary k-mers share one embedding vector.
- Each model's hyperparameters correspond to the chosen values in Table S8 of the paper's search space.
- Per-class performance is computed against training labels embedded in the sequence IDs of the held-out MAG reads.

## Limitations

- Adding new species requires retraining the entire deep neural network.
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing).
- The k-mer length used is optimized for data volumes of thousands of genomes; other scenarios should try different k-mer lengths (e.g. 6-12 bp).
- Mean recall of 0.428 on held-out MAGs indicates substantial false negatives, so recall on novel species remains limited.

## Evidence

- [results] embedding-based recurrent self-attention model selected as DeepMicrobes architecture: [evidence span withheld — anchor status: not_found]
- [readme] complementary k-mer merging in vocabulary: [evidence span withheld — anchor status: external_doc]
- [readme] held-out MAG benchmark reads: [evidence span withheld — anchor status: external_doc]
