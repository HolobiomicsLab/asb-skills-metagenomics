---
name: embedding-extraction-from-hybrid-cnn-bert-model
description: Use when you need internal representations — not just class predictions
  — from DeepTaxa's published HybridCNNBERT checkpoints, e.g. to test whether 16S
  rRNA embeddings from 69,335 Greengenes2 2024.09 test sequences cluster by species
  or phylum.
when_to_use_negative:
- You only need per-rank predictions, accuracy, F1, or ECE on labeled 16S data — use
  the deeptaxa predict CLI instead of extracting internal embeddings.
- Your sequences are short amplicons (V3–V4 ~420 bp or V4 ~253 bp) — the full-length
  checkpoint was trained on full-length ~1,500 bp 16S; use the region-specific v3v4
  or v4 checkpoints for those inputs.
- The sequences come from other reference databases (SILVA, RDP) or independent environmental/clinical
  datasets — DeepTaxa was trained and evaluated primarily on Greengenes2, and cross-database
  embedding behavior remains untested.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Provides the HybridCNNBERTClassifier whose fused CNN+BERT representation is
    extracted and whose published checkpoints are hosted on Hugging Face
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: DNABERT-2
  role: Its BPE tokenizer converts 16S rRNA sequences into fixed-length (512-token)
    token sequences used as model input
- name: PyTorch
  role: Loads the pickle-based .pt checkpoint and runs the batched forward passes
    that yield the fused embedding vectors
- name: scikit-learn
  role: Supplies t-SNE projection to 2D and silhouette score computation against rank-specific
    integer labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:fa2aaa2c6f9eb7b9531fc5f79b8bda748b7aba92089e1cf466e2b7c71dcf8b78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DNABERT-2
  - DeepTaxa
  - PyTorch
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# embedding-extraction-from-hybrid-cnn-bert-model

## Summary

Extract the fused 896-dimensional representation (CNN features projected to 896 dims combined with the BERT [CLS] vector) from the published DeepTaxa HybridCNNBERTClassifier checkpoint, then assess taxonomic structure of Greengenes2 test-set embeddings via t-SNE and rank-specific silhouette scores.

## When to use

Use when you need internal representations — not just class predictions — from DeepTaxa's published HybridCNNBERT checkpoints, e.g. to test whether 16S rRNA embeddings from 69,335 Greengenes2 2024.09 test sequences cluster by species or phylum.

## When NOT to use

- You only need per-rank predictions, accuracy, F1, or ECE on labeled 16S data — use the deeptaxa predict CLI instead of extracting internal embeddings.
- Your sequences are short amplicons (V3–V4 ~420 bp or V4 ~253 bp) — the full-length checkpoint was trained on full-length ~1,500 bp 16S; use the region-specific v3v4 or v4 checkpoints for those inputs.
- The sequences come from other reference databases (SILVA, RDP) or independent environmental/clinical datasets — DeepTaxa was trained and evaluated primarily on Greengenes2, and cross-database embedding behavior remains untested.

## Inputs

- Published full-length HybridCNNBERT checkpoint from Hugging Face (deeptaxa-full-length-v2.pt, 76.4 M parameters, PyTorch pickle format)
- Greengenes2 2024.09 test FASTA (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Greengenes2 2024.09 test taxonomy labels (gg_2024_09_testing.tsv.gz) for rank-specific integer label assignment

## Outputs

- Fused 896-dimensional embedding matrix (one vector per test sequence; CNN-projected features combined with BERT [CLS])
- 2D t-SNE projection of the fused embeddings
- Species-level and phylum-level silhouette scores computed on the 2D embedding

## How to apply

Load the full-length checkpoint (deeptaxa-full-length-v2.pt, 76.4 M parameters, final epoch-10 model) from huggingface.co/systems-genomics-lab/deeptaxa with PyTorch; the checkpoints are pickle-based, so load only from the official repository. Tokenize the 69,335 Greengenes2 2024.09 test FASTA sequences with the DNABERT-2 BPE tokenizer, truncating or padding to a fixed 512 tokens as in training. Run batched forward passes (training used batch size 64 and mixed precision) through HybridCNNBERTClassifier and capture the fused representation: the multiscale CNN features projected to 896 dimensions and combined with the BERT [CLS] vector (hidden=896). Project the extracted embedding matrix to 2D with t-SNE and compute silhouette scores on the 2D embedding twice, once with rank-specific integer species labels and once with phylum labels. Validation anchor: the species-level silhouette must reproduce the reported 0.7377 and the phylum-level score must be near zero.

## Related tools

- **DeepTaxa** (Provides the HybridCNNBERTClassifier whose fused CNN+BERT representation is extracted and whose published checkpoints are hosted on Hugging Face) — https://github.com/systems-genomics-lab/deeptaxa
- **DNABERT-2** (Its BPE tokenizer converts 16S rRNA sequences into fixed-length (512-token) token sequences used as model input)
- **PyTorch** (Loads the pickle-based .pt checkpoint and runs the batched forward passes that yield the fused embedding vectors)
- **scikit-learn** (Supplies t-SNE projection to 2D and silhouette score computation against rank-specific integer labels)

## Examples

```
import torch; ckpt = torch.load('../deeptaxa-data/models/deeptaxa-full-length-v2.pt', map_location='cpu')
```

## Evaluation signals

- Species-level silhouette score on the 2D t-SNE embedding reproduces the reported value of 0.7377; the phylum-level score is near zero.
- Every extracted fused vector is 896-dimensional, matching the CNN embed_dim=896 and BERT hidden=896 architecture.
- All 69,335 test sequences yield exactly one embedding; inputs are uniformly 512 BPE tokens after truncation/padding.
- The phylum-level t-SNE projection shows only partial separation with overlap among less-abundant phyla, while abundant species form compact clusters.
- Checkpoint metadata (deeptaxa describe) confirms the full-length family, seed, and ~76.4 M parameter count before extraction.

## Limitations

- Checkpoint files use PyTorch's pickle-based serialization, so they must be downloaded only from the official Hugging Face repository.
- Sequences are truncated or padded to 512 BPE tokens, so embeddings reflect a fixed-length token window rather than arbitrary-length raw 16S sequence.
- Published performance and embedding structure were established on Greengenes2 2024.09 with a single stratified 80/20 split; no k-fold cross-validation or cross-database (SILVA, RDP) evaluation exists.

## Evidence

- [methods] Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [readme] **HybridCNNBERTClassifier** (default): Fuses CNN local motif features with BERT global context. Used for the published checkpoints.: [evidence span withheld — anchor status: external_doc]
- [abstract] We present DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary: "We present DeepTaxa, a hybrid CNN-BERT framework that pairs a multiscale CNN with a transformer trained from scratch on the DNABERT-2 BPE vocabulary"
