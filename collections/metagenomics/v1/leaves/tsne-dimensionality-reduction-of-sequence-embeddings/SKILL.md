---
name: tsne-dimensionality-reduction-of-sequence-embeddings
description: Use when you have per-sequence embeddings from a trained classifier checkpoint
  and need to test whether the representation space is taxonomically structured at
  a given rank (e.g., species or phylum).
when_to_use_negative:
- You need classification predictions or accuracy/F1/ECE metrics rather than an embedding-space
  qualitative/structural check — use deeptaxa predict instead
- Input is raw FASTQ/FASTA sequence data and no trained checkpoint is available to
  produce embeddings
- You need to compare performance across alternative reference databases (SILVA, RDP)
  or independent datasets, which the paper does not evaluate
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Provides the trained HybridCNNBERTClassifier checkpoint and fused-representation
    extraction used as t-SNE input
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Loads the pickle-based .pt checkpoint and runs forward passes to extract 896-dim
    fused embeddings
  repo: https://github.com/systems-genomics-lab/deeptaxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:b4eb537b03c6628063d1b14a5a8e0c0c7a1765e100b49f0810934b4abf2831e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - PyTorch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# t-SNE dimensionality reduction of sequence embeddings

## Summary

Projects high-dimensional neural embeddings (e.g., DeepTaxa's 896-dim fused CNN-BERT representations of 16S rRNA sequences) to 2D with t-SNE to visually and quantitatively assess whether embeddings cluster by taxonomic label.

## When to use

Use when you have per-sequence embeddings from a trained classifier checkpoint and need to test whether the representation space is taxonomically structured at a given rank (e.g., species or phylum).

## When NOT to use

- You need classification predictions or accuracy/F1/ECE metrics rather than an embedding-space qualitative/structural check — use deeptaxa predict instead
- Input is raw FASTQ/FASTA sequence data and no trained checkpoint is available to produce embeddings
- You need to compare performance across alternative reference databases (SILVA, RDP) or independent datasets, which the paper does not evaluate

## Inputs

- Trained DeepTaxa HybridCNNBERT checkpoint (.pt) from the Hugging Face repository
- Greengenes2 2024.09 test sequences (gg_2024_09_testing.fna.gz, 69,335 sequences)
- Greengenes2 2024.09 test taxonomy labels (gg_2024_09_testing.tsv.gz) for rank-specific integer labels
- Per-sequence fused embeddings (896-dim CNN-projected features + BERT [CLS] vector)

## Outputs

- 2D t-SNE projection of test-sequence embeddings
- Species-level silhouette score on the 2D projection (expected 0.7377)
- Phylum-level silhouette score on the 2D projection (expected near zero)
- 2D scatter plot colored by species and by phylum labels

## How to apply

Load a published checkpoint (e.g., the full-length HybridCNNBERT model, 76.4 M parameters, final epoch-10) with PyTorch, then run forward passes over the Greengenes2 2024.09 test FASTA (69,335 sequences) after DNABERT-2 BPE tokenization truncated/padded to 512 tokens, extracting the fused representation (CNN features projected to 896 dimensions combined with the BERT [CLS] vector). Project the extracted embeddings to 2D with t-SNE (sklearn.manifold.TSNE, n_components=2). Compute silhouette scores on the 2D projection using rank-specific integer labels — once with species labels and once with phylum labels — to complement the visual inspection. Interpret compact, well-separated species clusters as evidence of taxonomic structure; expect only partial phylum separation with overlap among less-abundant phyla.

## Related tools

- **DeepTaxa** (Provides the trained HybridCNNBERTClassifier checkpoint and fused-representation extraction used as t-SNE input) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Loads the pickle-based .pt checkpoint and runs forward passes to extract 896-dim fused embeddings) — https://github.com/systems-genomics-lab/deeptaxa

## Examples

```
from sklearn.manifold import TSNE; emb2d = TSNE(n_components=2).fit_transform(fused_embeddings)
```

## Evaluation signals

- Species-level silhouette score on the 2D embedding matches the reported value of 0.7377
- Phylum-level silhouette score is near zero, consistent with partial phylum separation
- Species t-SNE plot shows abundant species forming compact clusters while less-abundant phyla overlap
- Extracted fused vectors are 896-dimensional (CNN features projected to 896 dims combined with the BERT [CLS] vector)
- Embedding extraction covers all 69,335 Greengenes2 2024.09 test sequences

## Limitations

- Full-length checkpoint evaluation was trained/evaluated only on Greengenes2; transfer of embedding structure to other databases is untested
- Phylum-level structure is only partially resolved: the phylum projection shows overlap among less-abundant phyla, so phylum silhouette scores near zero are expected rather than a failure
- t-SNE projections are stochastic; results depend on initialization and perplexity, so verify with silhouette scores rather than visual layout alone

## Evidence

- [other] Run forward passes with the checkpoint to extract the fused representation (CNN features projected to 896 dimensions and combined with the BERT [CLS] vector): [evidence span withheld — anchor status: not_found]
- [other] Validation: the species-level silhouette score must match the reported value of 0.7377 and the phylum-level score must be near zero.: [evidence span withheld — anchor status: not_found]
- [readme] HybridCNNBERTClassifier (default): Fuses CNN local motif features with BERT global context.: [evidence span withheld — anchor status: external_doc]
