---
name: 16s-rrna-multirank-classification-head-design
description: Use when building or reconstructing a 16S rRNA classifier that must predict
  all seven Linnean ranks (Domain through Species) in a single forward pass, or when
  verifying that a checkpoint exposes exactly seven per-rank output distributions.
when_to_use_negative:
- The task requires only a single taxonomic rank prediction (e.g., species-only assignment)
  rather than the full seven-rank hierarchy the parallel heads produce
- Input is a feature/abundance table or sequence variants rather than raw 16S rRNA
  sequences, or the marker gene is 18S rRNA or shotgun metagenomic data, which the
  article notes remain untested for this framework
- Base-level quality scores must be incorporated — DeepTaxa operates on FASTA-formatted
  sequences only
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3293
tools:
- name: DeepTaxa
  role: Framework whose HybridCNNBERTClassifier implements the seven parallel per-rank
    linear heads with softmax at T=1.0
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Implementation framework for the parallel linear heads, softmax layers, and
    checkpoint serialization (.pt files)
- name: Hugging Face Transformers
  role: Used with PyTorch to build the transformer encoder whose [CLS] representation
    feeds the classification heads
- name: DNABERT-2
  role: Supplies only the BPE vocabulary used to tokenize the 512-token inputs consumed
    by the head stack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:804c52c167430137237e0c607ca5246d9b10f86b4417e252d9a295f964329c70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Hugging Face Transformers
  - PyTorch
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# 16s-rrna-multirank-classification-head-design

## Summary

Design pattern from DeepTaxa for emitting taxonomic predictions at all seven Linnean ranks simultaneously: seven parallel linear classifier heads attached to the fused 896-dim CNN-BERT representation, each with softmax at uniform temperature T=1.0, so one forward pass over a 512-token BPE input yields domain-through-species distributions.

## When to use

Use when building or reconstructing a 16S rRNA classifier that must predict all seven Linnean ranks (Domain through Species) in a single forward pass, or when verifying that a checkpoint exposes exactly seven per-rank output distributions.

## When NOT to use

- The task requires only a single taxonomic rank prediction (e.g., species-only assignment) rather than the full seven-rank hierarchy the parallel heads produce
- Input is a feature/abundance table or sequence variants rather than raw 16S rRNA sequences, or the marker gene is 18S rRNA or shotgun metagenomic data, which the article notes remain untested for this framework
- Base-level quality scores must be incorporated — DeepTaxa operates on FASTA-formatted sequences only

## Inputs

- 16S rRNA gene sequences (FASTA, e.g., gg_2024_09_training.fna.gz) BPE-tokenized with the DNABERT-2 tokenizer (fixed 4096-token vocab), truncated/padded to 512 tokens
- Per-rank taxonomy labels TSV (e.g., gg_2024_09_training.tsv.gz) defining each head's class vocabulary (species head: 16,909 classes for full-length, 8,347 for V3-V4)
- Fused 896-dimensional CNN+BERT representation from HybridCNNBERTClassifier

## Outputs

- Seven per-rank softmax probability distributions (Domain, Phylum, Class, Order, Family, Genus, Species) per input sequence
- Per-rank accuracy, F1, and ECE metrics (species accuracy 92.95-92.96%, species ECE 0.0242 on Greengenes2 2024.09)
- PyTorch checkpoint with weights, label encoders, and ~76.4M parameters (full-length configuration)

## How to apply

Attach seven parallel linear heads to the fused representation produced by learnable alpha/beta weighting of multiscale CNN features (kernels 3, 5, 7 at 256 filters, projected to 896 dims) with the BERT [CLS] vector, plus an additive BERT residual connection. Size each head to its rank's label vocabulary from the Greengenes2 training TSV: the full-length model's species head covers 16,909 species (76.4M total parameters) while the V3-V4 model's covers 8,347 species (75.8M parameters). Apply softmax at uniform temperature T=1.0 to each head and train all heads jointly with cross-entropy (AdamW, LR 5e-4, batch 64, dropout 0.20, 10 epochs). Verify correctness by running a forward pass on a dummy batch of 512-token DNABERT-2 BPE inputs and confirming exactly seven per-rank distributions and a parameter count agreeing with the reported 76.4M for the full-length HybridCNNBERT checkpoint.

## Related tools

- **DeepTaxa** (Framework whose HybridCNNBERTClassifier implements the seven parallel per-rank linear heads with softmax at T=1.0) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Implementation framework for the parallel linear heads, softmax layers, and checkpoint serialization (.pt files))
- **Hugging Face Transformers** (Used with PyTorch to build the transformer encoder whose [CLS] representation feeds the classification heads)
- **DNABERT-2** (Supplies only the BPE vocabulary used to tokenize the 512-token inputs consumed by the head stack)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type hybridcnnbert --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- A forward pass on a dummy batch of 512-token BPE inputs yields exactly seven per-rank output distributions
- Parameter count agrees with the reported 76.4M for the full-length HybridCNNBERT checkpoint (75.8M for V3-V4, reflecting the smaller 8,347-species vocabulary)
- Species-level accuracy near 92.95-92.96% and F1 0.9212 on the Greengenes2 2024.09 held-out test set, with F1 above 0.99 from domain through class
- Cross-seed standard deviation ≤0.0008 F1 at every rank (≤0.0004 F1 in the 5-seed v2 checkpoints)
- Species-level expected calibration error of 0.0242 under the uniform softmax temperature T=1.0

## Limitations

- Head vocabularies are tied to Greengenes2 2024.09 labels; performance across other reference databases (SILVA, RDP) or independent environmental/clinical datasets remains untested
- Species-head size (and hence total parameter count) varies with the label space: 16,909 species for full-length and V4 vs 8,347 for V3-V4 (75.8M parameters)
- Classification is limited to 'Seen' species labels present in the training vocabulary; 'Unseen' labels cannot be emitted by the fixed species head
- Softmax uses a uniform temperature T=1.0 with no per-rank calibration step; fastai-style temperature scaling is not applied

## Evidence

- [methods] Attach seven parallel linear classifier heads, one per Linnean rank, with softmax at uniform temperature T=1.0.: [evidence span withheld — anchor status: not_found]
- [abstract] DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank): "DeepTaxa achieves species-level accuracy of 92.96% and F1 of 0.9212 (3-seed mean; cross-seed standard deviation ≤0.0008 F1 at every rank)"
- [readme] The V3-V4 model is slightly smaller because its per-rank heads cover a smaller species vocabulary (8,347 vs 16,909).: [evidence span withheld — anchor status: not_found]
- [readme] It classifies sequences into all seven taxonomic ranks (Domain through Species) in a single forward pass, achieving 92.95% species-level accuracy: [evidence span withheld — anchor status: external_doc]
