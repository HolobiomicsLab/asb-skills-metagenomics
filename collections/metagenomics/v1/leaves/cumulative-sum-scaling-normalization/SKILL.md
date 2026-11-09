---
name: cumulative-sum-scaling-normalization
description: Use when shotgun metagenome read-mapping produced gene frequency counts
  across samples and differential analysis must be composition-aware (sparse, zero-inflated
  data spanning taxa, gene, KO, EC and reaction levels).
when_to_use_negative:
- The count table has already been CSS-normalized (re-normalizing aggregated KO/EC/reaction
  tables instead of the gene table would violate the CAMAMED aggregation-after-normalization
  order).
- Data are not mapping-based gene frequencies from a gene catalog (e.g., no per-sample
  gene counts and gene lengths available for the low-count and length-correction steps).
- Sample size is very small — the article notes metagenomeSeq has a high false-positive
  rate for differential abundance analysis when the sample size is small.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0659
tools:
- name: metagenomeSeq
  role: Performs the CSS normalization exploiting its zero-inflated log-normal model
    for sparse compositional gene abundance data
- name: CAMAMED
  role: Pipeline module that orchestrates low-count filtering (<5 reads), gene-length
    correction and CSS at taxa and gene levels before functional aggregation
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Runs the ready-to-use camamed_pipeline / camamed_pipeline_db images so metagenomeSeq
    and dependencies need no separate installation
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:dc7c75d44b320bc56b30a5372becd342aef68e89e0b077f263803b578db4e6bf
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# cumulative-sum-scaling-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply cumulative sum-scaling (CSS) normalization via metagenomeSeq to correct compositional bias in sparse metagenomic count data at both taxa and gene levels, as implemented in the CAMAMED pipeline before functional (KO, EC, reaction) aggregation.

## When to use

Use when shotgun metagenome read-mapping produced gene frequency counts across samples and differential analysis must be composition-aware (sparse, zero-inflated data spanning taxa, gene, KO, EC and reaction levels).

## When NOT to use

- The count table has already been CSS-normalized (re-normalizing aggregated KO/EC/reaction tables instead of the gene table would violate the CAMAMED aggregation-after-normalization order).
- Data are not mapping-based gene frequencies from a gene catalog (e.g., no per-sample gene counts and gene lengths available for the low-count and length-correction steps).
- Sample size is very small — the article notes metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small.

## Inputs

- Mapped gene frequency count table (gene x sample) from mapping shotgun reads to a non-redundant gene catalog (e.g., IGC)
- Species/taxon abundance table from taxonomic profiling (e.g., MetaPhlAn2 output)
- Gene length annotation table for length correction

## Outputs

- CSS-normalized gene frequency table
- CSS-normalized species (taxon) frequency table
- Normalized KO, EC number and reaction frequency tables (css_normalized_frequencies.tsv) obtained by summing normalized gene frequencies within each functional category

## How to apply

First load the mapped gene frequency count table produced by mapping reads to a non-redundant gene catalog (e.g., IGC; CAMAMED expects ~3,354,281 mapped genes for the cohort1 CRC analysis). Filter out genes with fewer than five mapped reads summed across all samples, then divide each gene frequency by its gene length. Apply CSS normalization independently on the gene abundance table and on the species (taxon) abundance table using metagenomeSeq, which is chosen for its zero-inflated log-normal handling of sparse compositional data. Only after normalization, aggregate gene-level frequencies into KO, EC number and reaction tables by summing the normalized frequencies of their subset genes. The rationale is that without CSS correction, results from the two colorectal cancer datasets were not consistent, so compositional bias removal is essential before Kruskal-Wallis testing with Benjamini-Hochberg correction.

## Related tools

- **metagenomeSeq** (Performs the CSS normalization exploiting its zero-inflated log-normal model for sparse compositional gene abundance data)
- **CAMAMED** (Pipeline module that orchestrates low-count filtering (<5 reads), gene-length correction and CSS at taxa and gene levels before functional aggregation) — https://github.com/mhnb/camamed
- **Docker** (Runs the ready-to-use camamed_pipeline / camamed_pipeline_db images so metagenomeSeq and dependencies need no separate installation)

## Examples

```
python camamed_init
```

## Evaluation signals

- Genes with fewer than five mapped reads summed across all samples are absent from the normalized output table.
- CSS is applied independently to the taxa table and the gene abundance table — check two separate normalization passes.
- KO, EC and reaction frequencies equal the sum of CSS-normalized gene frequencies of their member genes (aggregation happens after normalization).
- Downstream BH-adjusted Kruskal-Wallis results reproduce the reported significant-entity ratios (species 2.67%, genes 1.92%, KO 2.98%, EC 2.55%, reactions 2.45%) only when CSS is applied.
- Output CSV contains differential abundance results at all five levels (taxon, gene, KO, EC, reaction) with the mapped-gene count 3,354,281.

## Limitations

- metagenomeSeq has a high false-positive rate for differential abundance analysis when the sample size is small.
- No software version is stated for CAMAMED or metagenomeSeq, and the repository has no changelog, so exact CSS parameters are hard to pin down.
- Without applying CSS, results from the two colorectal cancer datasets were not consistent — skipping normalization breaks cross-dataset reproducibility.

## Evidence
