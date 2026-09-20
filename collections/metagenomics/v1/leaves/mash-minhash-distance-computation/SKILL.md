---
name: mash-minhash-distance-computation
description: Use when computing fast all-vs-all pairwise distances between many metagenomic
  FASTA/FASTQ samples where sketching unique k-mers is acceptable, or when benchmarking
  abundance-aware k-mer metrics against Mash.
when_to_use_negative:
- The analysis goal requires abundance-resolved distances — the paper shows that data
  reduction approaches like unique-k-mer sketching 'greatly diminish the resolution'
  of large-scale comparative analyses (Mash distances compress across mock 1–4).
- Input is already a pairwise distance matrix or feature table rather than raw metagenomic
  reads.
- No sketching approximation is acceptable and exact k-mer abundance distances are
  required (use Libra scoring instead).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Mash
  role: Computes MinHash-based pairwise distances from unique k-mer sketches of the
    mock-community metagenomes, serving as the fast comparison baseline to Libra.
- name: Libra
  role: Provides the k-mer abundance distance matrices (COSINESIMILARITY scoring with
    NATURAL and LOGARITHM weighting) against which Mash distances are compared.
  repo: https://github.com/iychoi/libra
- name: Simka
  role: Provides Jaccard-ab and Bray-Curtis pairwise distance matrices as an additional
    comparison point for the mock-community series.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:4aa5ef18f1c9e17bf29fad88056b3893f780bc20cb9565cf76842c7eb3e19bb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Libra
  - Mash
  - Simka
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mash-minhash-distance-computation

## Summary

Generates MinHash-based pairwise distances between metagenomes by indexing samples with sketches of unique k-mers, used in the Libra paper as a fast comparison baseline against Libra's k-mer abundance distances on simulated 454 mock-community data.

## When to use

Use when computing fast all-vs-all pairwise distances between many metagenomic FASTA/FASTQ samples where sketching unique k-mers is acceptable, or when benchmarking abundance-aware k-mer metrics against Mash.

## When NOT to use

- The analysis goal requires abundance-resolved distances — the paper shows that data reduction approaches like unique-k-mer sketching 'greatly diminish the resolution' of large-scale comparative analyses (Mash distances compress across mock 1–4).
- Input is already a pairwise distance matrix or feature table rather than raw metagenomic reads.
- No sketching approximation is acceptable and exact k-mer abundance distances are required (use Libra scoring instead).

## Inputs

- Simulated 454 metagenomic read datasets (FASTA/FASTQ) from the staggered HM-277D mock community (mock 1–4)
- Same multi-sample set used for Libra k-mer indexing and Simka comparison

## Outputs

- Mash MinHash-based all-vs-all pairwise distance matrix between samples

## How to apply

Run Mash on the same sample set used for Libra scoring (here, simulated 454 metagenomes derived from the staggered HM-277D mock community, mock 1–4) to obtain MinHash-based pairwise distances from sketches of unique k-mers. Mash indexes each sample by its unique k-mers, so read count information is reduced and distances compress relative to Libra's COSINESIMILARITY scoring with LOGARITHM weighting. Compare the Mash distance matrix against Libra matrices produced with `-w NATURAL` and `-w LOGARITHM` and against Simka Jaccard-ab and Bray-Curtis matrices to assess how each metric resolves the composition series. A correct application reproduces the paper's qualitative pattern: Mash distances are compressed across mock 1–4, whereas Libra logarithmic weighting shows a stepwise distance increase.

## Related tools

- **Mash** (Computes MinHash-based pairwise distances from unique k-mer sketches of the mock-community metagenomes, serving as the fast comparison baseline to Libra.)
- **Libra** (Provides the k-mer abundance distance matrices (COSINESIMILARITY scoring with NATURAL and LOGARITHM weighting) against which Mash distances are compared.) — https://github.com/iychoi/libra
- **Simka** (Provides Jaccard-ab and Bray-Curtis pairwise distance matrices as an additional comparison point for the mock-community series.)

## Evaluation signals

- Mash pairwise distances across the mock 1–4 composition series are compressed relative to Libra's logarithmically weighted distances, which show a stepwise increase.
- Distances are invariant to read duplication (no measurable distance change when reads are duplicated).
- The output is a complete symmetric all-vs-all distance matrix over the same sample set used for Libra and Simka runs.
- Cross-metric comparison shows Simka Bray-Curtis/Jaccard and Libra NATURAL weighting saturating at maximum difference between mocks 3 and 4 while Mash compresses — matching the reported Fig. 2 pattern.

## Limitations

- Sketching unique k-mers discards read count/abundance information, compressing distances between compositionally distinct communities (observed on mock 1–4).
- Presence/absence and data-reduction metrics 'greatly diminish the resolution of large-scale comparative analyses' compared with Libra's all-read k-mer abundance approach.

## Evidence

- [intro] Mash indexes samples by unique k-mers, making sketching-based distance insensitive to abundance: "The fastest of these methods, Mash [15], indexes samples by unique k-mers"
- [abstract] Data reduction methods including sketching diminish comparative resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution of"
- [intro] Sourmash uses the same algorithm and distance metric as Mash: "Sourmash [18], a toolkit for manipulating MinHash sketches, uses the same underlying algorithm and distance metric as Mash"
- [readme] Libra scoring options used as the comparison baseline in the README: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
