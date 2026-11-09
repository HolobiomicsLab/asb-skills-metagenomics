---
name: sequence-to-tfrecord-conversion
description: Use when short metagenomic sequencing reads (FASTQ/FASTA) must be encoded
  as TFRecord with the DeepMicrobes merged 12-mer vocabulary before training a DNN
  classifier or running prediction on held-out reads.
when_to_use_negative:
- The input data are already in TFRecord format produced with the same 12-mer merged
  vocabulary
- The downstream classifier is not DeepMicrobes (e.g. Kraken or Kaiju), which consume
  k-mer/protein databases rather than TFRecord embeddings
- Reads have lengths and error profiles strongly different from the short Illumina-style
  reads the 12-mer vocabulary was tuned for (e.g. long-read sequencing)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DeepMicrobes
  role: Provides the tfrecord conversion utility and documentation (document/tfrecord.md)
    that converts FASTQ/FASTA sequences to TFRecord for model training and prediction
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: Distributes the 12-mer vocabulary used in TFRecord conversion, including merged
    complementary k-mers, and the labeled genome sequences used to create the training
    set
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:d83cd10be304750971a62ea8174542addb82764a888dbb1106a9d1e27037d2be
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sequence-to-tfrecord-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert metagenomic FASTQ/FASTA short reads into TensorFlow TFRecord format using the DeepMicrobes tfrecord utility, encoding sequences with the paper's 12-mer vocabulary so they can be consumed by the k-mer embedding + BiLSTM + self-attention classification models.

## When to use

Use when short metagenomic sequencing reads (FASTQ/FASTA) must be encoded as TFRecord with the DeepMicrobes merged 12-mer vocabulary before training a DNN classifier or running prediction on held-out reads.

## When NOT to use

- The input data are already in TFRecord format produced with the same 12-mer merged vocabulary
- The downstream classifier is not DeepMicrobes (e.g. Kraken or Kaiju), which consume k-mer/protein databases rather than TFRecord embeddings
- Reads have lengths and error profiles strongly different from the short Illumina-style reads the 12-mer vocabulary was tuned for (e.g. long-read sequencing)

## Inputs

- FASTQ or FASTA short metagenomic sequencing read files (training reads simulated from the 2505 human gut reference genomes, held-out reads from 3269 gut-derived MAGs, or query metagenome reads)
- DeepMicrobes k-mer vocabulary files for TFRecord conversion (12-mers, with merged complementary k-mers)
- Sequence labels embedded in FASTA/FASTQ sequence IDs (for training data)

## Outputs

- TFRecord files encoding each read as merged 12-mer vocabulary indices plus label information
- TFRecord-formatted prediction inputs ready for the DeepMicrobes prediction workflow

## How to apply

Retrieve the k-mer vocabulary shipped with MicrobeLab/DeepMicrobes-data (the pre-trained DeepMicrobes models use 12-mers) and note that complementary k-mers must be merged, since they share the same embedding vector. Run the DeepMicrobes TFRecord conversion documented in document/tfrecord.md on the FASTQ/FASTA inputs, keeping training labels embedded in the sequence IDs as provided in the DeepMicrobes-data genome collections. Apply the same vocabulary and merging rules to training reads (e.g. simulated reads from the 2505 human gut reference genomes), held-out reads (e.g. reads from the 3269 ERP108418 MAGs), and any new query metagenome so that the embedding indices are consistent between training and prediction. The rationale is that DeepMicrobes reads k-mer embedding indices, not raw bases, so the TFRecord step is mandatory before 'How to train the DNN model' or 'How to make predictions' workflows. For genomes beyond the paper's data volume, consider whether a different k-mer length (6-12 bp) is warranted before fixing the vocabulary.

## Related tools

- **DeepMicrobes** (Provides the tfrecord conversion utility and documentation (document/tfrecord.md) that converts FASTQ/FASTA sequences to TFRecord for model training and prediction) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Distributes the 12-mer vocabulary used in TFRecord conversion, including merged complementary k-mers, and the labeled genome sequences used to create the training set) — https://github.com/MicrobeLab/DeepMicrobes-data

## Evaluation signals

- The output parses as valid TFRecord (e.g. loadable via tf.data.TFRecordDataset) with per-read sequences encoded as 12-mer vocabulary indices rather than raw bases
- Complementary k-mers map to the same embedding index, confirming the 'merged' k-mer representation was applied during conversion
- Training labels are preserved and match the labels embedded in the source FASTA/FASTA sequence IDs from DeepMicrobes-data
- The same vocabulary file was used for training reads and held-out/prediction reads so that k-mer indices are consistent between models and queries
- Downstream training or prediction jobs (submitted per the GPU guide) run without vocabulary-size or out-of-index errors

## Limitations

- The 12-mer k-mer length used was optimized for the typical data volume of thousands of genomes; other scenarios may require trying different k-mer lengths (e.g. 6-12 bp)
- Adding new species requires retraining the entire deep neural network, so new vocabulary/conversion plus retraining is needed for expanded reference sets
- Retraining and reconversion might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- Complementary k-mers must be represented as merged k-mers during conversion; forgetting this step yields inconsistent embedding vectors

## Evidence

- [readme] README specifies the 12-mer vocabulary: [evidence span withheld — anchor status: external_doc]
- [readme] README explains merged complementary k-mers: [evidence span withheld — anchor status: external_doc]
