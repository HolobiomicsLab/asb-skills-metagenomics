---
name: silhouette-score-cluster-structure-evaluation
description: 'Use when you have per-sequence embedding vectors from a trained classifier
  (here: 69,335 Greengenes2 2024.09 test sequences) plus taxonomy labels, and you
  need to test whether the embedding space clusters by taxonomic rank.'
when_to_use_negative:
- Input is raw FASTQ reads or a feature table rather than model-internal embedding
  vectors — silhouette on non-embedding inputs does not answer the representational-structure
  question
- The goal is classification accuracy benchmarking against DADA2, QIIME 2, Kraken
  2, or SINTAX — use the accuracy/F1/ECE evaluation pipeline instead
- Sequences come from a different reference database (SILVA, RDP) or unseen marker
  genes — the article notes DeepTaxa was trained and evaluated primarily on Greengenes2
  and such performance remains untested
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: DeepTaxa
  role: Provides the HybridCNNBERT model and checkpoint from which fused 896-dimensional
    embeddings are extracted for projection and scoring
  repo: https://github.com/systems-genomics-lab/deeptaxa
- name: PyTorch
  role: Loads the published epoch-10 checkpoint and runs the forward passes that extract
    the fused CNN+BERT representation
  repo: https://huggingface.co/systems-genomics-lab/deeptaxa
- name: scikit-learn
  role: Supplies t-SNE projection and the silhouette_score metric computed against
    rank-specific integer labels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbag166
    title: 'DeepTaxa: a hybrid CNN-BERT framework for 16S rRNA taxonomic classification'
  source_content_hash: sha256:6753b358a31ea62b4071ffa8da1e45936bc87987254fd5817a189f4aa696068a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/systems-genomics-lab/deeptaxa
  tools:
  - DeepTaxa
  - PyTorch
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# silhouette-score-cluster-structure-evaluation

## Summary

Quantifies whether deep-learned 16S rRNA embeddings (e.g. DeepTaxa's fused HybridCNNBERT representations) are taxonomically structured by computing silhouette scores on 2D t-SNE projections using rank-specific integer labels at species and phylum levels. Complements qualitative t-SNE inspection with a numeric cluster-cohesion measure.

## When to use

Use when you have per-sequence embedding vectors from a trained classifier (here: 69,335 Greengenes2 2024.09 test sequences) plus taxonomy labels, and you need to test whether the embedding space clusters by taxonomic rank.

## When NOT to use

- Input is raw FASTQ reads or a feature table rather than model-internal embedding vectors — silhouette on non-embedding inputs does not answer the representational-structure question
- The goal is classification accuracy benchmarking against DADA2, QIIME 2, Kraken 2, or SINTAX — use the accuracy/F1/ECE evaluation pipeline instead
- Sequences come from a different reference database (SILVA, RDP) or unseen marker genes — the article notes DeepTaxa was trained and evaluated primarily on Greengenes2 and such performance remains untested

## Inputs

- Published model checkpoint (e.g. deeptaxa-full-length-v2.pt, 76.4 M parameters, epoch-10)
- Held-out test FASTA of 16S rRNA sequences (69,335 Greengenes2 2024.09 test sequences)
- Taxonomy label file (gg_2024_09_testing.tsv.gz) providing species- and phylum-rank labels
- Extracted fused embedding matrix (n_sequences x 896) from the HybridCNNBERT forward passes
- 2D t-SNE projection of the embeddings

## Outputs

- Species-level silhouette score on the 2D embedding (expected 0.7377)
- Phylum-level silhouette score on the 2D embedding (expected near zero)
- 2D t-SNE projection colored by species and by phylum labels

## How to apply

Load the published full-length DeepTaxa HybridCNNBERT checkpoint (76.4 M parameters, final epoch-10 model) from the Hugging Face repository with PyTorch. Tokenize the 69,335 Greengenes2 2024.09 test sequences with the DNABERT-2 BPE tokenizer, truncating or padding each to a fixed length of 512 tokens, then run forward passes to extract the fused representation (CNN features projected to 896 dimensions and combined with the BERT [CLS] vector via learnable alpha/beta weights and a BERT residual connection). Project the extracted embeddings to 2D with t-SNE, then compute the silhouette score on the 2D embedding twice: once with rank-specific integer species labels and once with phylum labels. Decision point: the species-level score should be high (reported value 0.7377) indicating compact, well-separated species clusters for abundant taxa, while the phylum-level score near zero indicates only partial phylum separation with overlap among less-abundant phyla. Report both scores alongside the t-SNE projections to substantiate claims of taxonomic structure.

## Related tools

- **DeepTaxa** (Provides the HybridCNNBERT model and checkpoint from which fused 896-dimensional embeddings are extracted for projection and scoring) — https://github.com/systems-genomics-lab/deeptaxa
- **PyTorch** (Loads the published epoch-10 checkpoint and runs the forward passes that extract the fused CNN+BERT representation) — https://huggingface.co/systems-genomics-lab/deeptaxa
- **scikit-learn** (Supplies t-SNE projection and the silhouette_score metric computed against rank-specific integer labels)

## Examples

```
from sklearn.metrics import silhouette_score; sil_species = silhouette_score(embedding_2d, species_labels); sil_phylum = silhouette_score(embedding_2d, phylum_labels)
```

## Evaluation signals

- Species-level silhouette score on the 2D embedding reproduces the reported value of 0.7377
- Phylum-level silhouette score is near zero, consistent with partial separation and overlap among less-abundant phyla
- Embedding extraction confirms the fused representation has 896 dimensions (CNN features projected to 896 dimensions combined with the BERT [CLS] vector)
- Tokenized inputs are all exactly 512 tokens after truncation/padding
- t-SNE projection shows abundant species forming compact clusters while less-abundant phyla overlap

## Limitations

- Phylum-level structure is only partially resolved: less-abundant phyla overlap in the projection, so a near-zero phylum silhouette means the metric cannot distinguish weak structure from none
- Silhouette is computed on the 2D t-SNE embedding, which distorts high-dimensional distances; scores reflect projected structure, not the raw 896-D space
- Findings are grounded in Greengenes2 2024.09; generalization to other reference databases or independent environmental/clinical datasets remains untested

## Evidence

- [methods] Tokenization and fixed length: "Each input 16S rRNA gene sequence is tokenized into BPE tokens and either truncated or padded to a fixed length of 512 tokens"
- [other] Expected silhouette values: [evidence span withheld — anchor status: not_found]
- [other] Projection step: [evidence span withheld — anchor status: not_found]
- [readme] Fusion architecture: [evidence span withheld — anchor status: not_found]
