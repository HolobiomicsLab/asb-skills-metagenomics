---
name: synthetic-community-truth-comparison
description: Decision support for choosing an OTU-picking pipeline when a synthetic
  dataset with known OTU composition is available and abundance-accuracy (not just
  OTU richness) must be compared across tools.
when_to_use_negative:
- The dataset is real biological data with no known true composition — RSS against
  truth cannot be computed
- The comparison goal is OTU count accuracy or richness estimation rather than relative-abundance
  accuracy
- Input is already a distance/phylogenetic comparison (e.g. UniFrac or genus-level
  consistency), for which different metrics apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: OTU-picking pipeline whose abundance estimates are benchmarked; best RSS (0.004)
    on 16S-R
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline; RSS of 0.027 on 16S-R top-20 abundances
- name: UPARSE
  role: Comparison pipeline; RSS of 0.028 on 16S-R top-20 abundances
- name: mothur
  role: Comparison pipeline; RSS of 0.008 on 16S-R top-20 abundances
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:b716b3227a121fc1de752ad4b8b587cb5c9cca3fa62ec958a8ea570ff283f633
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# synthetic-community-truth-comparison

## Summary

Benchmarks OTU-picking pipelines (MICCA, QIIME, UPARSE, mothur) against a synthetic community with known composition by computing the Residual Sum of Squares (RSS) between estimated and true top-20 relative abundances. This quantifies which pipeline most accurately recovers abundance profiles, not just OTU counts.

## When to use

Decision support for choosing an OTU-picking pipeline when a synthetic dataset with known OTU composition is available and abundance-accuracy (not just OTU richness) must be compared across tools.

## When NOT to use

- The dataset is real biological data with no known true composition — RSS against truth cannot be computed
- The comparison goal is OTU count accuracy or richness estimation rather than relative-abundance accuracy
- Input is already a distance/phylogenetic comparison (e.g. UniFrac or genus-level consistency), for which different metrics apply

## Inputs

- Synthetic amplicon dataset with known community composition (e.g. 16S-R: 500 OTUs, 100000 reads, downloadable from ftp://ftp.fmach.it/metagenomics/micca/scirep/)
- OTU abundance tables (counts per OTU) produced by each pipeline being compared (MICCA, QIIME, UPARSE, mothur)
- Reference vector of true relative abundances for the synthetic community

## Outputs

- Per-pipeline RSS values for top-20 relative abundances (e.g. rss_16sr_top20.csv)
- Ranking of pipelines by deviation from truth abundance profile

## How to apply

Obtain the synthetic dataset and its known composition from the deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/ (e.g. 16S-R: 500 OTUs, 100000 sequence reads with a realistic 454 error model). Load the OTU abundance tables produced by each pipeline (MICCA, QIIME, UPARSE, mothur), convert counts to relative abundances, and extract the 20 most abundant OTUs. For each pipeline compute the residual sum of squares between its top-20 estimated relative abundances and the known truth abundances. In the MICCA paper this yielded RSS of 0.004 (MICCA), 0.008 (mothur), 0.027 (QIIME) and 0.028 (UPARSE) on 16S-R, with MICCA showing the smallest deviation. Write the values to a comparison file (e.g. rss_16sr_top20.csv) and validate against reported benchmarks.

## Related tools

- **MICCA** (OTU-picking pipeline whose abundance estimates are benchmarked; best RSS (0.004) on 16S-R) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline; RSS of 0.027 on 16S-R top-20 abundances)
- **UPARSE** (Comparison pipeline; RSS of 0.028 on 16S-R top-20 abundances)
- **mothur** (Comparison pipeline; RSS of 0.008 on 16S-R top-20 abundances)

## Examples

```
python compute_rss.py --truth 16sr_truth_abundances.tsv --tables micca_otu.tsv qiime_otu.tsv uparse_otu.tsv mothur_otu.tsv --top 20 --out rss_16sr_top20.csv
```

## Evaluation signals

- Recomputed RSS values match reported benchmarks: 0.004 (MICCA), 0.008 (mothur), 0.027 (QIIME), 0.028 (UPARSE) on 16S-R
- MICCA shows the smallest deviation from true top-20 relative abundances among all compared pipelines
- Each pipeline's OTU counts are converted to relative abundances (summing to 1) before RSS computation
- Exactly the 20 most abundant OTUs are extracted consistently across pipelines before comparison
- Output csv contains the four RSS values for MICCA, QIIME, UPARSE and mothur

## Limitations

- Validation relies on simulated data with known OTU composition; no official well-studied benchmark datasets exist for metagenomic pipelines
- Results reflect the 16S-R synthetic conditions (454 error model, 20% chimeras in related 16S-10/ITS-10 sets); behaviour may differ on real or other sequencing-platform data
- UPARSE discards a large fraction of reads (only 42.3% and 66.8% kept on 16S-10 and ITS-10), which affects abundance estimates compared to less stringent filters

## Evidence

- [results] RSS 0.004 for MICCA on 16S-R top-20 abundances vs QIIME/UPARSE/mothur: "with a value of the Residual Sum of Squares (RSS) of 0.004, to be compared with 0.027 and 0.028 for QIIME and UPARSE, respectively (0.008 for mothur)"
- [results] 16S-R synthetic dataset description: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [readme] Dataset download location from MICCA README: "Dataset download:
ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [discussion] Lack of benchmark datasets limitation: "Given the lack of official and well-studied datasets to test the analysis tools, it is generally difficult to assess the accuracy and reliability of a metagenomic pipeline"
- [abstract] MICCA accuracy claim from abstract: "MICCA provides estimates of the number of OTUs and of other common ecological indices that are more accurate and robust than currently available pipelines."
