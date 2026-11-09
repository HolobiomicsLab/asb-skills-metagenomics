---
name: bidirectional-lstm-sequence-modeling
description: Use when encoding variable-length DNA sequencing reads (e.g. FASTQ/FASTA
  metagenomic reads) as dense neural representations where oligonucleotide order and
  context matter for downstream taxonomic classification.
when_to_use_negative:
- Input reads have lengths and error profiles strongly different from the training
  data (e.g. long-read sequencing) — the authors state retraining might be required
- The task requires adding new species to a trained classifier without retraining
  — the authors note adding new species requires retraining the entire deep neural
  network
- A curated reference/taxonomic tree and alignment-based or k-mer database lookup
  (e.g. Kraken-style LCA) is sufficient and training data are unavailable
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Provides the implementation of the Embed + LSTM + Attention architecture,
    trained model parameters, and hyperparameters for taxonomic classification of
    short metagenomics reads
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: MicrobeLab/DeepMicrobes-data
  role: Supplies benchmark datasets, supplementary tables (e.g. Table S8 hyperparameter
    search space), and training reference metadata (2,505 genomes, Table S1)
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:476c7754bf9245cc3721ce5259adcafc075def4d7d2d4eb23f422244078bcf84
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
  - MicrobeLab/DeepMicrobes-data
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bidirectional-lstm-sequence-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs DeepMicrobes' recurrent sequence encoder: a k-mer embedding layer feeding a bidirectional LSTM that captures oligonucleotide order and context for taxonomic classification of short metagenomics sequencing reads. The BiLSTM hidden states are aggregated by multi-head self-attention before classification, which the authors found outperforms CNN-based encoders.

## When to use

Use when encoding variable-length DNA sequencing reads (e.g. FASTQ/FASTA metagenomic reads) as dense neural representations where oligonucleotide order and context matter for downstream taxonomic classification.

## When NOT to use

- Input reads have lengths and error profiles strongly different from the training data (e.g. long-read sequencing) — the authors state retraining might be required
- The task requires adding new species to a trained classifier without retraining — the authors note adding new species requires retraining the entire deep neural network
- A curated reference/taxonomic tree and alignment-based or k-mer database lookup (e.g. Kraken-style LCA) is sufficient and training data are unavailable

## Inputs

- DNA sequencing reads (FASTQ or FASTA, convertible to TFRecord)
- k-mer index encoding of each input sequence
- Collection of training genomes (e.g. the 2,505-species complete bacterial repertoire of the human gut microbiota)

## Outputs

- Bidirectional LSTM hidden state matrix H per input sequence
- Attention-weighted matrix M = H·A of shape 2*d_lstm x r
- Softmax class probabilities over taxonomic labels (species/genus)

## How to apply

Define a k-mer embedding layer that looks up dense embedding vectors from a canonical k-mer index encoding of each input sequence (the paper found k-mer embedding significantly boosts model performance, with k optimized for thousands of genomes; other scenarios should try k-mer lengths of 6-12 bp). Feed the embedding matrix into a bidirectional LSTM to produce hidden state matrix H that captures oligonucleotide order and context. Optionally follow with the r-row self-attention A = softmax(Ws2 tanh(H^T Ws1)) and compute the attention-weighted matrix M = H·A (shape 2*d_lstm by r, independent of input sequence length). Pass M through a three-layer fully connected ReLU MLP and a softmax to obtain class probabilities over taxonomic labels. Hyperparameters were selected from the search space documented in Table S8 of the DeepMicrobes supplement.

## Related tools

- **DeepMicrobes** (Provides the implementation of the Embed + LSTM + Attention architecture, trained model parameters, and hyperparameters for taxonomic classification of short metagenomics reads) — https://github.com/MicrobeLab/DeepMicrobes
- **MicrobeLab/DeepMicrobes-data** (Supplies benchmark datasets, supplementary tables (e.g. Table S8 hyperparameter search space), and training reference metadata (2,505 genomes, Table S1)) — https://github.com/MicrobeLab/DeepMicrobes-data

## Examples

```
python scripts/convert.py --input reads.fastq --output reads.tfrecord; python deepmicrobes/train.py --model Embed-LSTM-Attention --kmer 12 --train_tfrecord reads.tfrecord
```

## Evaluation signals

- Forward-pass sanity check: a dummy sequence flows through embedding → BiLSTM → self-attention → MLP and the softmax output is probability-normalized with the expected number of taxonomic classes
- The attention-weighted matrix M has shape 2*d_lstm by r and is independent of input sequence length
- Read-level precision and recall on variable-length reads simulated from the 3,269 held-out MAGs (Tables S9-S10) should match or approach the reported best-model (Embed + LSTM + Attention) values
- Ablation check: removing k-mer embedding should degrade performance, since the authors report k-mer embedding significantly boosts model performance
- Classifier comparison: the model should outperform Kraken and other state-of-the-art tools in genus/species identification on gut-derived benchmark data

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- The k-mer length used was optimized for typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (e.g. 6-12 bp)
- Training a deep neural network bypasses taxonomic-tree curation but requires a collection of training genomes for the target rank

## Evidence

- [intro] Here we describe DeepMicrobes, a deep learning-based computational framework for taxonomic classification of short metagenomics sequencing reads.: "Here we describe DeepMicrobes, a deep learning-based computational framework for taxonomic classification of short metagenomics sequencing reads."
- [results] In the selected architecture, the attention-weighted matrix M = HA (shape 2dlstm-by-r, independent of input sequence length) is passed through a three-layer fully connected ReLU classifier (MLP) and a softmax function.: [evidence span withheld — anchor status: not_found]
- [discussion] We found that k-mer embedding significantly boosts model performance.: "We found that k-mer embedding significantly boosts model performance."
- [discussion] Our finding that LSTM surpasses CNN highlights the importance of order and context of oligonucleotides in taxonomic classification.: "Our finding that LSTM surpasses CNN highlights the importance of order and context of oligonucleotides in taxonomic classification."
- [supplementary] The DeepMicrobes program, trained model parameters, hyperparameters and the implementation of the other DNN architectures are provided at GitHub: "The DeepMicrobes program, trained model parameters, hyperparameters and the implementation of the other DNN architectures are provided at GitHub"
- [methods] How to convert fastq/fasta sequences to TFRecord: "* [How to convert fastq/fasta sequences to TFRecord]"
