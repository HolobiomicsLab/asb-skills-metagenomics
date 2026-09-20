---
name: otu-relative-abundance-computation
description: Use when comparing OTU-picking pipelines on data with a known composition
  (e.g. the 16S-R synthetic dataset of 500 OTUs and 100000 reads), and you need a
  quantitative abundance-accuracy metric rather than OTU-count or rarefaction curves.
when_to_use_negative:
- The dataset has no known/synthetic truth composition (e.g. real HMP, Delaware Bay,
  or Dethlefsen data), so RSS against truth cannot be computed
- The question is OTU-count estimation accuracy or chimeric-read retention rather
  than relative-abundance accuracy of the most abundant OTUs
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: Pipeline whose OTU table is scored; achieved the lowest RSS (0.004) on 16S-R
    top-20 relative abundances
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline; RSS 0.027 on 16S-R top-20 relative abundances
- name: UPARSE
  role: Comparison pipeline; RSS 0.028 on 16S-R top-20 relative abundances
- name: mothur
  role: Comparison pipeline; RSS 0.008 on 16S-R top-20 relative abundances
- name: RDP
  role: Classifier used to assign OTUs to known genera at confidence level ≥ 90%
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:d3082cf15ba7123d2add25cd3d063335dec3484a226b3e52db05a71aff34f8df
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# otu-relative-abundance-computation

## Summary

Converts per-pipeline OTU count tables to relative abundances and scores the top-20 most abundant OTUs against known/synthetic truth compositions using Residual Sum of Squares (RSS), to quantify which OTU-picking pipeline (MICCA, QIIME, UPARSE, mothur) best estimates true abundances.

## When to use

Use when comparing OTU-picking pipelines on data with a known composition (e.g. the 16S-R synthetic dataset of 500 OTUs and 100000 reads), and you need a quantitative abundance-accuracy metric rather than OTU-count or rarefaction curves.

## When NOT to use

- The dataset has no known/synthetic truth composition (e.g. real HMP, Delaware Bay, or Dethlefsen data), so RSS against truth cannot be computed
- The question is OTU-count estimation accuracy or chimeric-read retention rather than relative-abundance accuracy of the most abundant OTUs

## Inputs

- OTU abundance (count) tables from each pipeline (MICCA, QIIME, UPARSE, mothur) on the same dataset
- Known/synthetic truth composition of the dataset (e.g. 16S-R truth abundances)
- Dataset deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/

## Outputs

- Per-pipeline top-20 relative-abundance vectors
- RSS values per pipeline written to rss_16sr_top20.csv (MICCA 0.004, mothur 0.008, QIIME 0.027, UPARSE 0.028 on 16S-R)

## How to apply

Obtain the synthetic dataset and its known composition from ftp://ftp.fmach.it/metagenomics/micca/scirep/, and load the OTU abundance tables produced by each pipeline (MICCA, QIIME, UPARSE, mothur). Convert each pipeline's OTU counts to relative abundances (counts divided by sample total) and extract the 20 most abundant OTUs. For each pipeline, compute the residual sum of squares between its top-20 relative abundances and the known synthetic truth abundances, and write the four RSS values to rss_16sr_top20.csv. On the 16S-R dataset this procedure yields RSS of 0.004 (MICCA), 0.008 (mothur), 0.027 (QIIME) and 0.028 (UPARSE), so the smallest RSS identifies the most abundance-accurate pipeline. OTU taxonomy labels in the paper were assigned with RDP at confidence level ≥ 90%, with unclassifiable OTUs discarded.

## Related tools

- **MICCA** (Pipeline whose OTU table is scored; achieved the lowest RSS (0.004) on 16S-R top-20 relative abundances) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline; RSS 0.027 on 16S-R top-20 relative abundances)
- **UPARSE** (Comparison pipeline; RSS 0.028 on 16S-R top-20 relative abundances)
- **mothur** (Comparison pipeline; RSS 0.008 on 16S-R top-20 relative abundances)
- **RDP** (Classifier used to assign OTUs to known genera at confidence level ≥ 90%)

## Examples

```
rel = counts.div(counts.sum(axis=0), axis=1); top20 = rel.loc[rel.sum(axis=1).nlargest(20).index]; rss = ((top20.T - truth)**2).sum(); rss.to_csv('rss_16sr_top20.csv')
```

## Evaluation signals

- Recomputed RSS on 16S-R top-20 relative abundances reproduces the benchmark: MICCA 0.004, mothur 0.008, QIIME 0.027, UPARSE 0.028
- MICCA has the smallest RSS of the four pipelines in the output csv
- Each relative-abundance vector sums to 1 and contains exactly the 20 most abundant OTUs
- rss_16sr_top20.csv contains exactly four rows (MICCA, QIIME, UPARSE, mothur) with non-negative RSS values

## Limitations

- Only simulated/synthetic data with known OTU composition were used for validation, since no official and well-studied benchmark datasets exist for metagenomic pipelines
- UPARSE's stringent quality filtering (truncation at first base below the quality threshold plus minimum read length) discards a large fraction of sequence data, which affects its downstream abundance estimates
- Pipelines converge to different fractions of the real OTU number (MICCA ~77%, MICCA-FAST ~68%, UPARSE ~64% on average), so abundance comparisons on shared top OTUs may not capture full community differences

## Evidence

- [results] MICCA achieves the smallest deviation from true top-20 relative abundances with an RSS of 0.004, compared with 0.027 for QIIME, 0.028 for UPARSE, and 0.008 for mothur.: "with a value of the Residual Sum of Squares (RSS) of 0.004, to be compared with 0.027 and 0.028 for QIIME and UPARSE, respectively (0.008 for mothur)"
- [results] The 16S-R synthetic dataset contains 500 OTUs and 100000 reads.: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [methods] The dataset deposit is available at the stated FTP location.: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [results] OTUs were classified with RDP at confidence ≥ 90% and unclassifiable OTUs discarded.: "The OTUs identified by the different pipelines were classified to known genera using RDP6 (confidence level ≥ 90%)"
- [discussion] UPARSE's stringent filtering causes loss of a large fraction of sequence data.: "UPARSE, which implements a stringent quality filtering algorithm at the cost of discarding a large fraction of the sequence data"
