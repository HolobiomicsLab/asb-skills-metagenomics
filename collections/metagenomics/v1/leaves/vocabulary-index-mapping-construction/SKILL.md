---
name: vocabulary-index-mapping-construction
description: Use when preparing a deep-learning taxonomic classifier (e.g. DeepMicrobes
  Embed+LSTM+Attention) that needs a k-mer-to-integer vocabulary file feeding an embedding
  layer, or when building a canonical k-mer lexicon from FASTA training genomes.
when_to_use_negative:
- The input is already a TFRecord or a finished vocabulary file — use DeepMicrobes'
  fastq/fasta-to-TFRecord conversion step instead of rebuilding the vocabulary
- You only need a k-mer database for an alignment/LCA classifier like Kraken or Kaiju
  — those tools build their own databases and do not consume an embedding-layer vocabulary
  file
- You are classifying already-trained DeepMicrobes models on new reads — prediction
  reuses the existing vocabulary and does not require reconstruction
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Jellyfish
  role: Counts k-mers over the training corpus with stride 1 in canonicalization mode
    (-C) so each k-mer and its reverse complement collapse to a single canonical form
- name: DeepMicrobes
  role: Consumes the vocabulary-to-index mapping to back the k-mer embedding layer
    used during DNN training; vocabulary construction is part of its training workflow
  repo: https://github.com/MicrobeLab/DeepMicrobes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:9bdb09c68dd8f6caf307f1faf0f8539a93908e25e7e4242a1259d848dbea2004
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# vocabulary-index-mapping-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct the canonical k-mer vocabulary for DeepMicrobes by counting k-mers over the training genome corpus with Jellyfish in canonicalization mode (-C), then assign each distinct canonical k-mer a unique integer index that backs the embedding layer, reserving index 0 for padding and using <unk> for k-mers containing ambiguous N bases.

## When to use

Use when preparing a deep-learning taxonomic classifier (e.g. DeepMicrobes Embed+LSTM+Attention) that needs a k-mer-to-integer vocabulary file feeding an embedding layer, or when building a canonical k-mer lexicon from FASTA training genomes.

## When NOT to use

- The input is already a TFRecord or a finished vocabulary file — use DeepMicrobes' fastq/fasta-to-TFRecord conversion step instead of rebuilding the vocabulary
- You only need a k-mer database for an alignment/LCA classifier like Kraken or Kaiju — those tools build their own databases and do not consume an embedding-layer vocabulary file
- You are classifying already-trained DeepMicrobes models on new reads — prediction reuses the existing vocabulary and does not require reconstruction

## Inputs

- Training genome FASTA files (e.g. the 2,505 bacterial genomes of the human gut repertoire)
- Metagenomic read FASTA/FASTQ sequences used as an alternative corpus
- Chosen k-mer length (within the 6-12 bp range suggested in the paper)

## Outputs

- Plain-text vocabulary-to-index mapping file with index 0 reserved for padding and an <unk> symbol for N-containing k-mers
- Jellyfish canonical k-mer counts over the training corpus

## How to apply

Load the training genome FASTA files (e.g. the 2,505-gut-species repertoire) that serve as the corpus, then count k-mers with stride 1 using Jellyfish in canonicalization mode (-C) so each k-mer and its reverse complement collapse to a single canonical form. Order the distinct canonical k-mers in lexical order and assign each a unique integer index, explicitly reserving position 0 for the padding token. Treat any k-mer containing ambiguous bases (N) as the <unk> token rather than giving it a vocabulary entry. Write the vocabulary-to-index mapping to a plain-text vocabulary file that backs the embedding layer during DNN training. Choose the k-mer length within the 6-12 bp range discussed in the paper (the paper's default was optimized for thousands of genomes); note there is no numeric acceptance threshold for this step, so validation is structural rather than statistical.

## Related tools

- **Jellyfish** (Counts k-mers over the training corpus with stride 1 in canonicalization mode (-C) so each k-mer and its reverse complement collapse to a single canonical form)
- **DeepMicrobes** (Consumes the vocabulary-to-index mapping to back the k-mer embedding layer used during DNN training; vocabulary construction is part of its training workflow) — https://github.com/MicrobeLab/DeepMicrobes

## Examples

```
jellyfish count -C -m 12 -s 100M -t 8 -o kmers.jf training_genomes.fa
```

## Evaluation signals

- The vocabulary file exists and is a plain-text k-mer-to-index mapping
- Index 0 is reserved for the padding token
- An <unk> symbol is present in the vocabulary and is used for any k-mer containing ambiguous N bases
- The listed canonical k-mers are sorted in lexical order, and each k-mer maps to the same entry as its reverse complement (canonicalization check)
- The chosen k-mer length falls within the 6-12 bp range discussed in the paper; Table S7 reports the effect of k-mer length on vocabulary size as a sanity reference

## Limitations

- The k-mer length used in the study is optimized for the data volume of thousands of genomes; strongly different data volumes may require trying different k-mer lengths (e.g. 6-12 bp)
- No numeric acceptance threshold exists for this step, so validation is limited to structural QC (index 0 = padding, <unk> present, lexical ordering)
- Adding new species requires retraining the entire deep neural network, so extending the vocabulary means rebuilding and retraining rather than incremental updates

## Evidence

- [methods] Jellyfish canonicalization and stride-1 counting: [evidence span withheld — anchor status: not_found]
- [methods] <unk> handling of N-containing k-mers: [evidence span withheld — anchor status: not_found]
- [methods] Vocabulary built with Jellyfish and <unk> inclusion: [evidence span withheld — anchor status: not_found]
- [discussion] K-mer length guidance for other scenarios: [evidence span withheld — anchor status: not_found]
