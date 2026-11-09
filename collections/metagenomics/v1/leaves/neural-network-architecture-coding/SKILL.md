---
name: neural-network-architecture-coding
description: Use when coding or reproducing a DNN that maps variable-length short
  metagenomic sequencing reads to species/genus labels via k-mer embedding -> BiLSTM
  -> multi-head self-attention -> 3-layer MLP, per the DeepMicrobes Model Architectures
  specification.
when_to_use_negative:
- Input is long-read data or reads whose lengths and error profiles differ strongly
  from training data — the paper notes retraining might be required
- Goal is to add new species to a trained model without retraining — the framework
  requires retraining the entire deep neural network when species are added
- A pure k-mer lookup/LCA classifier (e.g. Kraken) suffices and no order/context modeling
  is needed
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepMicrobes
  role: Reference implementation of the k-mer embedding -> BiLSTM -> self-attention
    -> 3-layer MLP classifier; provides trained model parameters, hyperparameters
    and alternative DNN architectures
  repo: https://github.com/MicrobeLab/DeepMicrobes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:88380b49f10950bfd69a3a8740f95c5358b889cd3aa4af94a3662313f5f76e7f
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
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# neural-network-architecture-coding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the DeepMicrobes embedding-based recurrent self-attention classifier: a k-mer embedding layer feeding a BiLSTM, r-row self-attention aggregation (M = HA), and a three-layer fully connected ReLU MLP with softmax that converts the length-independent attention-weighted representation into taxonomic class probabilities.

## When to use

Use when coding or reproducing a DNN that maps variable-length short metagenomic sequencing reads to species/genus labels via k-mer embedding -> BiLSTM -> multi-head self-attention -> 3-layer MLP, per the DeepMicrobes Model Architectures specification.

## When NOT to use

- Input is long-read data or reads whose lengths and error profiles differ strongly from training data — the paper notes retraining might be required
- Goal is to add new species to a trained model without retraining — the framework requires retraining the entire deep neural network when species are added
- A pure k-mer lookup/LCA classifier (e.g. Kraken) suffices and no order/context modeling is needed

## Inputs

- Short metagenomic sequencing reads (fastq/fasta, convertible to TFRecord)
- Canonical k-mer index encoding of each input DNA sequence
- Number of taxonomic classes (e.g. 2505 gut bacterial species)
- Hyperparameters from Table S8 (embedding dimension, 2*dlstm hidden size, attention rank r, MLP layer sizes)

## Outputs

- Softmax-normalized class probabilities over taxonomic labels for each read
- Trained model parameters deployable for metagenomic prediction via the DeepMicrobes pipeline

## How to apply

Define a k-mer embedding layer that looks up dense vectors from a canonical k-mer index encoding of each input sequence; k-mer embedding significantly boosts model performance, and the k-mer length should be tuned (e.g. 6-12 bp) to the data volume. Feed the embedding matrix into a bidirectional LSTM so the hidden state matrix H captures oligonucleotide order and context, which the paper shows outperforms CNNs for this task. Implement the r-row self-attention A = softmax(Ws2 tanh(H^T Ws1)) exactly as specified in the Model Architectures equations, then compute the attention-weighted matrix M = H·A (shape 2dlstm-by-r, independent of input sequence length). Finally pass M through a three-layer fully connected ReLU classifier (MLP) and a softmax function that converts output activations to class probabilities over taxonomic labels. Hyperparameters and the implementation of the other DNN architectures are available in the MicrobeLab/DeepMicrobes GitHub repository (Table S8 lists the hyperparameter search space).

## Related tools

- **DeepMicrobes** (Reference implementation of the k-mer embedding -> BiLSTM -> self-attention -> 3-layer MLP classifier; provides trained model parameters, hyperparameters and alternative DNN architectures) — https://github.com/MicrobeLab/DeepMicrobes

## Examples

```
A = tf.nn.softmax(Ws2 @ tf.tanh(H @ Ws1), axis=-1); M = H @ A; probs = softmax(MLP_3layer_relu(M))
```

## Evaluation signals

- Forward pass on a dummy sequence flows through embedding -> BiLSTM -> self-attention -> MLP and yields an output tensor of shape (batch, n_classes)
- Softmax output is probability-normalized (each read's class probabilities sum to 1.0)
- M = H·A has shape 2dlstm-by-r and is independent of input sequence length, confirming length invariance
- Attention matrix A is the specified r-row form softmax(Ws2 tanh(H^T Ws1)) with rows summing to 1
- Read-level precision/recall on held-out simulated reads matches or approaches the reported best model (Embed + LSTM + Attention, Tables S9-S10)

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads with strongly different lengths and error profiles (e.g. long-read sequencing)
- The k-mer length used was optimized for the typical data volume of thousands of genomes; other scenarios require trying different k-mer lengths (6-12 bp)
- Choosing a different DNN architecture (CNN vs LSTM) changes performance; hyperparameters must be re-adopted per model and per layer

## Evidence

- [methods] attention-weighted MLP and softmax: [evidence span withheld — anchor status: not_found]
- [methods] self-attention formula: [evidence span withheld — anchor status: not_found]
