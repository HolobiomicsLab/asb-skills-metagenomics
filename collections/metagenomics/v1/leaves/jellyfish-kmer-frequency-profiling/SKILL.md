---
name: jellyfish-kmer-frequency-profiling
description: Use when preparing the embedding input vocabulary for DeepMicrobes training
  — i.e. you have training genome FASTA files (e.g. the 2,505 gut microbiota species
  repertoire) and need a k-mer-to-index mapping file.
when_to_use_negative:
- The input is already a TFRecord training corpus or a trained DeepMicrobes model
  — no vocabulary construction is needed at prediction or profiling time
- The goal is abundance profiling on an existing metagenome dataset with a pre-trained
  model — this skill only concerns vocabulary construction before training
- Adding new species to an existing classifier — the paper notes this requires retraining
  the entire deep neural network, so a patched vocabulary alone is insufficient
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Jellyfish
  role: Counts k-mers over the training corpus with stride 1 in canonicalization mode
    (-C) to build the canonical vocabulary
- name: DeepMicrobes
  role: Consumer of the vocabulary file — the mapping backs the k-mer embedding layer
    of the Embed + LSTM + Attention DNN
  repo: https://github.com/MicrobeLab/DeepMicrobes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:b1dac3cef8e017859fefaad003f10369d774053aec74d3024d9b4cd593332325
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# jellyfish-kmer-frequency-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the canonical k-mer vocabulary and index mapping that backs the embedding layer of DeepMicrobes: Jellyfish counts k-mers over the training genome corpus in canonicalization mode, k-mers containing Ns map to <unk>, and index 0 is reserved for padding.

## When to use

Use when preparing the embedding input vocabulary for DeepMicrobes training — i.e. you have training genome FASTA files (e.g. the 2,505 gut microbiota species repertoire) and need a k-mer-to-index mapping file.

## When NOT to use

- The input is already a TFRecord training corpus or a trained DeepMicrobes model — no vocabulary construction is needed at prediction or profiling time
- The goal is abundance profiling on an existing metagenome dataset with a pre-trained model — this skill only concerns vocabulary construction before training
- Adding new species to an existing classifier — the paper notes this requires retraining the entire deep neural network, so a patched vocabulary alone is insufficient

## Inputs

- Training genome FASTA files (e.g. the 2,505 human gut microbiota bacterial repertoire used for species-level training)
- Metagenomic read FASTA/FASTQ sequences (via the fastq/fasta-to-TFRecord conversion path)
- k-mer length parameter (6-12 bp range discussed in the paper)

## Outputs

- Plain-text vocabulary file mapping canonical k-mers to integer indices (index 0 = padding, <unk> entry present)
- Jellyfish k-mer counts over the training corpus

## How to apply

Load the training genome FASTA files (or metagenomic read sequences) that serve as the training corpus for the DeepMicrobes DNN. Count k-mers over the corpus with stride 1 using Jellyfish in canonicalization mode (-C) so each k-mer and its reverse complement collapse to a single canonical form. Order the distinct canonical k-mers in lexical order and assign each a unique integer index, reserving position 0 for the padding token; any k-mer containing ambiguous bases (N) is treated as the <unk> token rather than given its own entry. Write the vocabulary-to-index mapping to a plain-text vocabulary file that will back the embedding layer during model training. The k-mer length should match DeepMicrobes' optimization for thousands of genomes; the paper reports 6-12 bp as the range worth trying in other scenarios, and Table S7 documents the effect of k-mer length on vocabulary size.

## Related tools

- **Jellyfish** (Counts k-mers over the training corpus with stride 1 in canonicalization mode (-C) to build the canonical vocabulary)
- **DeepMicrobes** (Consumer of the vocabulary file — the mapping backs the k-mer embedding layer of the Embed + LSTM + Attention DNN) — https://github.com/MicrobeLab/DeepMicrobes

## Examples

```
jellyfish count -C -m 12 -s 10M -t 16 -o kmers.jf training_genomes.fa && jellyfish dump -c kmers.jf > canonical_kmers.txt
```

## Evaluation signals

- Vocabulary file exists and is non-empty after the Jellyfish counting run
- Index 0 is reserved for the padding token in the mapping
- The <unk> symbol is present in the vocabulary to represent k-mers containing Ns
- Listed canonical k-mers are sorted in lexical order (structural QC check — the paper reports no numeric acceptance threshold for this step)
- Vocabulary size is consistent with the chosen k-mer length, cross-checkable against Table S7 (effect of k-mer length on vocabulary size)

## Limitations

- The k-mer length used in the study was optimized for the typical data volume of thousands of genomes; other scenarios (e.g. much larger or smaller corpora) require trying different k-mer lengths (6-12 bp)
- Adapting the vocabulary/model to reads with strongly different lengths and error profiles (e.g. long-read sequencing) may require retraining
- No numeric acceptance threshold is reported for vocabulary construction — QC is structural only

## Evidence

- [supplementary] k-mer length vs vocabulary size supplementary table: [evidence span withheld — anchor status: external_doc]
