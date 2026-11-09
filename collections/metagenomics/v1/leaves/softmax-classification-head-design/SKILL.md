---
name: softmax-classification-head-design
description: Use when building or reusing a deep learning classifier (e.g. DeepMicrobes-style
  Embed + LSTM + Attention) that must map a fixed-size aggregated sequence representation
  to normalized probabilities over taxonomic labels.
when_to_use_negative:
- The backbone output is already a probability distribution or the task is regression
  rather than multiclass taxonomic classification
- Sequence representation is length-dependent (e.g. raw LSTM outputs per timestep)
  — aggregate via self-attention (M = H·A) first, since the MLP head expects the length-independent
  2*lstm-by-r matrix
- 'Predicting taxa not present in training: adding new species requires retraining
  the entire deep neural network'
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0659
tools:
- name: DeepMicrobes
  role: Framework providing the embedding + BiLSTM + self-attention backbone and the
    three-layer ReLU MLP with softmax classification head, plus training and prediction
    scripts
  repo: https://github.com/MicrobeLab/DeepMicrobes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa009
    title: 'DeepMicrobes: taxonomic classification for metagenomics with deep learning'
  source_content_hash: sha256:2ee4b75b4a9a43367a4955fcd73632a90ac2df4114e5137e583b7fbd52b7e49e
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# softmax-classification-head-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Design of the classification head in DeepMicrobes' embedding-based BiLSTM self-attention model: the length-independent attention-weighted matrix M = H·A (2*lstm by r) is passed through a three-layer fully connected ReLU MLP whose output activations are converted to taxonomic class probabilities by a softmax layer.

## When to use

Use when building or reusing a deep learning classifier (e.g. DeepMicrobes-style Embed + LSTM + Attention) that must map a fixed-size aggregated sequence representation to normalized probabilities over taxonomic labels.

## When NOT to use

- The backbone output is already a probability distribution or the task is regression rather than multiclass taxonomic classification
- Sequence representation is length-dependent (e.g. raw LSTM outputs per timestep) — aggregate via self-attention (M = H·A) first, since the MLP head expects the length-independent 2*lstm-by-r matrix
- Predicting taxa not present in training: adding new species requires retraining the entire deep neural network

## Inputs

- Attention-weighted hidden state matrix M = H·A of shape (2*lstm, r) from the BiLSTM + self-attention backbone
- Taxonomic label set (e.g. 2,505 human gut bacterial species from the training repertoire)
- Model hyperparameters (hidden sizes, r, MLP layer widths) from the hyperparameter search space (Table S8)

## Outputs

- Probability-normalized softmax vector over taxonomic classes per input read
- Trained three-layer ReLU MLP classification head with trained model parameters saved for prediction

## How to apply

After the bidirectional LSTM produces hidden state matrix H, compute the r-row self-attention A = softmax(Ws2 tanh(H^T Ws1)) and the attention-weighted aggregation M = H·A, whose shape (2*lstm by r) is independent of input read length. Feed M into a three-layer fully connected classifier with ReLU activations, then apply a softmax function that converts the output activations to class probabilities over the taxonomic labels (e.g. 2,505 gut species). Hyperparameters for each model and layer are selected from the search space documented in Table S8, and implementation code is in the MicrobeLab/DeepMicrobes repository. Validate by instantiating the assembled module and running a forward pass on a dummy sequence to confirm the tensor flows embedding → BiLSTM → self-attention → MLP and that the softmax output is probability-normalized with the correct shape.

## Related tools

- **DeepMicrobes** (Framework providing the embedding + BiLSTM + self-attention backbone and the three-layer ReLU MLP with softmax classification head, plus training and prediction scripts) — https://github.com/MicrobeLab/DeepMicrobes

## Examples

```
python DeepMicrobes/train.py --model embed_lstm_attention --train_tfrecord train.tfrecord --output_dir trained_model/
```

## Evaluation signals

- Forward-pass shape check: output tensor from the MLP has exactly one activation per taxonomic class for a dummy input sequence
- Softmax output rows sum to 1 (probability-normalized) for every input read
- Attention-weighted matrix M has fixed shape 2*lstm-by-r regardless of input sequence length
- Read-level precision and recall on variable-length reads simulated from the 3,269 held-out MAGs (Tables S9–S10) confirm the full architecture (Embed + LSTM + Attention) outperforms ablations
- Trained model reproduces species/genus identification gains over Kraken-style baselines reported in the benchmark datasets

## Limitations

- Adding new species requires retraining the entire deep neural network
- Retraining might be required to adapt to reads whose lengths and error profiles are strongly different (e.g. long-read sequencing)
- The k-mer length used is optimized for typical data volume of thousands of genomes; other scenarios should try different k-mer lengths (e.g. 6-12 bp)

## Evidence

- [full_text] finding: MLP softmax head: [evidence span withheld — anchor status: not_found]
- [full_text] workflow: validation forward pass: [evidence span withheld — anchor status: not_found]
