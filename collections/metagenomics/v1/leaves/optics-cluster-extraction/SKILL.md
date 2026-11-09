---
name: optics-cluster-extraction
description: Use when per-sample AD/DP allele counts (cellsnp-lite output) are available
  for a shotgun metagenomic population and variants must be classified as shared/informative
  versus background noise by clustering sample allele frequencies.
when_to_use_negative:
- Input is not per-sample allele-count data (no AD/DP from cellsnp-lite or an equivalent
  pileup) — the AF-per-sample vectors required by 1-D OPTICS cannot be formed
- Goal is mitochondrial/mtDNA variant detection in single-cell omics — use MQuad's
  binomial mixture model instead, which is the predecessor tool MetaQuad extends
- Sample count is below the minSample threshold (e.g. fewer than 5 samples at minSample=5),
  since no cluster can meet the minimum size constraint
edam_operation: http://edamontology.org/operation_3695
tools:
- name: MetaQuad
  role: Runs 1-D OPTICS clustering per cellsnp-lite variant end to end, producing
    num_of_clusters and BIC_params.csv
  repo: https://github.com/holab-hku/MetaQuad.git
- name: cellsnp-lite
  role: Upstream pileup of per-sample AD/DP allele counts that feed the AF vectors
    clustered by OPTICS
  repo: https://github.com/single-cell-genetics/cellsnp-lite
- name: Python
  role: Language of the open-source MetaQuad package implementing the scikit-learn
    OPTICS clustering
  repo: https://github.com/holab-hku/MetaQuad
- name: MQuad
  role: Predecessor tool whose interface MetaQuad extends to shotgun metagenomic data
  repo: https://github.com/single-cell-genetics/MQuad
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae030
    title: 'MetaQuad: shared informative variants discovery in metagenomic samples'
  source_content_hash: sha256:c6bbfca7b57c2588a6c1cf6a34ccf3e401c2e2ad71f2f03c6d30113690714ac6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/holab-hku/MetaQuad.git
  tools:
  - MQuad
  - MetaQuad
  - Python
  - cellsnp-lite
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# optics-cluster-extraction

## Summary

MetaQuad's core method: run one-dimensional OPTICS density-based clustering (scikit-learn) over per-sample allele frequencies for each cellsnp-lite variant, to separate shared informative variants from background noise (sequencing errors, non-polymorphic sites) in shotgun metagenomic data.

## When to use

Use when per-sample AD/DP allele counts (cellsnp-lite output) are available for a shotgun metagenomic population and variants must be classified as shared/informative versus background noise by clustering sample allele frequencies.

## When NOT to use

- Input is not per-sample allele-count data (no AD/DP from cellsnp-lite or an equivalent pileup) — the AF-per-sample vectors required by 1-D OPTICS cannot be formed
- Goal is mitochondrial/mtDNA variant detection in single-cell omics — use MQuad's binomial mixture model instead, which is the predecessor tool MetaQuad extends
- Sample count is below the minSample threshold (e.g. fewer than 5 samples at minSample=5), since no cluster can meet the minimum size constraint

## Inputs

- cellsnp-lite output folder (cellSNP) containing per-sample AD and DP allele-count matrices for each variant
- minSample minimum cluster-size parameter (recommended 5% of the total sample size)

## Outputs

- BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, number_DP
- Per-variant informative/binary classification where informative variants satisfy num_of_clusters >= 2

## How to apply

Load the cellsnp-lite output folder and compute the alternate-allele frequency (AD/DP) for every sample at each variant; each variant then yields a set of one-dimensional data points (one AF per sample). For each variant, run 1-D OPTICS (scikit-learn), which computes reachability and core distances, extracts clusters, and labels low-density points as noise, enforcing a minimum cluster size minSample (recommended 5% of total samples; e.g. --minSample=5). Record the predicted number of clusters (num_of_clusters) per variant and flag a variant as informative when num_of_clusters >= 2. Write BIC_params.csv with columns variant_names, num_of_clusters, mean_DP, mean_AD, and number_DP (count of non-zero-DP samples); MetaQuad, an extension of MQuad, performs these steps end to end from the cellSNP folder.

## Related tools

- **MetaQuad** (Runs 1-D OPTICS clustering per cellsnp-lite variant end to end, producing num_of_clusters and BIC_params.csv) — https://github.com/holab-hku/MetaQuad.git
- **cellsnp-lite** (Upstream pileup of per-sample AD/DP allele counts that feed the AF vectors clustered by OPTICS) — https://github.com/single-cell-genetics/cellsnp-lite
- **Python** (Language of the open-source MetaQuad package implementing the scikit-learn OPTICS clustering) — https://github.com/holab-hku/MetaQuad
- **MQuad** (Predecessor tool whose interface MetaQuad extends to shotgun metagenomic data) — https://github.com/single-cell-genetics/MQuad

## Examples

```
metaquad -i cellSNP -o Metaquad --minSample=5
```

## Evaluation signals

- BIC_params.csv exists and contains all required columns: variant_names, num_of_clusters, mean_DP, mean_AD, number_DP
- Every num_of_clusters value is a non-negative integer
- All variants flagged as informative satisfy num_of_clusters >= 2
- Each extracted cluster meets the minSample minimum-size constraint (minSample=5 in the packaged command)
- number_DP correctly equals the count of non-zero-DP samples for each variant

## Limitations

- Recommended parameters minSample = 5% of the total sample size and num_of_clusters >= 2 must be adapted to sample size; too-small cohorts violate the minSample constraint
- cellsnp-lite pileup quality gates clustering: the README recommends higher minMAF/minCOUNT (minMAF = 0.02, minCOUNT = 100) to reduce computation time in subsequent steps
- Noise points (sequencing errors, non-polymorphic sites) are deliberately labelled as noise by OPTICS, so variants with a single AF cluster are never informative

## Evidence

- [abstract] Density-based clustering separates informative variants from noise: "MetaQuad utilizes a density-based clustering technique to effectively distinguish between shared variants and non-polymorphic sites using shotgun metagenomic data."
- [intro] Recommended MetaQuad thresholds: "Detect informative mutations using MetaQuad, with the following recommended parameters: minSample = 5% of the total sample size and num_of_clusters >= 2."
- [methods] Packaged CLI invocation: "metaquad  -i cellSNP -o Metaquad --minSample=5"
- [methods] MetaQuad takes cellSNP-lite output: "MetaQuad takes the output folder of cellSNP-lite"
- [intro] Extension of MQuad: "MetaQuad is an extension of [MQuad](https://github.com/single-cell-genetics/MQuad), specifically tailored to support shotgun metagenomic data."
- [readme] Density clustering model rationale: "It uses a density clustering model to assess the informative variants among background noise."
