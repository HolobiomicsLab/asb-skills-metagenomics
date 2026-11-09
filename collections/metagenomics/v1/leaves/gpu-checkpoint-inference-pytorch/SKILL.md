---
name: gpu-checkpoint-inference-pytorch
description: Use when you need to extract internal fused embeddings or predictions
  from a published DeepTaxa .pt checkpoint with PyTorch, e.g., for embedding-space
  visualization or quantitative cluster-structure analysis.
when_to_use_negative:
- You only need final classification accuracy/F1 on labeled data — use `deeptaxa predict`
  with taxonomy files rather than hand-rolling embedding extraction
- Input sequences are short amplicons (V3-V4 ~420 bp or V4 ~253 bp) — load the matching
  region-specific checkpoint, not the full-length 16S model
- You need k-fold cross-validated embeddings — the article notes only a single stratified
  split with three training seeds exists, so embedding analysis is limited to that
  split
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Framework providing the HybridCNNBERTClassifier checkpoints and the `deeptaxa
    describe`/`predict` CLI used to load and run the model
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Deep-learning runtime used to load the pickle-serialized .pt checkpoints and
    execute GPU forward passes for embedding extraction
- name: DNABERT-2
  role: BPE tokenizer applied to 16S sequences, truncated or padded to 512 tokens
    before inference
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:0d9a011d51683a839aef9f13faf9ede5c15d9a41cbd88696d5f97aae813b9439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - PyTorch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gpu-checkpoint-inference-pytorch

## Summary

Load a published DeepTaxa PyTorch checkpoint (e.g., the 76.4 M-parameter full-length HybridCNNBERT epoch-10 model from Hugging Face) and run forward passes over the 69,335 Greengenes2 2024.09 test sequences to extract fused CNN+BERT [CLS] representations for downstream analysis such as t-SNE projection and silhouette scoring.

## When to use

Use when you need to extract internal fused embeddings or predictions from a published DeepTaxa .pt checkpoint with PyTorch, e.g., for embedding-space visualization or quantitative cluster-structure analysis.

## When NOT to use

- You only need final classification accuracy/F1 on labeled data — use `deeptaxa predict` with taxonomy files rather than hand-rolling embedding extraction
- Input sequences are short amplicons (V3-V4 ~420 bp or V4 ~253 bp) — load the matching region-specific checkpoint, not the full-length 16S model
- You need k-fold cross-validated embeddings — the article notes only a single stratified split with three training seeds exists, so embedding analysis is limited to that split

## Inputs

- PyTorch checkpoint file (.pt) from huggingface.co/systems-genomics-lab/deeptaxa (e.g., deeptaxa-full-length-v2.pt)
- Greengenes2 2024.09 held-out test FASTA (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Taxonomy label TSV (gg_2024_09_testing.tsv.gz) for rank-specific integer labels

## Outputs

- Fused representation matrix (CNN 896-dim features fused with BERT [CLS] vector) for every test sequence
- Per-sequence predicted taxonomic labels at all seven ranks (optional, from the same forward pass)
- Embedding matrix ready for 2D t-SNE projection and silhouette scoring

## How to apply

Download the checkpoint (e.g., deeptaxa-full-length-v2.pt, 76.4 M parameters, trained 10 epochs with AdamW, LR=5e-4, batch size 64) from huggingface.co/systems-genomics-lab/deeptaxa and load it with PyTorch. Tokenize the 69,335 Greengenes2 2024.09 test FASTA sequences with the DNABERT-2 BPE tokenizer, truncating or padding each sequence to a fixed length of 512 tokens. Run batched GPU forward passes to extract the fused representation: CNN features projected to 896 dimensions combined with the BERT [CLS] vector via learnable alpha/beta fusion weights plus a residual connection. Inspect checkpoints with `deeptaxa describe` to confirm architecture metadata before inference; note that v2 checkpoints ship five seeds (42, 123, 456, 789, 1011) plus a default seed-42 model. The resulting embedding matrix can then be projected (e.g., t-SNE to 2D) and scored against rank-specific integer taxonomic labels.

## Related tools

- **DeepTaxa** (Framework providing the HybridCNNBERTClassifier checkpoints and the `deeptaxa describe`/`predict` CLI used to load and run the model) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Deep-learning runtime used to load the pickle-serialized .pt checkpoints and execute GPU forward passes for embedding extraction)
- **DNABERT-2** (BPE tokenizer applied to 16S sequences, truncated or padded to 512 tokens before inference)

## Examples

```
deeptaxa predict --fasta-file ../deeptaxa-data/greengenes/gg_2024_09_testing.fna.gz --checkpoint ../deeptaxa-data/models/deeptaxa-full-length-v2.pt --output-dir ../deeptaxa-outputs/predictions
```

## Evaluation signals

- Species-level silhouette score on the 2D t-SNE projection of extracted embeddings matches the reported 0.7377
- Phylum-level silhouette score is near zero, consistent with partial separation and overlap among less-abundant phyla
- Loaded checkpoint reports 76.4 M parameters for the full-length family (75.8 M for V3-V4) and its `seed` field is self-identifying
- Tokenized inputs are exactly 512 BPE tokens per sequence after truncation/padding
- Number of extracted embeddings equals 69,335 (the full Greengenes2 2024.09 test set)

## Limitations

- Checkpoint files use PyTorch's pickle-based serialization and should only be downloaded from the official Hugging Face repository
- Results depend on seed, hardware, and package versions; the default checkpoint is seed-42 and cross-seed F1 std is small (≤0.0004 at every rank) but nonzero
- Embeddings reflect a single stratified train/validation/test split, not k-fold cross-validation

## Evidence

- [methods] Tokenize and infer on full test set: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [methods] Training hyperparameters: "The model was trained using the AdamW optimizer (Loshchilov and Hutter 2019) with hyperparameters set to β1=0.9, β2=0.999, ε=1×10−8, and a weight decay of 0.01."
- [abstract] Checkpoint availability: "Source code, trained checkpoints for full-length 16S and V3–V4 amplicons, curated datasets, and reproducible workflows are publicly available"
- [readme] Checkpoint serialization warning: [evidence span withheld — anchor status: external_doc]
- [readme] Ensemble seeds available: [evidence span withheld — anchor status: external_doc]
