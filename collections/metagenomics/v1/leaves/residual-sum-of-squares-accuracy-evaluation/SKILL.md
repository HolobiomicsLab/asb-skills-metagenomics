---
name: residual-sum-of-squares-accuracy-evaluation
description: Use when a synthetic dataset with known OTU composition is available
  and you need to score how faithfully different amplicon pipelines (MICCA, QIIME,
  UPARSE, mothur) reproduce true relative abundances.
when_to_use_negative:
- The dataset is real (e.g. HMP, Delaware Bay, Dethlefsen time-series) with no known
  true composition — RSS against truth is undefined
- The question is OTU count accuracy or richness estimation rather than relative-abundance
  accuracy (the paper assesses OTU-number convergence separately via rarefaction)
- The input is a raw FASTQ file that has not yet been processed by an OTU-picking
  pipeline to produce an abundance table
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MICCA
  role: OTU-picking pipeline being benchmarked; achieved the smallest top-20 RSS (0.004)
    on 16S-R
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Comparison pipeline; RSS of 0.027 on the 16S-R top-20 benchmark
- name: UPARSE
  role: Comparison pipeline; RSS of 0.028 on the 16S-R top-20 benchmark
- name: mothur
  role: Comparison pipeline; RSS of 0.008 on the 16S-R top-20 benchmark
- name: RDP
  role: Classifier used to assign OTUs to known genera at confidence level ≥ 90% during
    pipeline evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:60a0801f260cb3cf141e4011c07af3821566bd90ed086a8a5e2523e960b27790
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# residual-sum-of-squares-accuracy-evaluation

## Summary

Quantifies how accurately an OTU-picking pipeline recovers known relative abundances by computing the Residual Sum of Squares (RSS) between a pipeline's top-20 OTU relative abundances and the true abundances of a synthetic dataset. In the MICCA paper this benchmark on 16S-R gave RSS values of 0.004 (MICCA), 0.008 (mothur), 0.027 (QIIME) and 0.028 (UPARSE).

## When to use

Use when a synthetic dataset with known OTU composition is available and you need to score how faithfully different amplicon pipelines (MICCA, QIIME, UPARSE, mothur) reproduce true relative abundances.

## When NOT to use

- The dataset is real (e.g. HMP, Delaware Bay, Dethlefsen time-series) with no known true composition — RSS against truth is undefined
- The question is OTU count accuracy or richness estimation rather than relative-abundance accuracy (the paper assesses OTU-number convergence separately via rarefaction)
- The input is a raw FASTQ file that has not yet been processed by an OTU-picking pipeline to produce an abundance table

## Inputs

- Synthetic 16S dataset with known composition (16S-R: 500 OTUs, 100000 reads) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- OTU abundance tables produced by MICCA, QIIME, UPARSE and mothur on the synthetic dataset
- Known true relative-abundance vector for the synthetic dataset

## Outputs

- rss_16sr_top20.csv containing RSS values for MICCA (0.004), mothur (0.008), QIIME (0.027), UPARSE (0.028)
- Top-20 OTU relative-abundance vectors per pipeline

## How to apply

Obtain the synthetic dataset and its known composition from the MICCA deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/ (for the 16S-R benchmark this is 500 OTUs and 100,000 454-style reads). Load the OTU abundance tables produced by each pipeline on the same synthetic reads, convert raw counts to relative abundances, and select the 20 most abundant OTUs. For each pipeline compute the residual sum of squares between its top-20 relative abundance vector and the known synthetic truth vector; lower RSS means more accurate abundance profiling. Write one RSS value per pipeline to a table (rss_16sr_top20.csv) and confirm the recomputed values reproduce the reported benchmark, with MICCA showing the smallest deviation (0.004 vs 0.008/0.027/0.028). Because the truth composition is known, this is an absolute accuracy check rather than an inter-pipeline consistency check.

## Related tools

- **MICCA** (OTU-picking pipeline being benchmarked; achieved the smallest top-20 RSS (0.004) on 16S-R) — https://github.com/compmetagen/micca
- **QIIME** (Comparison pipeline; RSS of 0.027 on the 16S-R top-20 benchmark)
- **UPARSE** (Comparison pipeline; RSS of 0.028 on the 16S-R top-20 benchmark)
- **mothur** (Comparison pipeline; RSS of 0.008 on the 16S-R top-20 benchmark)
- **RDP** (Classifier used to assign OTUs to known genera at confidence level ≥ 90% during pipeline evaluation)

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('top20_16sr.csv'); t.groupby('pipeline').apply(lambda d: ((d.rel_abundance-d.truth)**2).sum()).to_csv('rss_16sr_top20.csv')"
```

## Evaluation signals

- Recomputed RSS for MICCA on 16S-R equals 0.004 (smallest of all pipelines)
- RSS ordering reproduces the reported benchmark: MICCA (0.004) < mothur (0.008) < QIIME (0.027) < UPARSE (0.028)
- rss_16sr_top20.csv contains exactly four RSS values, one per pipeline
- Top-20 selection is performed after converting counts to relative abundances (RSS computed on fractions summing to 1, not raw counts)
- All pipelines are evaluated on the same synthetic dataset with the same 500-OTU truth composition

## Limitations

- Validation relies on simulated/synthetic data with known OTU composition; the article notes that given the lack of official and well-studied datasets it is generally difficult to assess the accuracy and reliability of a metagenomic pipeline
- Benchmark restricted to the top-20 most abundant OTUs; low-abundance OTU accuracy is not measured
- UPARSE's stringent quality filtering discards a large fraction of reads (42.3% kept on 16S-10), which affects comparability of abundance estimates across pipelines

## Evidence

- [results] RSS benchmark values for the four pipelines on 16S-R: "with a value of the Residual Sum of Squares (RSS) of 0.004, to be compared with 0.027 and 0.028 for QIIME and UPARSE, respectively (0.008 for mothur)"
- [results] Description of the synthetic dataset used for the benchmark: "we have generated 16S-R, a large synthetic sequence dataset including 500 different OTUs and 100000 sequence reads"
- [methods] Dataset deposit location used to obtain 16S-R and its known composition: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
- [readme] MICCA quality filtering basis relevant to how its abundance table was produced: "filters low quality sequences according to the maximum allowed **expected error (EE) rate** %"
