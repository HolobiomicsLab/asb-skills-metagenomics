---
name: metagenomic-sequence-preprocessing
description: Use when raw FASTA/FASTQ metagenomic reads or training genomes must be
  transformed into fixed-length k-mer token ID sequences (canonical k-mers, <unk>
  handling, padding index 0) before DNN embedding and training.
when_to_use_negative:
- Input is already tokenized/encoded (e.g. existing TFRecord files) — re-counting
  k-mers would duplicate preprocessing
- The goal is alignment- or LCA-based classification (e.g. Kraken-style database building)
  rather than k-mer embedding for a neural network
- Reads come from a sequencing platform with lengths/error profiles strongly different
  from the training data (e.g. long reads), where the paper warns retraining — including
  re-encoding — may be required
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Provides the vocabulary construction, <unk>/padding conventions, and the fastq/fasta-to-TFRecord
    conversion that feed its embedding layer
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Jellyfish
  role: Counts k-mers over the training corpus with stride 1 in canonicalization mode
    (-C) to build the canonical k-mer vocabulary
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:d8b6fc7609e658f62084d1f89a1d730cefc741e9b3b8fb0d94ebeebea3af0ef5
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
  - Jellyfish
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metagenomic-sequence-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes raw metagenomic FASTA/FASTQ sequences into integer-indexed k-mer token sequences (canonical form, with <unk> for N-containing k-mers and index 0 reserved for padding) so they can be converted to TFRecord and fed to the DeepMicrobes embedding layer for taxonomic classification.

## When to use

Use when raw FASTA/FASTQ metagenomic reads or training genomes must be transformed into fixed-length k-mer token ID sequences (canonical k-mers, <unk> handling, padding index 0) before DNN embedding and training.

## When NOT to use

- Input is already tokenized/encoded (e.g. existing TFRecord files) — re-counting k-mers would duplicate preprocessing
- The goal is alignment- or LCA-based classification (e.g. Kraken-style database building) rather than k-mer embedding for a neural network
- Reads come from a sequencing platform with lengths/error profiles strongly different from the training data (e.g. long reads), where the paper warns retraining — including re-encoding — may be required

## Inputs

- Training genome FASTA files (e.g. the 2,505 gut bacterial reference genomes)
- Metagenomic sequencing reads in FASTQ or FASTA format
- Chosen k-mer length (article range: 6-12 bp)

## Outputs

- Plain-text k-mer vocabulary file (canonical k-mers in lexical order, index 0 = padding, <unk> entry for N-containing k-mers)
- TFRecord files of tokenized sequences (via the fastq/fasta-to-TFRecord conversion step)
- Jellyfish k-mer count table over the training corpus

## How to apply

Count k-mers over the training genome FASTA corpus with stride 1 using Jellyfish in canonicalization mode (-C), so each k-mer and its reverse complement collapse to a single canonical form. Order the distinct canonical k-mers lexically and assign each a unique integer index, reserving position 0 for the padding token; map any k-mer containing an ambiguous base (N) to the <unk> token instead of a vocabulary entry. Write the vocabulary-to-index mapping to a plain-text vocabulary file that backs the embedding layer during model training, then convert the tokenized fastq/fasta sequences to TFRecord as documented in the DeepMicrobes repository. Choose the k-mer length deliberately: the study's k was optimized for thousands of genomes, and the authors advise trying 6-12 bp for other scenarios, since k drives vocabulary size (Table S7). Rationale: the paper found that k-mer embedding significantly boosts model performance, making correct vocabulary construction a prerequisite for the Embed + LSTM + Attention classifier.

## Related tools

- **DeepMicrobes** (Provides the vocabulary construction, <unk>/padding conventions, and the fastq/fasta-to-TFRecord conversion that feed its embedding layer) — https://github.com/MicrobeLab/DeepMicrobes
- **Jellyfish** (Counts k-mers over the training corpus with stride 1 in canonicalization mode (-C) to build the canonical k-mer vocabulary)

## Examples

```
jellyfish count -C -m 12 -s 100M -t 32 training_genomes.fa
```

## Evaluation signals

- Vocabulary file exists and index 0 is reserved exclusively for the padding token
- <unk> token is present in the vocabulary and every k-mer containing an N maps to it rather than a distinct entry
- Canonical k-mers listed in the vocabulary are sorted in lexical order (structural QC; the paper reports no numeric threshold for this step)
- Each observed k-mer collapses to one canonical form (k-mer and reverse complement share a single index)
- Downstream TFRecord conversion completes and token IDs fall within [0, vocabulary size - 1]

## Limitations

- The k-mer length used in the study is optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (6-12 bp)
- Retraining (and thus re-encoding) might be required to adapt to reads whose lengths and error profiles are strongly different, e.g. long-read sequencing
- No software version numbers or numeric acceptance threshold are stated for this preprocessing step in the article or README

## Evidence

- [discussion] should try different k-mer lengths (e.g. 6-12 bp): [evidence span withheld — anchor status: not_found]
