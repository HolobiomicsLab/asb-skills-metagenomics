---
name: pytorch-transformer-encoder-implementation
description: Use when a hybrid sequence classifier needs a from-scratch transformer
  pathway over BPE-tokenized DNA (512-token inputs) whose output is fused with CNN
  features, rather than fine-tuning a pre-trained language model.
when_to_use_negative:
- A pre-trained genome language model (e.g. full DNABERT-2 weights) is to be fine-tuned
  rather than used only for its BPE vocabulary
- Only local motif features are needed and the CNNClassifier variant suffices (faster
  training, slightly lower species accuracy)
- Input is short-read data without quality handling needs beyond FASTA — note the
  model does not incorporate base-level quality scores
edam_operation: http://edamontology.org/operation_3093
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Provides the HybridCNNBERTClassifier implementation embedding the from-scratch
    transformer encoder with learnable fusion weights and seven per-rank heads
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Framework in which the 4-layer transformer encoder and fusion logic are constructed
    and trained
- name: Hugging Face Transformers
  role: Used with PyTorch to build the transformer encoder, drawing only the DNABERT-2
    BPE vocabulary
- name: DNABERT-2
  role: Supplies the BPE tokenizer vocabulary (4096 subword tokens) for encoding 16S
    sequences into 896-dimensional embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:9d1fe329c4690e049c5d332bb435bf8a36fea9562f39355c6c640ae27aeb333f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - Hugging Face Transformers
  - PyTorch
  edam_operation: http://edamontology.org/operation_3093
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pytorch-transformer-encoder-implementation

## Summary

Constructs a transformer encoder from scratch in PyTorch/Hugging Face Transformers (4 layers, 7 heads, hidden size 896, FFN 3584, GELU) over the DNABERT-2 BPE vocabulary, as used in DeepTaxa's HybridCNNBERTClassifier for 16S rRNA taxonomic classification.

## When to use

Use when a hybrid sequence classifier needs a from-scratch transformer pathway over BPE-tokenized DNA (512-token inputs) whose output is fused with CNN features, rather than fine-tuning a pre-trained language model.

## When NOT to use

- A pre-trained genome language model (e.g. full DNABERT-2 weights) is to be fine-tuned rather than used only for its BPE vocabulary
- Only local motif features are needed and the CNNClassifier variant suffices (faster training, slightly lower species accuracy)
- Input is short-read data without quality handling needs beyond FASTA — note the model does not incorporate base-level quality scores

## Inputs

- 16S rRNA gene sequences (FASTA, e.g. Greengenes2 gg_2024_09_training.fna.gz, ~1,500 bp full-length)
- Taxonomy labels TSV covering seven Linnean ranks (e.g. gg_2024_09_training.tsv.gz)
- DNABERT-2 BPE tokenizer (4096-token subword vocabulary)

## Outputs

- Trained HybridCNNBERTClassifier checkpoint (.pt) with ~76.4M parameters for the full-length model
- Per-rank softmax predictions from seven parallel linear classifier heads (temperature T=1.0)
- Per-epoch validation metrics JSON (loss, accuracy, F1, precision, recall at each rank)

## How to apply

Tokenize 16S rRNA sequences with the DNABERT-2 BPE tokenizer (fixed 4096-token subword vocabulary), truncating or padding to 512 tokens. Build the encoder in PyTorch with Hugging Face Transformers using the compact configuration: 4 layers, 7 attention heads, 896 hidden size, 3584-dimensional feedforward intermediate, GELU activations, and random (from-scratch) initialization — DNABERT-2 contributes only its BPE vocabulary, not pre-trained weights. Produce the [CLS] representation and fuse it with multiscale CNN features (kernels 3/5/7 at 256 filters, projected to 896 dimensions) via two learnable scalar weights (alpha/beta) plus an additive BERT residual connection. The rationale is that a from-scratch transformer captures long-range dependencies that the CNN's local motifs miss, while avoiding the cost and domain mismatch of fine-tuning a pre-trained genome model. Train with AdamW (beta1=0.9, beta2=0.999, eps=1e-8, weight decay 0.01), LR 5e-4 with linear warmup-and-decay, batch size 64, dropout 0.20, for 10 epochs, then verify the parameter count matches the reported ~76.4M for the full-length checkpoint.

## Related tools

- **DeepTaxa** (Provides the HybridCNNBERTClassifier implementation embedding the from-scratch transformer encoder with learnable fusion weights and seven per-rank heads) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Framework in which the 4-layer transformer encoder and fusion logic are constructed and trained)
- **Hugging Face Transformers** (Used with PyTorch to build the transformer encoder, drawing only the DNABERT-2 BPE vocabulary)
- **DNABERT-2** (Supplies the BPE tokenizer vocabulary (4096 subword tokens) for encoding 16S sequences into 896-dimensional embeddings)

## Examples

```
deeptaxa train --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_training.fna.gz --taxonomy-file ../deeptaxa-data/greengenes/gg_2024_09_training.tsv.gz --model-type hybridcnnbert --output-dir ../deeptaxa-outputs/
```

## Evaluation signals

- Forward pass on a dummy batch of 512-token BPE inputs yields exactly seven per-rank output distributions
- Total parameter count agrees with the reported 76.4M for the full-length HybridCNNBERT checkpoint
- Species-level accuracy near 92.95% and F1 0.9212 on the Greengenes2 2024.09 held-out test set (69,335 sequences), with cross-seed std ≤0.0004 F1
- Species-level expected calibration error ≈ 0.0242 and F1 above 0.99 from domain through class
- `deeptaxa describe --checkpoint` reports architecture metadata matching the compact configuration (896 hidden, 4 layers, 7 heads, FFN 3584)

## Limitations

- A from-scratch transformer on its own (BERTClassifier) underperforms substantially at the species rank; it is provided mainly for ablation
- Operates on FASTA-formatted sequences only; base-level quality scores are not incorporated
- Trained and evaluated primarily on Greengenes2; generalization to SILVA, RDP, or independent environmental/clinical datasets remains untested

## Evidence

- [abstract] transformer trained from scratch on the DNABERT-2 BPE vocabulary: "We present DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
- [methods] BERT architecture and tokenization details: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] Fusion mechanism specification: [evidence span withheld — anchor status: external_doc]
- [readme] BERT encoder configuration: [evidence span withheld — anchor status: external_doc]
- [readme] Ablation caveat on transformer-only variant: [evidence span withheld — anchor status: external_doc]
