---
name: model-parameter-count-verification
description: Use when a reimplementation of DeepTaxa's hybrid CNN-BERT classifier
  must be validated against published spec before training or inference reuse.
when_to_use_negative:
- You only need to run inference with an official checkpoint (deeptaxa describe already
  reports checkpoint metadata) rather than validating a from-scratch reimplementation
- You are verifying a CNN-only or BERT-only ablation variant, whose parameter counts
  will not match the 76.4M hybrid figure
- Input is a V3-V4 checkpoint being compared against the full-length 76.4M count —
  its smaller species vocabulary (8,347 vs 16,909 labels) makes it slightly smaller
  at 75.8M
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0654
tools:
- name: DeepTaxa
  role: Source of the HybridCNNBERTClassifier architecture definition and compact
    configuration being reconstructed and verified
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Framework used to instantiate the module and count parameters
- name: Hugging Face Transformers
  role: Provides the transformer encoder components built on the DNABERT-2 BPE vocabulary
- name: DNABERT-2
  role: Supplies only the BPE vocabulary used for the 896-dimensional token embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:433f3da4b97d2c711c0444ca91d6af209188335a19307e49d5e5e7b6155338ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Hugging Face Transformers
  - PyTorch
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# model-parameter-count-verification

## Summary

Verifies that a reconstructed HybridCNNBERTClassifier matches the published DeepTaxa architecture by instantiating it with the compact Table S1 configuration, running a dummy 512-token forward pass, counting parameters, and checking agreement with the reported 76.4M for the full-length checkpoint.

## When to use

Use when a reimplementation of DeepTaxa's hybrid CNN-BERT classifier must be validated against published spec before training or inference reuse.

## When NOT to use

- You only need to run inference with an official checkpoint (deeptaxa describe already reports checkpoint metadata) rather than validating a from-scratch reimplementation
- You are verifying a CNN-only or BERT-only ablation variant, whose parameter counts will not match the 76.4M hybrid figure
- Input is a V3-V4 checkpoint being compared against the full-length 76.4M count — its smaller species vocabulary (8,347 vs 16,909 labels) makes it slightly smaller at 75.8M

## Inputs

- Compact model configuration (Table S1 / config.) specifying embed_dim=896, CNN kernels [3,5,7] at 256 filters, BERT 4 layers / 7 heads / FFN 3584
- DNABERT-2 BPE tokenizer (fixed 4096-token subword vocabulary)
- Published parameter counts to compare against (76.4M full-length, 75.8M V3-V4, 76.4M V4)
- Dummy batch of 512-token BPE-tokenized 16S sequences

## Outputs

- Total learnable parameter count of the instantiated HybridCNNBERTClassifier
- Seven per-rank softmax output distributions from a dummy forward pass
- architecture_verification report recording parameter-count agreement

## How to apply

Build the model from the compact configuration: 896-dimensional token embeddings over the DNABERT-2 BPE vocabulary (4096-token subword vocab, inputs truncated/padded to 512 tokens), a multiscale CNN with kernel sizes 3, 5, and 7 at 256 filters each projected to 896 dimensions, and a from-scratch 4-layer transformer with 7 attention heads, 896 hidden size, and 3584-dimensional feedforward. Fuse CNN features with the BERT [CLS] representation via two learnable scalar weights plus an additive BERT residual, then attach seven parallel linear classifier heads (one per Linnean rank) with softmax at temperature T=1.0. Run a forward pass on a dummy batch of 512-token BPE inputs and assert exactly seven per-rank output distributions are returned. Sum all learnable parameters with a parameter counter and compare against the reported checkpoint sizes (76.4M for the full-length and V4 models; 75.8M for V3-V4, whose species head covers only 8,347 species labels versus 16,909). Discrepancies indicate wrong vocab size, wrong number of heads, wrong rank-head label spaces, or a mis-specified fusion module.

## Related tools

- **DeepTaxa** (Source of the HybridCNNBERTClassifier architecture definition and compact configuration being reconstructed and verified) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Framework used to instantiate the module and count parameters)
- **Hugging Face Transformers** (Provides the transformer encoder components built on the DNABERT-2 BPE vocabulary)
- **DNABERT-2** (Supplies only the BPE vocabulary used for the 896-dimensional token embeddings)

## Examples

```
deeptaxa describe --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt
```

## Evaluation signals

- sum(p.numel() for p in model.parameters()) agrees with 76.4M (± rounding) for the full-length hybrid configuration
- Dummy forward pass on 512-token BPE inputs returns exactly seven output tensors, one per Linnean rank (domain through species)
- Each per-rank output is a normalized softmax distribution (rows sum to 1.0) at temperature T=1.0
- Embedding dimension is 896 and CNN features are projected to 896 dimensions so fusion with the BERT [CLS] vector is shape-consistent
- CNN pathway uses kernel sizes 3, 5, and 7 with 256 filters per kernel before projection

## Limitations

- Parameter count alone cannot verify learned behavior — a from-scratch BERT-only transformer 'underperforms substantially at the species rank', so architectural agreement does not guarantee accuracy parity
- The V3-V4 family has a smaller parameter count (75.8M) due to its reduced species label space (8,347 vs 16,909), so a single expected count does not apply to all regions
- Published species accuracy differs slightly between the article (92.96%, 3-seed mean) and README (92.95%, 5-seed mean), so seed count and seed values (42, 123, 456, 789, 1011) affect reproduction

## Evidence

- [other] compact config check: [evidence span withheld — anchor status: not_found]
- [other] forward-pass check: [evidence span withheld — anchor status: not_found]
- [other] parameter-count agreement: [evidence span withheld — anchor status: not_found]
- [readme] architecture table: [evidence span withheld — anchor status: external_doc]
- [readme] V3-V4 size difference: [evidence span withheld — anchor status: external_doc]
- [readme] from-scratch transformer caveat: [evidence span withheld — anchor status: external_doc]
