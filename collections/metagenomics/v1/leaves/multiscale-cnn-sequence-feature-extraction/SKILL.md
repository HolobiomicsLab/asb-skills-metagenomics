---
name: multiscale-cnn-sequence-feature-extraction
description: Use when building or reproducing a 16S rRNA taxonomic classifier that
  needs local motif features (e.g., k=3/5/7 convolution windows over BPE-tokenized
  DNA) fused with long-range transformer context.
when_to_use_negative:
- Input is already a feature table or k-mer count matrix rather than raw sequence
  data
- The task requires long-range dependencies only and you intend a BERT-only model
  — the article notes a from-scratch transformer alone underperforms substantially
  at the species rank, but the CNN pathway specifically targets local motif capture
- Input sequences carry base-level quality scores or require quality-aware modeling
  — DeepTaxa operates on FASTA-formatted sequences only
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0632
tools:
- name: DeepTaxa
  role: Provides the multiscale CNN pathway (embed_dim=896, 256 filters, kernels [3,
    5, 7], 1 conv layer) inside the HybridCNNBERTClassifier architecture
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: Supplies the BPE tokenizer and fixed 4096-token subword vocabulary used to
    embed 16S sequences before CNN feature extraction
- name: PyTorch
  role: Implementation framework for the embedding layer, multi-kernel convolutions,
    and feature projection
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:6e08e01d5816535f28dac9a519b64a08c4bff5c3dea2f1a55b58a04095a21ea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Hugging Face Transformers
  - PyTorch
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multiscale-cnn-sequence-feature-extraction

## Summary

Extract local sequence features from 16S rRNA sequences using DeepTaxa's multiscale CNN pathway: 896-dimensional token embeddings over the DNABERT-2 BPE vocabulary convolved with parallel kernels of size 3, 5, and 7 at 256 filters each, then projected back to 896 dimensions for downstream rank-specific classification.

## When to use

Use when building or reproducing a 16S rRNA taxonomic classifier that needs local motif features (e.g., k=3/5/7 convolution windows over BPE-tokenized DNA) fused with long-range transformer context.

## When NOT to use

- Input is already a feature table or k-mer count matrix rather than raw sequence data
- The task requires long-range dependencies only and you intend a BERT-only model — the article notes a from-scratch transformer alone underperforms substantially at the species rank, but the CNN pathway specifically targets local motif capture
- Input sequences carry base-level quality scores or require quality-aware modeling — DeepTaxa operates on FASTA-formatted sequences only

## Inputs

- 16S rRNA gene sequences (FASTA), BPE-tokenized with the DNABERT-2 tokenizer and truncated/padded to 512 tokens
- Token embeddings of dimension 896 over the fixed 4096-token DNABERT-2 BPE vocabulary

## Outputs

- 896-dimensional local CNN feature vectors per sequence, ready for fusion with BERT [CLS] features
- Per-rank softmax predictions when used within the HybridCNNBERTClassifier (seven Linnean rank heads)

## How to apply

Tokenize each input 16S rRNA sequence with the DNABERT-2 BPE tokenizer (fixed 4096-token subword vocabulary) and truncate or pad to 512 tokens. Build 896-dimensional token embeddings and pass them through a single-layer multiscale CNN with parallel kernel sizes 3, 5, and 7 at 256 filters per kernel, which captures local sequence patterns at multiple granularities. Project the concatenated multiscale CNN features back to 896 dimensions so they can be fused (via learnable alpha/beta weights plus an additive BERT residual connection) with the BERT [CLS] representation in the HybridCNNBERTClassifier. Training uses cross-entropy loss with AdamW (batch 64, LR 5e-4, dropout 0.20, 10 epochs); the CNN-only variant (CNNClassifier) is available but yields slightly lower species accuracy. Rationale: convolutional kernels capture local k-mer-like motifs that a from-scratch transformer alone underrepresents at the species rank.

## Related tools

- **DeepTaxa** (Provides the multiscale CNN pathway (embed_dim=896, 256 filters, kernels [3, 5, 7], 1 conv layer) inside the HybridCNNBERTClassifier architecture) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (Supplies the BPE tokenizer and fixed 4096-token subword vocabulary used to embed 16S sequences before CNN feature extraction)
- **PyTorch** (Implementation framework for the embedding layer, multi-kernel convolutions, and feature projection)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type hybridcnnbert --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- Forward pass on a dummy batch of 512-token BPE inputs produces feature maps from all three kernel branches (3, 5, 7) projected to 896 dimensions
- In the full HybridCNNBERTClassifier, exactly seven per-rank output distributions are produced per input
- Parameter count of the full-length HybridCNNBERT checkpoint agrees with the reported 76.4M
- CNN-only ablation (CNNClassifier) trains and reaches slightly lower species accuracy than the hybrid (per ablation design)
- Training metrics JSON shows per-epoch validation accuracy/F1 improving across the 10 epochs (e.g., species accuracy ~92.95% on Greengenes2 2024.09 for the full model)

## Limitations

- A CNN-only variant is not competitive at fine ranks; the paper reports species accuracy benefits require fusion with the transformer branch
- Trained and evaluated primarily on Greengenes2; performance across other reference databases (SILVA, RDP) or independent datasets remains untested
- No incorporation of base-level quality scores — FASTA-only input
- Single stratified train/validation/test split with multiple seeds rather than k-fold cross-validation

## Evidence

- [readme] Multiscale CNN kernels and filters: [evidence span withheld — anchor status: external_doc]
- [methods] BPE tokenization and truncation length: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [abstract] Hybrid CNN-BERT rationale: "DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
- [readme] Transformer-alone underperformance motivating the CNN branch: [evidence span withheld — anchor status: not_found]
- [readme] Training hyperparameters: [evidence span withheld — anchor status: external_doc]
