---
name: allele-frequency-density-clustering
description: Use when you have cellsnp-lite AD/DP allele count matrices from shotgun
  metagenomic samples and need to identify shared informative variants versus background
  noise based on clustering of sample allele frequencies.
when_to_use_negative:
- Input is single-cell mitochondrial data rather than shotgun metagenomic samples
  — the predecessor MQuad uses a binomial mixture model with BIC-based filtering for
  that setting
- You need a statistical significance/informativeness score (e.g., deltaBIC) rather
  than a density-cluster count per variant
- Variants have not been piled up with cellsnp-lite and only raw BAMs or VCFs without
  AD/DP counts are available
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaQuad
  role: Implements the per-variant one-dimensional OPTICS density clustering over
    sample allele frequencies and outputs BIC_params.csv
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Piles up per-sample AD/DP allele counts from aligned shotgun metagenomic reads
    (recommended minMAF = 0.02, minCOUNT = 100) that serve as clustering input
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: MQuad
  role: Predecessor tool that MetaQuad extends; its binomial mixture model was adapted
    into density clustering for metagenomic data
  repo: https://github.com/single-cell-genetics/MQuad
- name: Python
  role: Open-source language in which MetaQuad is implemented; OPTICS clustering is
    performed with scikit-learn
  repo: https://github.com/holab-hku/MetaQuad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:5f15ca1b88acb49d1afba0aa99d7f59ae0bc530631bf043264b30ef5b193f652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MQuad
  - MetaQuad
  - Python
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# allele-frequency-density-clustering

## Summary

Distinguish shared informative variants from sequencing errors and non-polymorphic sites in metagenomic allele count data by running one-dimensional OPTICS density-based clustering (scikit-learn) over per-sample alternate-allele frequencies for each cellsnp-lite variant, flagging variants with >=2 clusters.

## When to use

Use when you have cellsnp-lite AD/DP allele count matrices from shotgun metagenomic samples and need to identify shared informative variants versus background noise based on clustering of sample allele frequencies.

## When NOT to use

- Input is single-cell mitochondrial data rather than shotgun metagenomic samples — the predecessor MQuad uses a binomial mixture model with BIC-based filtering for that setting
- You need a statistical significance/informativeness score (e.g., deltaBIC) rather than a density-cluster count per variant
- Variants have not been piled up with cellsnp-lite and only raw BAMs or VCFs without AD/DP counts are available

## Inputs

- cellsnp-lite output folder with per-sample AD and DP allele count matrices (cellSNP)
- Per-sample alternate-allele frequencies computed as AD/DP for each variant

## Outputs

- BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, number_DP
- Per-variant num_of_clusters labels with informative variants satisfying num_of_clusters >= 2
- Cluster assignments and noise-point labels per variant from OPTICS

## How to apply

Load the cellsnp-lite output folder and compute per-sample alternate-allele frequencies (AD/DP) for every variant. For each variant, run one-dimensional OPTICS density-based clustering over the vector of sample allele frequencies, computing reachability and core distances, extracting clusters, labelling noise points, and enforcing a minSample minimum cluster-size constraint (default recommendation: minSample = 5% of the total sample size). Record num_of_clusters per variant and flag a variant as informative when num_of_clusters >= 2. Rationale: density clustering groups samples with similar allele frequencies so that non-polymorphic sites and sequencing errors fall into a single cluster or are labelled as noise, while shared informative variants produce multiple frequency clusters. Optionally prefilter pileup with cellsnp-lite parameters minMAF = 0.02 and minCOUNT = 100 to reduce computation. Write BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP (count of non-zero-DP samples).

## Related tools

- **MetaQuad** (Implements the per-variant one-dimensional OPTICS density clustering over sample allele frequencies and outputs BIC_params.csv) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Piles up per-sample AD/DP allele counts from aligned shotgun metagenomic reads (recommended minMAF = 0.02, minCOUNT = 100) that serve as clustering input) — https://github.com/single-cell-genetics/cellsnp-lite
- **MQuad** (Predecessor tool that MetaQuad extends; its binomial mixture model was adapted into density clustering for metagenomic data) — https://github.com/single-cell-genetics/MQuad
- **Python** (Open-source language in which MetaQuad is implemented; OPTICS clustering is performed with scikit-learn) — https://github.com/holab-hku/MetaQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- BIC_params.csv exists and contains all required columns: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP
- Every num_of_clusters value is a non-negative integer
- Variants flagged as informative all satisfy num_of_clusters >= 2
- number_DP equals the count of samples with non-zero DP for each variant, and mean_DP/mean_AD are consistent with the input cellsnp-lite matrices
- Cluster sizes respect the minSample minimum (e.g., minSample=5); smaller groups are labelled noise

## Limitations

- minSample should be scaled to the cohort — recommended minSample = 5% of the total sample size; a fixed minSample=5 is inappropriate for very large or very small sample sets
- Increasing cellsnp-lite minMAF/minCOUNT above the recommended minMAF = 0.02 and minCOUNT = 100 is advised to reduce computation time but may drop low-frequency informative variants
- False positives can still occur; MetaQuad reduces false positive SNPs 'without greatly affecting the true positive rate' but does not eliminate them
- Variants with very few covered samples may not form clusters meeting minSample, and OPTICS will label them as noise

## Evidence

- [abstract] MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data.: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites"
- [intro] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [readme] MetaQuad takes the output folder of cellSNP-lite: metaquad -i cellSNP -o Metaquad --minSample=5: "MetaQuad takes the output folder of cellSNP-lite"
- [readme] Pile up variants using cellsnp-lite with minMAF = 0.02 and minCOUNT = 100; a higher value is recommended to reduce computation time.: "with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100. A higher value is recommended to reduce computation time"
- [results] Write BIC_params.csv with the columns variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP (count of non-zero-DP samples).: [evidence span withheld — anchor status: not_found]
