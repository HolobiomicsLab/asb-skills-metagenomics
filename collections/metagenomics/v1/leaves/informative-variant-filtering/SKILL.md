---
name: informative-variant-filtering
description: Use when you have a cellsnp-lite output folder (cellSNP) with per-sample
  AD/DP allele counts from shotgun metagenomic samples and need to distinguish shared
  informative variants from non-polymorphic sites or sequencing-error noise.
when_to_use_negative:
- Input is single-cell mitochondrial mutation data for clonal substructure inference
  — use MQuad (MetaQuad's predecessor), which applies a binomial mixture model instead
  of the metagenomic OPTICS workflow
- Reads are not yet filtered and aligned to a reference database — this skill requires
  an upstream cellsnp-lite pile-up step (minMAF = 0.02, minCOUNT = 100), not raw FASTQ
- Variants were pile-up filtered with too-stringent minMAF/minCOUNT, leaving too few
  samples per variant for OPTICS clusters to satisfy the minSample size constraint
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3697
tools:
- name: MetaQuad
  role: Runs the 1-D OPTICS density clustering over per-sample allele frequencies
    and outputs BIC_params.csv for informative variant discovery
  repo: https://github.com/holab-hku/MetaQuad
- name: cellsnp-lite
  role: Upstream pile-up of per-sample AD/DP allele counts (recommended minMAF = 0.02,
    minCOUNT = 100) that serves as MetaQuad's input folder
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: bwa
  role: Alignment tool used to align filtered shotgun metagenomic reads to a reference
    database before variant pile-up
- name: bowtie2
  role: Alternative alignment tool to bwa for producing .filtered.sorted.bam inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:09eaf0a9319af11e66539bd32f9aa11e615bc067f7eaccfb5107338d0c7cd2e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad
  tools:
  - MetaQuad
  - Python
  - bowtie2
  - bwa
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# informative-variant-filtering

## Summary

Filters shotgun metagenomic variants into shared informative variants vs. background noise (sequencing errors, non-polymorphic sites) using 1-D OPTICS density clustering of per-sample allele frequencies from cellsnp-lite pile-ups, implemented in the MetaQuad Python package.

## When to use

Use when you have a cellsnp-lite output folder (cellSNP) with per-sample AD/DP allele counts from shotgun metagenomic samples and need to distinguish shared informative variants from non-polymorphic sites or sequencing-error noise.

## When NOT to use

- Input is single-cell mitochondrial mutation data for clonal substructure inference — use MQuad (MetaQuad's predecessor), which applies a binomial mixture model instead of the metagenomic OPTICS workflow
- Reads are not yet filtered and aligned to a reference database — this skill requires an upstream cellsnp-lite pile-up step (minMAF = 0.02, minCOUNT = 100), not raw FASTQ
- Variants were pile-up filtered with too-stringent minMAF/minCOUNT, leaving too few samples per variant for OPTICS clusters to satisfy the minSample size constraint

## Inputs

- cellsnp-lite output folder (e.g. cellSNP) with per-sample AD and DP allele-count sparse matrices (.mtx) or equivalent VCF, from shotgun metagenomic BAMs aligned with bwa or bowtie2
- Parameter minSample (recommended 5% of the total sample size, e.g. --minSample=5)

## Outputs

- BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP (count of non-zero-DP samples)
- Set of informative variants satisfying num_of_clusters >= 2 for downstream analysis (e.g. nucleotide diversity via nucleotide_diversity_code.R)

## How to apply

Load the cellsnp-lite output folder produced with recommended parameters minMAF = 0.02 and minCOUNT = 100, and compute per-sample alternate-allele frequencies (AD/DP) for every variant. For each variant, run one-dimensional OPTICS density clustering (scikit-learn) over the sample allele frequencies — computing reachability and core distances, extracting clusters, labelling noise points, and enforcing the minSample minimum cluster size (recommended: 5% of total sample size, e.g. --minSample=5). Record the predicted number of clusters (num_of_clusters) per variant and flag a variant as informative when num_of_clusters >= 2. Rationale: density clustering groups samples with similar allele frequencies into clusters, so a variant supported by two or more clusters of adequate size reflects a shared subpopulation signal, while isolated AF values are treated as noise and rejected — which empirically reduces false positive SNPs without greatly affecting the true positive rate. Write BIC_params.csv containing variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP (count of non-zero-DP samples) for downstream filtering and nucleotide-diversity analysis.

## Related tools

- **MetaQuad** (Runs the 1-D OPTICS density clustering over per-sample allele frequencies and outputs BIC_params.csv for informative variant discovery) — https://github.com/holab-hku/MetaQuad
- **cellsnp-lite** (Upstream pile-up of per-sample AD/DP allele counts (recommended minMAF = 0.02, minCOUNT = 100) that serves as MetaQuad's input folder) — https://github.com/single-cell-genetics/cellsnp-lite
- **bwa** (Alignment tool used to align filtered shotgun metagenomic reads to a reference database before variant pile-up)
- **bowtie2** (Alternative alignment tool to bwa for producing .filtered.sorted.bam inputs)

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- BIC_params.csv exists and contains all required columns: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP
- Every num_of_clusters value in BIC_params.csv is a non-negative integer
- Variants flagged as informative satisfy num_of_clusters >= 2
- The minSample cluster-size constraint (recommended 5% of total sample size, e.g. 5) was applied during OPTICS clustering
- number_DP column matches the count of samples with non-zero DP depth for each variant

## Limitations

- Recommended cellsnp-lite parameters (minMAF = 0.02, minCOUNT = 100) trade sensitivity for computation time; a higher value is recommended to reduce computation time for subsequent steps
- minSample should be set to ~5% of the total sample size — a value too small admits noise clusters, too large discards genuine low-prevalence shared variants
- OPTICS treats singleton/small allele-frequency groups as noise, so variants present in fewer than minSample samples cannot be classified as informative

## Evidence

- [intro] It uses a density clustering model to assess the informative variants among background noise.: "It uses a density clustering model to assess the informative variants among background noise."
- [abstract] MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate: "MetaQuad significantly reduces the number of false positive SNPs without greatly affecting the true positive rate."
- [readme] metaquad  -i cellSNP -o Metaquad --minSample=5: "metaquad  -i cellSNP -o Metaquad --minSample=5"
- [readme] Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100.: "Pile up variants using [cellsnp-lite](https://github.com/single-cell-genetics/cellsnp-lite). <br/> with the following recommended parameters: minMAF = 0.02 and minCOUNT = 100."
- [methods] Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2.: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
