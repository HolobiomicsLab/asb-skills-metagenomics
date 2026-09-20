---
name: kmer-canonicalization-and-counting
description: Use when preparing training genomes or metagenomic FASTA/FASTQ reads
  for a k-mer-embedding DNN classifier and you need a canonical k-mer-to-index vocabulary
  file backing the embedding layer.
when_to_use_negative:
- Input is already a TFRecord or tokenized read set with an existing vocabulary —
  rebuilding the vocabulary would break index consistency with the trained embedding
  layer
- You need alignment-based or database-backed classification (e.g. BWA mapping or
  LCA databases like Kraken's) rather than an embedding-based DNN — this vocabulary
  is specific to DeepMicrobes' Embed + LSTM + Attention model
- Adding new species to an existing trained model — DeepMicrobes requires retraining
  the entire deep neural network, so the vocabulary must be rebuilt as part of retraining,
  not patched incrementally
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3474
tools:
- name: Jellyfish
  role: Counts k-mers with stride 1 in canonicalization mode (-C) so each k-mer and
    its reverse complement collapse to a single canonical form for the vocabulary
- name: DeepMicrobes
  role: Consumes the canonical k-mer vocabulary to back the embedding layer of its
    Embed + LSTM + Attention taxonomic classifier
  repo: https://github.com/MicrobeLab/DeepMicrobes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:b12ef8d3e85ab937a8608dda8ec52b62500c931e6edb54e01f3e2a098a893a34
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# k-mer canonicalization and counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct the canonical k-mer vocabulary that feeds DeepMicrobes' embedding layer: collapse each k-mer and its reverse complement to a single canonical form with Jellyfish (-C), assign lexical-order integer indices, and reserve index 0 and an <unk> token.

## When to use

Use when preparing training genomes or metagenomic FASTA/FASTQ reads for a k-mer-embedding DNN classifier and you need a canonical k-mer-to-index vocabulary file backing the embedding layer.

## When NOT to use

- Input is already a TFRecord or tokenized read set with an existing vocabulary — rebuilding the vocabulary would break index consistency with the trained embedding layer
- You need alignment-based or database-backed classification (e.g. BWA mapping or LCA databases like Kraken's) rather than an embedding-based DNN — this vocabulary is specific to DeepMicrobes' Embed + LSTM + Attention model
- Adding new species to an existing trained model — DeepMicrobes requires retraining the entire deep neural network, so the vocabulary must be rebuilt as part of retraining, not patched incrementally

## Inputs

- Training genome FASTA files (e.g. the 2,505 species reference genomes from the human gut bacterial repertoire)
- Metagenomic read sequences in FASTA/FASTQ format
- Chosen k-mer length (paper used a length optimized for thousands of genomes; 6-12 bp recommended range for other scenarios)

## Outputs

- Plain-text k-mer vocabulary file mapping canonical k-mers to integer indices, with index 0 reserved for padding and <unk> included for N-containing k-mers
- Jellyfish k-mer counts over the training corpus

## How to apply

Load the training genome FASTA files (e.g. the 2,505 human gut bacterial genomes used to train the DeepMicrobes species model) or metagenomic read sequences as the corpus. Count k-mers with stride 1 using Jellyfish in canonicalization mode (-C) so each k-mer and its reverse complement collapse to a single canonical form; the paper's k-mer length was optimized for thousands of genomes, with 6-12 bp suggested for other data volumes. Order the distinct canonical k-mers lexically and assign each a unique integer index, reserving position 0 for the padding token, and map any k-mer containing ambiguous bases (N) to the <unk> token rather than a vocabulary entry. Write the vocabulary-to-index mapping to a plain-text vocabulary file that backs the embedding layer during DNN training; Table S7 reports how k-mer length drives vocabulary size, which matters because k-mer embedding significantly boosts model performance.

## Related tools

- **Jellyfish** (Counts k-mers with stride 1 in canonicalization mode (-C) so each k-mer and its reverse complement collapse to a single canonical form for the vocabulary)
- **DeepMicrobes** (Consumes the canonical k-mer vocabulary to back the embedding layer of its Embed + LSTM + Attention taxonomic classifier) — https://github.com/MicrobeLab/DeepMicrobes

## Examples

```
jellyfish count -C -m 12 -s 100M -t 16 training_genomes.fa
```

## Evaluation signals

- Vocabulary file exists and is well-formed after the build
- Index 0 is reserved for the padding token
- The <unk> token is present and used for k-mers containing Ns
- Listed canonical k-mers are sorted in lexical order (structural QC check; the paper reports no numeric acceptance threshold for this step)
- Vocabulary size matches the expectation for the chosen k-mer length (cf. Table S7, 'The effect of k-mer length on vocabulary size')

## Limitations

- The k-mer length used in the study is optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (e.g. 6-12 bp)
- No numeric acceptance threshold is reported for vocabulary QC — validation is structural only
- Adding new species requires retraining the entire deep neural network, so the vocabulary cannot be incrementally extended for an already-trained model
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)

## Evidence

- [supplementary] k-mer length vs vocabulary size table: [evidence span withheld — anchor status: external_doc]
