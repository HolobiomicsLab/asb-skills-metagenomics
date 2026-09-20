---
name: self-attention-mechanism-implementation
description: 'Use when implementing or reconstructing DeepMicrobes'' Embed + LSTM
  + Attention species classifier: you have k-mer-embedded DNA reads and need a length-independent,
  attention-pooled representation for taxonomic classification.'
when_to_use_negative:
- Reads whose lengths and error profiles differ strongly from training data (e.g.
  long-read sequencing) — the paper notes retraining might be required to adapt
- Classifying species newly added to the database — adding new species requires retraining
  the entire deep neural network
- Abundance estimation-only tasks where a simpler k-mer or alignment classifier already
  suffices and the attention architecture's read-level precision/recall gains are
  not the goal
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Reference implementation of the k-mer embedding -> BiLSTM -> self-attention
    -> 3-layer MLP architecture; training code, trained model parameters and hyperparameters
    are provided in the repo
  repo: https://github.com/MicrobeLab/DeepMicrobes
- name: DeepMicrobes-data
  role: Supplementary tables defining the hyperparameter search space (Table S8) and
    read-level precision/recall benchmarks of the Embed + LSTM + Attention model (Tables
    S9-S12)
  repo: https://github.com/MicrobeLab/DeepMicrobes-data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:68f67fab48c61b64eb4424e1e7eb2652125a4b8f5f63ca3b3adfa32ae4205623
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# self-attention-mechanism-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the r-row self-attention layer of DeepMicrobes (A = softmax(Ws2 tanh(H^T Ws1))) that aggregates a BiLSTM's hidden states into a fixed-size attention-weighted matrix M = H·A, which is then classified by a 3-layer ReLU MLP + softmax over taxonomic labels.

## When to use

Use when implementing or reconstructing DeepMicrobes' Embed + LSTM + Attention species classifier: you have k-mer-embedded DNA reads and need a length-independent, attention-pooled representation for taxonomic classification.

## When NOT to use

- Reads whose lengths and error profiles differ strongly from training data (e.g. long-read sequencing) — the paper notes retraining might be required to adapt
- Classifying species newly added to the database — adding new species requires retraining the entire deep neural network
- Abundance estimation-only tasks where a simpler k-mer or alignment classifier already suffices and the attention architecture's read-level precision/recall gains are not the goal

## Inputs

- DNA sequencing reads converted to TFRecord (fastq/fasta input), k-mer encoded into canonical indices
- k-mer embedding matrix (dense vectors per canonical k-mer)
- Hidden state matrix H from a bidirectional LSTM over the embedding sequence

## Outputs

- Attention-weighted matrix M = H·A of shape 2dlstm-by-r (independent of read length)
- Per-read softmax probability distribution over taxonomic (species/genus) class labels

## How to apply

Encode each input read via a k-mer embedding layer that looks up dense vectors from a canonical k-mer index, then feed the embedding matrix into a bidirectional LSTM to obtain hidden state matrix H capturing oligonucleotide order and context. Compute the r-row self-attention A = softmax(Ws2 tanh(H^T Ws1)) exactly as in the paper's Model Architectures equations, then multiply M = H·A to aggregate recurrent states into a shape 2dlstm-by-r matrix that is independent of input sequence length. Pass M through a three-layer fully connected ReLU classifier (MLP) followed by a softmax that outputs normalized class probabilities over taxonomic labels. Hyperparameters (Ws1/Ws2 dimensions, r, LSTM size) should be selected per Table S8's hyperparameter search space rather than set arbitrarily. The k-mer embedding is essential: the paper found k-mer embedding significantly boosts model performance, and LSTM surpassed CNN, highlighting the importance of oligonucleotide order and context.

## Related tools

- **DeepMicrobes** (Reference implementation of the k-mer embedding -> BiLSTM -> self-attention -> 3-layer MLP architecture; training code, trained model parameters and hyperparameters are provided in the repo) — https://github.com/MicrobeLab/DeepMicrobes
- **DeepMicrobes-data** (Supplementary tables defining the hyperparameter search space (Table S8) and read-level precision/recall benchmarks of the Embed + LSTM + Attention model (Tables S9-S12)) — https://github.com/MicrobeLab/DeepMicrobes-data

## Examples

```
import tensorflow as tf; A = tf.nn.softmax(tf.tensordot(tf.tanh(tf.linalg.matmul(H, Ws1, transpose_b=True)), Ws2, axes=[[-1],[0]]), axis=1); M = tf.linalg.matmul(A, H, transpose_a=True)
```

## Evaluation signals

- M = H·A must have shape 2dlstm-by-r regardless of input read length — run reads of different lengths through and confirm the output matrix shape is invariant
- A forward pass on a dummy sequence must flow embedding -> BiLSTM -> self-attention -> MLP and end in a softmax whose outputs sum to 1 over the taxonomic labels
- Attention rows equal r: verify A = softmax(Ws2 tanh(H^T Ws1)) has exactly r rows as specified in the Model Architectures equations
- Read-level precision and recall on held-out MAG reads (Tables S9-S12) should match or approach the reported best model (Embed + LSTM + Attention)
- Removing the k-mer embedding or replacing BiLSTM with CNN should measurably degrade read-level performance, consistent with the paper's ablation findings

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- The k-mer length used was optimized for the data volume of thousands of genomes; other scenarios should try different k-mer lengths (e.g. 6-12 bp)

## Evidence

- [methods] attention-weighted matrix M = HA is MLP-classified: [evidence span withheld — anchor status: not_found]
- [readme] code availability: [evidence span withheld — anchor status: external_doc]
