---
name: fastq-to-tfrecord-conversion
description: Use when feeding FASTQ or FASTA metagenomic reads (e.g. simulated gut-MAG
  reads or iHMP gut metagenomes) into DeepMicrobes for training or prediction, since
  the LSTM/embedding models only consume TFRecord input.
when_to_use_negative:
- Input is not short-read metagenomic sequence (e.g. long-read data) — the article
  notes retraining may be required for reads with strongly different lengths and error
  profiles
- You plan to use an alignment- or k-mer-index-based classifier such as Kraken or
  Kaiju, which do not consume TFRecords
- A custom model with a k-mer length other than the default 12 is being trained without
  generating a matching (merged) vocabulary
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: DeepMicrobes
  role: Provides the fastq/fasta-to-TFRecord conversion utility and the 12-mer (merged)
    vocabulary consumed by the embedding layer
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: Ships the k-mer vocabulary files used in TFRecord conversion and the supporting
    data/tables (e.g. Table S7 vocabulary size)
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:1039ad4653a156802f32c3b245a921bdda713b67aa4042b211fc73e724373000
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
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fastq-to-tfrecord-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts FASTQ/FASTA metagenomic reads into the TFRecord format required by the DeepMicrobes deep learning framework, encoding sequences as k-mer token IDs from the framework's vocabulary (12-mers for the pre-trained models).

## When to use

Use when feeding FASTQ or FASTA metagenomic reads (e.g. simulated gut-MAG reads or iHMP gut metagenomes) into DeepMicrobes for training or prediction, since the LSTM/embedding models only consume TFRecord input.

## When NOT to use

- Input is not short-read metagenomic sequence (e.g. long-read data) — the article notes retraining may be required for reads with strongly different lengths and error profiles
- You plan to use an alignment- or k-mer-index-based classifier such as Kraken or Kaiju, which do not consume TFRecords
- A custom model with a k-mer length other than the default 12 is being trained without generating a matching (merged) vocabulary

## Inputs

- FASTQ metagenomic sequencing reads (e.g. iHMP PRJNA398089 gut metagenomes)
- FASTA nucleotide sequences
- k-mer vocabulary file (merged 12-mer vocabulary used by pre-trained DeepMicrobes models)

## Outputs

- TFRecord files encoding k-mer token IDs per read, consumable by DeepMicrobes training/prediction

## How to apply

Take the FASTQ/FASTA read files and run DeepMicrobes' conversion utility (documented in document/tfrecord.md) to serialize reads into TFRecord format. The conversion applies the k-mer vocabulary shipped with the framework: the pre-trained DeepMicrobes models use 12-mers, and Table S7 documents the effect of k-mer length on vocabulary size. Crucially, complementary k-mers are mapped to the same embedding vector, which requires using the 'merged' k-mer vocabulary when converting — a wrong (unmerged) vocabulary will silently mismatch the pre-trained model. The TFRecord output then serves directly as input to DeepMicrobes training (with a chosen embedding size K, e.g. K=8–12 for ablations) or to the species-level LSTM prediction step.

## Related tools

- **DeepMicrobes** (Provides the fastq/fasta-to-TFRecord conversion utility and the 12-mer (merged) vocabulary consumed by the embedding layer) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Ships the k-mer vocabulary files used in TFRecord conversion and the supporting data/tables (e.g. Table S7 vocabulary size)) — https://github.com/MicrobeLab/DeepMicrobes-data

## Evaluation signals

- TFRecord files are readable by TensorFlow (tf.data.TFRecordDataset) and parse into per-read examples matching the model's expected features
- Conversion used the 'merged' k-mer vocabulary so complementary 12-mers share token IDs, consistent with the pre-trained model's embedding matrix size
- Downstream DeepMicrobes prediction runs without vocabulary/embedding dimension errors, indicating k=12 tokens align with the 12-mer pre-trained models
- Read counts in TFRecords match the number of records in the source FASTQ/FASTA files (no reads dropped during conversion)

## Limitations

- The pre-trained models use 12-mers optimized for thousands of training genomes; other scenarios may require trying different k-mer lengths (e.g. 6–12 bp), which means the conversion vocabulary must change accordingly
- Adding new species requires retraining the entire deep neural network, so conversion for a new reference set implies regenerating training TFRecords
- Reads with lengths and error profiles strongly different from the training data (e.g. long reads) may require retraining rather than simple conversion

## Evidence

- [readme] Vocabulary readme: [evidence span withheld — anchor status: external_doc]
- [readme] Merged k-mer requirement: [evidence span withheld — anchor status: external_doc]
