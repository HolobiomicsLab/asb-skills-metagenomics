---
name: hybrid-cnn-bert-parallel-fusion-architecture
description: Use when a task requires capturing both local sequence motifs and long-range
  dependencies of 16S rRNA sequences for hierarchical taxonomic prediction across
  domain-through-species ranks, or when reimplementing/verifying the DeepTaxa architecture.
when_to_use_negative:
- Input is not 16S rRNA sequence data — the article notes evaluation on other marker
  genes (18S rRNA, shotgun metagenomics) remains untested
- Input requires base-level quality-score awareness — the model operates on FASTA
  only and does not incorporate quality scores
- A lightweight fast classifier is needed and long-range context is unnecessary —
  the README notes CNNClassifier is faster to train with only slightly lower species
  accuracy
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0084
tools:
- name: DeepTaxa
  role: Framework providing the HybridCNNBERTClassifier implementation, pretrained
    checkpoints, and CLI (deeptaxa train/predict/describe)
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Underlying deep-learning framework for building and training the CNN, transformer
    encoder, and fusion layer
- name: Hugging Face Transformers
  role: Supplies the DNABERT-2 BPE tokenizer and transformer building blocks for the
    from-scratch encoder
- name: DNABERT-2
  role: Source of the BPE vocabulary (4096-token subword vocab) only; the transformer
    encoder is trained from scratch, not fine-tuned
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:fe57916f91bd15c37b07ca4e6d64ca14f9eee089497c8b448337831a8a3f28fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Hugging Face Transformers
  - PyTorch
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hybrid-cnn-bert-parallel-fusion-architecture

## Summary

Reconstruct the DeepTaxa HybridCNNBERTClassifier: a multiscale CNN (kernels 3/5/7, 256 filters) fused via learnable scalar weights with a from-scratch 4-layer BERT encoder on the DNABERT-2 BPE vocabulary, feeding seven parallel per-rank linear heads for 16S rRNA taxonomic classification at all seven Linnean ranks.

## When to use

Use when a task requires capturing both local sequence motifs and long-range dependencies of 16S rRNA sequences for hierarchical taxonomic prediction across domain-through-species ranks, or when reimplementing/verifying the DeepTaxa architecture.

## When NOT to use

- Input is not 16S rRNA sequence data — the article notes evaluation on other marker genes (18S rRNA, shotgun metagenomics) remains untested
- Input requires base-level quality-score awareness — the model operates on FASTA only and does not incorporate quality scores
- A lightweight fast classifier is needed and long-range context is unnecessary — the README notes CNNClassifier is faster to train with only slightly lower species accuracy

## Inputs

- FASTA-formatted 16S rRNA gene sequences (full-length ~1,500 bp or amplicon windows)
- Taxonomy label TSV files with seven-rank lineage strings (Greengenes2 2024.09 format)
- DNABERT-2 BPE tokenizer vocabulary (4096 subword tokens)
- Model configuration (Table S1 compact config / config.json)

## Outputs

- HybridCNNBERTClassifier module (~76.4M parameters for full-length)
- Seven per-rank softmax probability distributions per input sequence
- Per-epoch checkpoints (.pt) with weights, optimizer/scheduler state, and label encoders
- Per-epoch validation metrics JSON (loss, accuracy, F1, precision, recall at each rank)

## How to apply

Tokenize 16S sequences with the DNABERT-2 BPE tokenizer (fixed 4096-token subword vocab) and truncate/pad to 512 tokens. Build 896-dimensional token embeddings and a one-layer multiscale CNN pathway with kernel sizes 3, 5, and 7 at 256 filters per kernel, projecting CNN features to 896 dimensions. Construct the transformer encoder from scratch (4 layers, 7 attention heads, 896 hidden size, 3584-dim feedforward, GELU), using DNABERT-2 only for the BPE vocabulary. Fuse the CNN features with the BERT [CLS] representation via two learnable scalar weights (alpha/beta) plus an additive BERT residual connection, then attach seven parallel linear classifier heads—one per Linnean rank—with softmax at temperature T=1.0. Verify by running a forward pass on a dummy 512-token batch confirming exactly seven per-rank output distributions and counting parameters to confirm ~76.4M for the full-length checkpoint. Train with cross-entropy loss, AdamW (beta1=0.9, beta2=0.999, eps=1e-8, weight decay=0.01), LR 5e-4 with linear warmup-and-decay, batch size 64, dropout 0.20, mixed precision, for 10 epochs.

## Related tools

- **DeepTaxa** (Framework providing the HybridCNNBERTClassifier implementation, pretrained checkpoints, and CLI (deeptaxa train/predict/describe)) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Underlying deep-learning framework for building and training the CNN, transformer encoder, and fusion layer)
- **Hugging Face Transformers** (Supplies the DNABERT-2 BPE tokenizer and transformer building blocks for the from-scratch encoder)
- **DNABERT-2** (Source of the BPE vocabulary (4096-token subword vocab) only; the transformer encoder is trained from scratch, not fine-tuned)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type hybridcnnbert --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- Forward pass on a dummy batch of 512-token BPE inputs yields exactly seven per-rank output distributions
- Parameter count of the reconstructed full-length checkpoint agrees with the reported 76.4M
- Species-level accuracy/F1 on Greengenes2 2024.09 test set (69,335 sequences) approaches 92.95% accuracy / 0.9212 F1 (3-seed mean; cross-seed std ≤0.0008 F1 at every rank)
- Species-level expected calibration error ≈ 0.0242, with ECE ≤ 0.0145 at every finer rank
- `deeptaxa describe --checkpoint` on a trained checkpoint reports the expected architecture fields (CNN kernels [3,5,7] at 256 filters, BERT 4 layers / 7 heads / hidden 896 / FFN 3584, fusion alpha/beta)

## Limitations

- Trained and evaluated primarily on Greengenes2 2024.09; performance on SILVA, RDP, or independent environmental/clinical datasets remains untested
- Operates on FASTA only and does not incorporate base-level quality scores, which may matter for noisy amplicon data
- A from-scratch BERT-only variant underperforms substantially at the species rank, so the hybrid fusion is essential to the reported accuracy
- No k-fold cross-validation; results rest on a single stratified 80/20 split with three training seeds

## Evidence

- [abstract] DeepTaxa addresses this gap with a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary, producing parallel rank-specific predictions across the seven Linnean ranks.: "DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
- [methods] Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] Fusion via learnable scalar weights plus a residual connection is described in the README architecture table.: [evidence span withheld — anchor status: external_doc]
- [readme] Compact architecture configuration per the README architecture table.: [evidence span withheld — anchor status: external_doc]
- [methods] AdamW hyperparameters for training the reconstructed model.: "The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8, and a weight decay of 0.01."
- [readme] BERT-only ablation is substantially weaker, justifying the hybrid fusion.: [evidence span withheld — anchor status: external_doc]
