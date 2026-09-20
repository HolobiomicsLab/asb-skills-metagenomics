---
name: gene-length-normalization
description: Use when a mapped gene frequency count table (reads per gene across samples)
  is the input and gene-level abundances must be compared across samples; long genes
  systematically accumulate more reads, biasing downstream normalization.
when_to_use_negative:
- Input is already normalized (e.g. relative frequencies or previously CSS-scaled
  data) — dividing by length again would introduce bias
- Input is taxon-level abundance rather than gene-level counts; species-level data
  undergo CSS normalization directly without gene-length correction
- Working with assembly-based gene content prediction rather than mapping-based counts
  against a gene catalog
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline that performs low-count filtering, divides gene frequencies by gene
    length, and orchestrates CSS normalization
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: R package applying cumulative sum-scaling (CSS) with a zero-inflated log-normal
    model to the length-normalized abundance data
  repo: https://github.com/mhnb/camamed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:ab889439a1f43f2a2b2e1867eb98a5a682b05d8b5a21ba245b8938876c173243
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
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-length-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Length-corrects mapped gene frequencies by dividing each gene's read count by its gene length before CSS normalization, as done in the CAMAMED pipeline so that longer genes do not accrue inflated abundance purely from being longer mapping targets.

## When to use

Use when a mapped gene frequency count table (reads per gene across samples) is the input and gene-level abundances must be compared across samples; long genes systematically accumulate more reads, biasing downstream normalization.

## When NOT to use

- Input is already normalized (e.g. relative frequencies or previously CSS-scaled data) — dividing by length again would introduce bias
- Input is taxon-level abundance rather than gene-level counts; species-level data undergo CSS normalization directly without gene-length correction
- Working with assembly-based gene content prediction rather than mapping-based counts against a gene catalog

## Inputs

- Mapped gene frequency count table (genes x samples) from mapping reads to a non-redundant gene catalog
- Gene length annotation for each gene in the catalog

## Outputs

- Length-normalized gene frequency matrix (input to CSS normalization)
- CSS-normalized gene, species, KO, EC number and reaction frequency tables written to css_normalized_frequencies.tsv

## How to apply

Load the count table of mapped gene frequencies produced by the upstream read-mapping step of CAMAMED against a non-redundant gene catalog. First filter the matrix to discard genes with fewer than five mapped reads summed across all samples. Then divide each gene's frequency by that gene's length to yield length-normalized gene frequencies. This length correction precedes cumulative sum-scaling (CSS) normalization via metagenomeSeq, which then handles compositional bias and zero inflation in the sparse count data. The length-normalized, CSS-scaled gene frequencies are the basis for aggregating into KO, EC number and reaction tables.

## Related tools

- **CAMAMED** (Pipeline that performs low-count filtering, divides gene frequencies by gene length, and orchestrates CSS normalization) — https://github.com/mhnb/camamed
- **metagenomeSeq** (R package applying cumulative sum-scaling (CSS) with a zero-inflated log-normal model to the length-normalized abundance data) — https://github.com/mhnb/camamed

## Examples

```
python camamed_init
```

## Evaluation signals

- No gene in the output matrix has < 5 total mapped reads across all samples (filter applied before length division)
- Each gene's normalized frequency equals its mapped read count divided by gene length, prior to CSS scaling
- Downstream CSS normalization succeeds without errors on the length-corrected matrix (sparse, compositional data)
- Re-aggregated KO/EC/reaction tables equal the sums of normalized subset-gene frequencies in each annotation category
- Output written to css_normalized_frequencies.tsv contains gene, species and functional-level frequency tables

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis with small sample sizes
- Gene-length correction requires gene length metadata for the gene catalog; without it, length bias remains in the counts
- Without applying CSS after length normalization, results across independent datasets were not consistent in the colorectal cancer case study

## Evidence

- [other] CAMAMED's normalization module removes genes with fewer than five mapped reads across all samples, divides each gene frequency by gene length, applies metagenomeSeq with cumulative sum-scaling (CSS): [evidence span withheld — anchor status: not_found]
