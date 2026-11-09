---
name: compositional-data-css-normalization
description: Use when metagenomic count data (taxa or gene-level abundance tables
  from case-control studies) is sparse and compositional, and differential abundance
  markers must be comparable or reproducible across independent datasets.
when_to_use_negative:
- Input data is not sparse compositional count data (e.g., already normalized relative
  abundances or non-metagenomic expression tables) — CSS correction targets compositional
  bias in metagenomic count tables
- No differential abundance or biomarker identification is intended and only raw mapping/gene
  frequency computation is needed — run the CAMAMED taxonomic/functional profiling
  stages without the CSS+metagenomeSeq step
- The sample size is very small and high metagenomeSeq false-positive rates are a
  concern; the article notes metagenomeSeq has a high false-positive rate for differential
  abundance analysis when the sample size is small
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: CAMAMED
  role: Pipeline that applies CSS normalization at taxa and gene levels and couples
    it with metagenomeSeq for compositional differential abundance analysis; distributed
    as Docker images camamed_pipeline and camamed_pipeline_db
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Fits the zero-inflated log-normal differential abundance model on CSS-corrected
    counts to extract marker taxa and genes
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Ready-to-use execution environment (camamed_pipeline / camamed_pipeline_db
    images on Docker Hub) avoiding manual dependency installation
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:e568912f67bf319a66c9102717ed940c6e62a9297e984df666500827ec25c64d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - metagenomeSeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# compositional-data-css-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A compositional data analysis (CoDA) skill from CAMAMED that applies cumulative sum-scaling (CSS) normalization to sparse metagenomic abundance tables at taxon, gene, KO, EC number, and reaction levels before zero-inflated metagenomeSeq differential abundance testing, correcting composition bias that otherwise makes case-control biomarker findings irreproducible across datasets.

## When to use

Use when metagenomic count data (taxa or gene-level abundance tables from case-control studies) is sparse and compositional, and differential abundance markers must be comparable or reproducible across independent datasets.

## When NOT to use

- Input data is not sparse compositional count data (e.g., already normalized relative abundances or non-metagenomic expression tables) — CSS correction targets compositional bias in metagenomic count tables
- No differential abundance or biomarker identification is intended and only raw mapping/gene frequency computation is needed — run the CAMAMED taxonomic/functional profiling stages without the CSS+metagenomeSeq step
- The sample size is very small and high metagenomeSeq false-positive rates are a concern; the article notes metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small

## Inputs

- Taxa abundance count table from mapping metagenome sequences to gene catalogs (e.g., MetaPhlAn2-based taxon profiles)
- Gene-level abundance count table (gene frequencies per sample from non-redundant gene catalog mapping)
- Functional abundance tables at KO, EC number, and reaction levels (from KEGG annotation of mapped genes)
- Case-control sample grouping metadata

## Outputs

- CSS-normalized abundance matrices at each feature level
- Differential abundance marker lists (taxa and genes) from the metagenomeSeq zero-inflated log-normal model
- Cross-dataset consistency comparison of CSS-corrected vs. uncorrected marker sets (e.g., a summary table such as css_cross_dataset_validation.csv)

## How to apply

Load the raw taxa and gene abundance count tables produced by mapping reads to non-redundant gene catalogs, then compute cumulative sum-scaling normalization factors independently for each feature level (taxa, gene, and downstream KO, EC number, and reaction levels). Fit the metagenomeSeq zero-inflated log-normal differential abundance model on CSS-corrected counts to extract case-control marker features; CAMAMED packages this because metagenomeSeq can handle compositional bias in sparse data. To assess whether the correction matters, rerun the identical differential abundance analysis without CSS on the same inputs and compare the marker sets: the article reports that CSS-corrected results from two independent colorectal cancer gut metagenome datasets were consistent, whereas 'without applying the CSS, the results from these datasets were not consistent'. The rationale is that normalization removes systematic variability and compositional bias, an essential preprocessing step before conventional statistical testing on compositional data.

## Related tools

- **CAMAMED** (Pipeline that applies CSS normalization at taxa and gene levels and couples it with metagenomeSeq for compositional differential abundance analysis; distributed as Docker images camamed_pipeline and camamed_pipeline_db) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Fits the zero-inflated log-normal differential abundance model on CSS-corrected counts to extract marker taxa and genes) — https://github.com/mhnb/camamed
- **Docker** (Ready-to-use execution environment (camamed_pipeline / camamed_pipeline_db images on Docker Hub) avoiding manual dependency installation) — https://github.com/mhnb/camamed

## Examples

```
python camamed_init
```

## Evaluation signals

- CSS-corrected marker sets show substantially greater cross-dataset agreement (overlap/consistency) than uncorrected marker sets when the same analysis is run on two independent datasets
- Reproducing the workflow on a second independent colorectal cancer dataset recovers the same taxa and gene markers reported from the first dataset
- The consistency summary output file exists and contains metrics for both CSS-corrected and uncorrected comparisons
- CSS normalization was computed independently per feature level (taxa and gene), not pooled across levels
- The zero-inflated log-normal metagenomeSeq model (not plain counts) was fit to the CSS-corrected tables

## Limitations

- Without CSS correction, marker results are inconsistent across independent datasets, so uncorrected runs should not be trusted for cross-study claims
- metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small
- No software version is stated for CAMAMED or its component tools, and the two colorectal cancer datasets are not identified by accession, complicating exact reproduction

## Evidence

- [abstract] the cumulative sum-scaling method is used at both taxa and gene levels for compositional data analysis in our pipeline: "the cumulative sum-scaling method is used at both taxa and gene levels for compositional data analysis in our pipeline"
- [intro] CAMAMED uses metagenomeSeq with CSS to handle compositional bias: "CAMAMED uses the metagenomeSeq due to its high capability of handling compositional bias in sparse metagenomic data in taxa, gene, KO, EC number and reaction levels"
- [discussion] Uncorrected results were inconsistent across the two colorectal cancer datasets: "However, without applying the CSS, the results from these datasets were not consistent (30)."
- [intro] Normalization is essential to remove systematic variability and compositional bias: "Normalization processes can identify and eliminate systematic variability and compositional bias, so it is an essential step in data preprocessing and analysis"
