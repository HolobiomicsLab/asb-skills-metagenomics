---
name: protein-sequence-kmer-vectorization
description: Use when raw protein FASTA sequences must be converted to fixed-length
  feature representations for classification, clustering, or search, and you want
  to test whether amino acid recoding plus k-mering outperforms naive unencoded k-mer
  features.
when_to_use_negative:
- Input is already a numeric feature table or embedding; k-mer vectorization is only
  needed for raw sequence FASTA input
- Your comparison requires substitution-sensitive similarity (e.g. alignment or HMM
  scores) rather than exact-match k-mer sets, which the article notes may be 'limited
  in terms of sensitivity and coverage'
- Very short sequences where the chosen k (e.g. k=14) approaches or exceeds sequence
  length, yielding degenerate k-mer sets
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0623
tools:
- name: Snekmer
  role: Recodes proteins into AAR k-mer vectors and runs the model/cluster/vectorize
    stages via Snakemake
  repo: github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Orchestrates the vectorization workflow; arguments passed through directly
    to Snakemake
  repo: github.com/PNNL-CompBio/Snekmer
- name: bsf-jaccard-py
  role: Blazing Signature Filter Jaccard similarity used downstream when clustering
    the k-mer vectors of large datasets
  repo: github.com/PNNL-CompBio/bsf-jaccard-py
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:32dcdfd69ae54e1ba43c67590fef76af081c229ebf06933dd258c5b95719ed95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blazing Signature Filter
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# protein-sequence-kmer-vectorization

## Summary

Recode protein sequences into amino-acid-reduced (AAR) k-mer feature vectors with Snekmer, sweeping reduced alphabets (e.g. MIQS size 10, standard chemistry size 7, size-20 no-recoding) and k-mer lengths k=4-16 to build reusable features for supervised modeling or clustering.

## When to use

Use when raw protein FASTA sequences must be converted to fixed-length feature representations for classification, clustering, or search, and you want to test whether amino acid recoding plus k-mering outperforms naive unencoded k-mer features.

## When NOT to use

- Input is already a numeric feature table or embedding; k-mer vectorization is only needed for raw sequence FASTA input
- Your comparison requires substitution-sensitive similarity (e.g. alignment or HMM scores) rather than exact-match k-mer sets, which the article notes may be 'limited in terms of sensitivity and coverage'
- Very short sequences where the chosen k (e.g. k=14) approaches or exceeds sequence length, yielding degenerate k-mer sets

## Inputs

- Protein sequences in FASTA format (e.g. nitrogen cycling family FASTA files or UniProt-derived proteomes placed in the Snekmer input/ directory)
- Snekmer config.yaml specifying 'k', 'alphabet', 'cv', and 'random_state'

## Outputs

- AAR k-mer feature vectors per sequence (produced by the vectorize stage)
- Cross-validation ROC AUC / PR AUC reports per alphabet x k combination (model mode)
- Per-family ROC AUC values and cluster assignment tables (cluster mode)
- Timing table such as snekmer_feature_timing.csv with per-parameterization runtimes and the MIQS-vs-unencoded speedup ratio

## How to apply

Place protein FASTA files (e.g. nitrogen cycling families or a 10-genome UniProt set) into the Snekmer input directory and set parameters in resources/config.yaml: 'k' (integer, e.g. 4-16), 'alphabet' (one of 0=hydro, 1=standard, 2=solvacc, 3=hydrocharge, 4=hydrostruct, 5=miqs, or None for unencoded size-20), 'cv': 5 for model mode, and a fixed 'random_state' for reproducibility. Run the Snakemake-driven pipeline via 'snekmer model' or 'snekmer cluster'; use 'snekmer {mode} --until vectorize' to run only the feature-generation stage when timing or reusing vectors. AAR recoding shrinks the k-mer alphabet (e.g. MIQS collapses 20 residues to 10), which links distant sequence similarity and makes feature generation roughly 4x faster than unencoded 4-mers at comparable model quality. For clustering of very large sets, pair the vectors with the agglomerative-jaccard method (complete linkage, dist_thresh 95); the best empirical parameterizations in the article were MIQS k=4 (supervised) and hydrophobicity k=14 (10-genome clustering).

## Related tools

- **Snekmer** (Recodes proteins into AAR k-mer vectors and runs the model/cluster/vectorize stages via Snakemake) — github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Orchestrates the vectorization workflow; arguments passed through directly to Snakemake) — github.com/PNNL-CompBio/Snekmer
- **bsf-jaccard-py** (Blazing Signature Filter Jaccard similarity used downstream when clustering the k-mer vectors of large datasets) — github.com/PNNL-CompBio/bsf-jaccard-py

## Examples

```
snekmer model --until vectorize  # with resources/config.yaml: k: 4, alphabet: 5 (miqs)
```

## Evaluation signals

- Supervised check: with 7 alphabets x 13 k-mer lengths (k=4-16), all cross-validation ROC AUC and PR AUC entries are finite, and MIQS at k=4 plus Standard at k=8 attain ROC AUC = 1.0
- Clustering check: per-family ROC AUC against HMM ground-truth labels averages > 0.9 in more than half of alphabet/k-mer combinations
- Timing check: MIQS-recoded 4-mer vectorization completes approximately 4x faster than unencoded (size-20) 4-mer vectorization, matching Supplementary Table S2
- Downstream comparison check: hydrophobicity AAR with k=14 yields the best V-score (harmonic mean of homogeneity and completeness) among the three benchmarked parameterizations on the 46,908-protein set

## Limitations

- The k-mer representation is exact-match and 'may be limited in terms of sensitivity and coverage'; AAR recoding is the mechanism that links distant sequence similarity
- Large-scale Jaccard clustering of vectors benefits from the BSF C++ extension (requires GCC 4.9+ / OpenMP); without it the workflow falls back to scipy.spatial.distance.pdist
- Best parameters are dataset-dependent: MIQS k=4 was best for nitrogen-cycling supervised models while hydrophobicity k=14 was best for 10-genome clustering, so a parameter sweep is required rather than reusing a single setting

## Evidence

- [abstract] Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either"
- [readme] Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach.: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach."
- [supplementary] Models built from MIQS-recoded 4-mers perform comparably to unencoded 4-mers, but the MIQS-recoded feature generation step is nearly 4 times faster: [evidence span withheld — anchor status: not_found]
- [readme] There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
