---
name: embedding-vocabulary-design-for-dna-models
description: 'Use when preparing a DNA/embedding layer for a deep-learning taxonomic
  classifier: you need a fixed, canonical, index-mapped k-mer vocabulary before training
  the embedding.'
when_to_use_negative:
- Your input is already TFRecord-encoded reads — the vocabulary must be built before
  converting fastq/fasta sequences to TFRecord
- You are only running pre-trained DeepMicrobes predictions on a metagenome dataset;
  the shipped model already includes its vocabulary
- You need alignment/k-mer LCA classification (e.g. a Kraken-style database) rather
  than a learned embedding vocabulary
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: DNN framework whose embedding layer consumes the canonical k-mer vocabulary
    for species-level read classification
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: Jellyfish
  role: Counts k-mers over the training FASTA corpus with stride 1 in canonicalization
    mode (-C) to derive the vocabulary
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:f4a3bc6e621957d4209a8b331c881668c059e68cde0f3d36b89dc27542239e7c
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# embedding-vocabulary-design-for-dna-models

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the canonical k-mer vocabulary (with an <unk> token for N-containing k-mers and padding at index 0) that backs the embedding layer of DeepMicrobes' LSTM-based taxonomic classifier. Canonicalization collapses each k-mer with its reverse complement so the vocabulary and embedding table are half-sized and strand-invariant.

## When to use

Use when preparing a DNA/embedding layer for a deep-learning taxonomic classifier: you need a fixed, canonical, index-mapped k-mer vocabulary before training the embedding.

## When NOT to use

- Your input is already TFRecord-encoded reads — the vocabulary must be built before converting fastq/fasta sequences to TFRecord
- You are only running pre-trained DeepMicrobes predictions on a metagenome dataset; the shipped model already includes its vocabulary
- You need alignment/k-mer LCA classification (e.g. a Kraken-style database) rather than a learned embedding vocabulary

## Inputs

- Training genome FASTA files (e.g. 2,505 human gut microbiota reference genomes)
- Chosen k-mer length (6-12 bp range discussed in the paper)
- Jellyfish k-mer counting parameters (hash size, thread count)

## Outputs

- Plain-text vocabulary file mapping canonical k-mers to integer indices (index 0 = padding token, <unk> symbol for N-containing k-mers)
- Vocabulary backing the embedding layer of the DeepMicrobes DNN (Embed + LSTM + Attention model)

## How to apply

Load the training genome FASTA files (e.g. the 2,505 human gut bacterial genomes used for DeepMicrobes species classification) as the counting corpus. Run Jellyfish in canonicalization mode (-C) with stride 1 so each k-mer and its reverse complement collapse to one canonical form, then assign distinct canonical k-mers integer indices in lexical order, reserving index 0 for the padding token. K-mers containing ambiguous bases (N) are not given vocabulary entries; they are mapped to an explicit <unk> symbol. Write the vocabulary-to-index mapping to a plain-text file that the embedding layer reads during DNN training; the paper found that k-mer embedding significantly boosts model performance, and chose k-mer length optimized for thousands of genomes (6-12 bp being reasonable in other scenarios).

## Related tools

- **DeepMicrobes** (DNN framework whose embedding layer consumes the canonical k-mer vocabulary for species-level read classification) — https://github.com/MicrobeLab/DeepMicrobes
- **Jellyfish** (Counts k-mers over the training FASTA corpus with stride 1 in canonicalization mode (-C) to derive the vocabulary)

## Examples

```
jellyfish count -m 12 -s 10M -C genome.fa
```

## Evaluation signals

- Vocabulary file exists and is plain-text with a well-formed k-mer-to-index mapping
- Index 0 is reserved for the padding token
- The <unk> symbol is present and used for k-mers containing Ns
- All listed k-mers are canonical and sorted in lexical order (structural QC; the paper reports no numeric acceptance threshold)
- Vocabulary size matches the expectation for the chosen k (cf. Table S7: 'The effect of k-mer length on vocabulary size')

## Limitations

- The k-mer length used is optimized for thousands of genomes; other scenarios require trying different k-mer lengths (e.g. 6-12 bp)
- Structural QC only — the paper reports no numeric acceptance threshold for vocabulary construction
- Adding new species or adapting to reads with strongly different lengths/error profiles requires retraining the entire DNN, which re-entails rebuilding the vocabulary

## Evidence

- [supplementary] k-mer length vs vocabulary size (Table S7): [evidence span withheld — anchor status: external_doc]
- [discussion] k-mer length choice: [evidence span withheld — anchor status: not_found]
