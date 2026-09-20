---
name: simka-jaccard-braycurtis-matrix-generation
description: Use when you need pairwise k-mer-based distance matrices (Jaccard-ab,
  Bray-Curtis) between multiple metagenome read sets, especially to benchmark abundance-aware
  metrics against presence/absence baselines.
when_to_use_negative:
- When abundance-resolved distances are required — presence/absence k-mer metrics
  'greatly diminish the resolution' and compress or saturate differences (e.g., maximum
  distance already between mock 3 and mock 4)
- When datasets are massive and a scalable Hadoop-based implementation is needed —
  the Libra paper's scalable alternative is Libra itself, not Simka
- When the input is already a k-mer count index or feature table rather than raw read
  files
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: Simka
  role: Computes the pairwise Jaccard-ab and Bray-Curtis distance matrices between
    metagenomes that serve as the presence/absence comparator to Libra's cosine-similarity
    distances
- name: Libra
  role: Produces the reference COSINESIMILARITY distance matrices (with NATURAL and
    LOGARITHM weighting) against which Simka's matrices are compared on the mock 1–4
    series
  repo: https://github.com/iychoi/libra
- name: Mash
  role: Provides MinHash-based unique-k-mer sketch distances as a second comparator
    in the same mock-community benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giy165
    title: 'Libra: scalable <i>k-</i>mer–based tool for massive all-vs-all metagenome
      comparisons'
  source_content_hash: sha256:cc83682671729dc9653d156cd140598b888fc260fe43b395ac895336757bd880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iychoi/libra
  tools:
  - Libra
  - Mash
  - Simka
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simka-jaccard-braycurtis-matrix-generation

## Summary

Generate all-vs-all metagenome distance matrices with Simka using its Jaccard-ab and Bray-Curtis k-mer presence/absence metrics, as used in the Libra paper to benchmark Libra's COSINESIMILARITY distance on simulated 454 mock communities (mock 1–4).

## When to use

Use when you need pairwise k-mer-based distance matrices (Jaccard-ab, Bray-Curtis) between multiple metagenome read sets, especially to benchmark abundance-aware metrics against presence/absence baselines.

## When NOT to use

- When abundance-resolved distances are required — presence/absence k-mer metrics 'greatly diminish the resolution' and compress or saturate differences (e.g., maximum distance already between mock 3 and mock 4)
- When datasets are massive and a scalable Hadoop-based implementation is needed — the Libra paper's scalable alternative is Libra itself, not Simka
- When the input is already a k-mer count index or feature table rather than raw read files

## Inputs

- Metagenomic read sets in FASTA/FASTQ format (e.g., simulated 454 mock-community datasets, mock 1–4, at 0.5/1/5/10 M read depths)
- Sample list defining the all-vs-all comparison set

## Outputs

- All-vs-all pairwise distance matrix using Simka Jaccard-ab
- All-vs-all pairwise distance matrix using Simka Bray-Curtis

## How to apply

Run Simka on the FASTA/FASTQ read sets (e.g., the simulated 454 metagenomes derived from the HM-277D staggered mock community, mock 1–4) so it counts shared k-mers and emits all-vs-all distance matrices under both the Jaccard-ab and Bray-Curtis metrics. Use these matrices as comparator baselines: in the Libra paper they are contrasted with Libra's COSINESIMILARITY scoring run under NATURAL and LOGARITHM weighting and with Mash MinHash distances. Judge the run by re-deriving the reported qualitative patterns: Simka distances remain flat across the 0.5/1/5/10 M read-depth series (no measurable change under read duplication) and reach maximum difference between mock 3 and mock 4, because presence/absence k-mer metrics discard read-count information. A correct application preserves these saturating/flat behaviors rather than the stepwise distance increase Libra's logarithmic weighting produces.

## Related tools

- **Simka** (Computes the pairwise Jaccard-ab and Bray-Curtis distance matrices between metagenomes that serve as the presence/absence comparator to Libra's cosine-similarity distances)
- **Libra** (Produces the reference COSINESIMILARITY distance matrices (with NATURAL and LOGARITHM weighting) against which Simka's matrices are compared on the mock 1–4 series) — https://github.com/iychoi/libra
- **Mash** (Provides MinHash-based unique-k-mer sketch distances as a second comparator in the same mock-community benchmark)

## Evaluation signals

- Simka distance matrices show no measurable distance change under read duplication of the same sample
- Simka Jaccard-ab and Bray-Curtis distances stay flat across the 0.5/1/5/10 M sequencing-depth series
- Distances hit maximum difference between mock 3 and mock 4, while Libra with LOGARITHM weighting instead shows a stepwise increase across mock 1–4
- Both the Jaccard-ab and Bray-Curtis matrices cover the full all-vs-all sample set with no missing pairs
- Matrices are consistent with k-mer presence/absence semantics (bounded distances, insensitive to read-count normalization)

## Limitations

- Presence/absence distance metrics greatly diminish the resolution of large-scale comparative analyses — distances compress or saturate (maximum difference already between mock 3 and mock 4)
- Simka distances are flat across varying sequencing depth, so depth-driven abundance changes are not captured
- No concrete Simka CLI parameters are given in the provided context, so exact run configuration must come from Simka's own documentation

## Evidence

- [intro] Simka computes distance matrices: "Recently, Simka [15] was developed to compute a distance matrix between metagenomes"
- [abstract] Presence/absence metrics reduce resolution: "methods commonly implemented to reduce compute time for large datasets, such as data reduction, read count normalization, and presence/absence distance metrics, greatly diminish the resolution"
- [readme] Libra scoring algorithm options include Bray-Curtis, used in the same benchmark context: "s : scoring algorithm. COSINESIMILARITY (default) | BRAYCURTIS | JENSENSHANNON"
