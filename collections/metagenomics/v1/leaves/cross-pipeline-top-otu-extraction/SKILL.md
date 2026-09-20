---
name: cross-pipeline-top-otu-extraction
description: Use when a synthetic amplicon dataset with known composition is available
  and multiple pipelines' OTU tables must be ranked by accuracy of top-OTU relative
  abundance estimates.
when_to_use_negative:
- The dataset has no known ground-truth composition (e.g. real HMP, Delaware Bay or
  Dethlefsen samples) — RSS against truth is undefined
- Input is already a relative-abundance or rarefied table rather than raw pipeline
  OTU counts, which would bias the relative-abundance conversion
- The question concerns OTU-count/rarefaction convergence rather than top-OTU abundance
  accuracy — the article reports those metrics separately
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: MICCA
  role: OTU-picking pipeline under evaluation; its filtered counts feed the top-20
    relative-abundance RSS computation
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline whose top-20 relative abundances are scored by RSS against
    truth
- name: UPARSE
  role: Comparison pipeline; known to discard many reads and converge to ~64% of real
    OTU number
- name: mothur
  role: Comparison pipeline with performance qualitatively similar to QIIME (RSS 0.008)
- name: RDP
  role: Classifier assigning OTUs to known genera at confidence level ≥ 90% for consistent
    cross-pipeline taxonomy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:898a355273d208b413d75cabb499c84f96a78dbbb3a562055cb196efbc8de869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - QIIME
  - RDP
  - UPARSE
  - mothur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-pipeline-top-otu-extraction

## Summary

Compares OTU-picking pipelines (MICCA, QIIME, UPARSE, mothur) by converting each pipeline's OTU count table to relative abundances, extracting the top-20 most abundant OTUs, and scoring deviation from known synthetic truth via Residual Sum of Squares (RSS).

## When to use

Use when a synthetic amplicon dataset with known composition is available and multiple pipelines' OTU tables must be ranked by accuracy of top-OTU relative abundance estimates.

## When NOT to use

- The dataset has no known ground-truth composition (e.g. real HMP, Delaware Bay or Dethlefsen samples) — RSS against truth is undefined
- Input is already a relative-abundance or rarefied table rather than raw pipeline OTU counts, which would bias the relative-abundance conversion
- The question concerns OTU-count/rarefaction convergence rather than top-OTU abundance accuracy — the article reports those metrics separately

## Inputs

- 16S-R synthetic sequence dataset (500 different OTUs, 100000 sequence reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Known synthetic truth composition of the 16S-R dataset
- OTU abundance tables from MICCA, QIIME, UPARSE and mothur run on 16S-R

## Outputs

- rss_16sr_top20.csv containing RSS values for MICCA, QIIME, UPARSE and mothur
- Ranked pipeline comparison: MICCA 0.004, mothur 0.008, QIIME 0.027, UPARSE 0.028

## How to apply

Download the 16S-R synthetic dataset and its known composition from ftp://ftp.fmach.it/metagenomics/micca/scirep/. Load the OTU abundance tables produced on 16S-R by MICCA, QIIME, UPARSE and mothur, convert each table's counts to relative abundances, and extract the 20 most abundant OTUs per pipeline. For each pipeline, compute the residual sum of squares between its top-20 relative abundances and the known synthetic truth abundances. The rationale is that RSS on the most abundant OTUs isolates abundance-estimation accuracy from total-OTU-count effects (e.g. UPARSE converging to ~64% of real OTU number, MICCA to ~77%). Write the four RSS values to rss_16sr_top20.csv and compare against the published benchmark. Taxonomy assignment across pipelines used RDP at a confidence level ≥ 90%.

## Related tools

- **MICCA** (OTU-picking pipeline under evaluation; its filtered counts feed the top-20 relative-abundance RSS computation) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline whose top-20 relative abundances are scored by RSS against truth)
- **UPARSE** (Comparison pipeline; known to discard many reads and converge to ~64% of real OTU number)
- **mothur** (Comparison pipeline with performance qualitatively similar to QIIME (RSS 0.008))
- **RDP** (Classifier assigning OTUs to known genera at confidence level ≥ 90% for consistent cross-pipeline taxonomy)

## Examples

```
import pandas as pd; rss = ((top20_truth - top20_pred)**2).sum().sum(); pd.DataFrame({'pipeline':['MICCA','QIIME','UPARSE','mothur'],'rss':[rss_micca,0.027,0.028,0.008]}).to_csv('rss_16sr_top20.csv', index=False)
```

## Evaluation signals

- Recomputed RSS for MICCA on 16S-R equals 0.004 (smallest deviation from truth)
- Recomputed RSS values match the benchmark: 0.004 (MICCA), 0.008 (mothur), 0.027 (QIIME), 0.028 (UPARSE)
- Relative abundances per OTU sum to 1 within each pipeline's table before extracting the top 20
- Exactly 20 OTUs extracted per pipeline and aligned to truth OTUs before computing RSS
- rss_16sr_top20.csv contains four rows/pipeline entries with positive RSS values

## Limitations

- Only simulated data with known OTU composition (16S-10, ITS-10, 16S-R) can be validated this way; no official benchmark datasets exist for real communities
- UPARSE discards a large fraction of sequence data (only 42.3% and 66.8% of reads kept on 16S-10 and ITS-10), which affects downstream abundance estimates
- Estimated OTU counts converge below the real number even for the best pipelines (~77% for MICCA), so total-OTU accuracy is not captured by the top-20 RSS metric
- Non-convergence of rarefaction curves for QIIME and mothur can force sequencing to excessive depth, complicating fair comparison

## Evidence

- [results] MICCA achieves the smallest deviation from true top-20 relative abundances on 16S-R: "with a value of the Residual Sum of Squares (RSS) of 0.004, to be compared with 0.027 and 0.028 for QIIME and UPARSE, respectively (0.008 for mothur)"
- [results] 16S-R synthetic dataset description: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [readme] Dataset availability for reproducing the benchmark: "Dataset download:
ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [results] Taxonomic consistency step supporting cross-pipeline comparison: "The OTUs identified by the different pipelines were classified to known genera using RDP6 (confidence level ≥ 90%)"
- [discussion] Caveat on benchmark availability: "Given the lack of official and well-studied datasets to test the analysis tools, it is generally difficult to assess the accuracy and reliability of a metagenomic pipeline"
