---
name: chimera-retention-quantification
description: Use when comparing OTU pipelines on datasets with known ground-truth
  chimera content (e.g., 454 error-model simulations with 20% chimeras) and you need
  to measure how many chimeric reads each pipeline retains after filtering and chimera
  detection.
when_to_use_negative:
- Input data are real experimental reads with unknown chimera content — ground-truth
  chimera fractions cannot be computed, so retention cannot be validated
- You only need an OTU count or taxonomic profile and not a filtering/chimera-detection
  accuracy comparison between pipelines
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: UPARSE
  role: Pipeline whose truncation-based quality filtering, chimera detection, and
    dereplication steps are being benchmarked for chimeric-read retention
- name: MICCA
  role: Comparison pipeline using flexible expected-error (EE)-based filtering; provides
    the contrasting chimera-detection performance
  repo: https://github.com/compmetagen/micca
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:b56a525af5c1e09d0971c637904793bf18e5834592ff964a495408544ab9bcd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - UPARSE
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# chimera-retention-quantification

## Summary

Benchmarking skill that quantifies the fraction of chimeric sequences surviving a pipeline's quality filtering and chimera detection, using simulated amplicon datasets with a known 20% chimeric content. It exposes pipelines that let chimeras through (e.g., UPARSE retaining 15.6%/15.5% on 16S-10/ITS-10).

## When to use

Use when comparing OTU pipelines on datasets with known ground-truth chimera content (e.g., 454 error-model simulations with 20% chimeras) and you need to measure how many chimeric reads each pipeline retains after filtering and chimera detection.

## When NOT to use

- Input data are real experimental reads with unknown chimera content — ground-truth chimera fractions cannot be computed, so retention cannot be validated
- You only need an OTU count or taxonomic profile and not a filtering/chimera-detection accuracy comparison between pipelines

## Inputs

- Simulated 16S-10 dataset (FASTQ, realistic 454 error model, 20% chimeric sequences)
- Simulated ITS-10 dataset (FASTQ, realistic 454 error model, 20% chimeric sequences)
- Pipeline-specific filtering/chimera-detection parameters (e.g., quality-score threshold, minimum read length, max expected error)

## Outputs

- Chimera-retention percentages per dataset (expected 15.6% on 16S-10 and 15.5% on ITS-10 for UPARSE)
- Retained-read fractions per dataset (expected 42.3% on 16S-10 and 66.8% on ITS-10 for UPARSE)
- uparse_filtering_results.csv with the four computed metrics

## How to apply

Download the simulated 16S-10 and ITS-10 datasets (realistic 454 error model, 20% chimeric sequences) from ftp://ftp.fmach.it/metagenomics/micca/scirep/. Apply each pipeline's quality filtering — UPARSE truncates each read at the first base whose quality score falls below a specified threshold and enforces a minimum read length, whereas MICCA filters by a maximum allowed expected error (EE) rate — then run each pipeline's chimera identification and dereplication steps. Count chimeric sequences retained per dataset and compute the percentage of chimeric reads retained relative to the input chimeras. Validate against the paper-reported worst-case UPARSE values of 15.6% (16S-10) and 15.5% (ITS-10), and record the four metrics (retained-read and chimera-retention fractions for both datasets) to a CSV such as uparse_filtering_results.csv.

## Related tools

- **UPARSE** (Pipeline whose truncation-based quality filtering, chimera detection, and dereplication steps are being benchmarked for chimeric-read retention)
- **MICCA** (Comparison pipeline using flexible expected-error (EE)-based filtering; provides the contrasting chimera-detection performance) — https://github.com/compmetagen/micca

## Examples

```
micca filter -i 16S-10.fastq -o filtered.fastq -e 0.5 && micca otu -i filtered.fastq -m denovo -o otus/ # then count chimeras retained vs. the 20% ground truth
```

## Evaluation signals

- Reproduced retained-read fraction on 16S-10 equals the reported 42.3%
- Reproduced retained-read fraction on ITS-10 equals the reported 66.8%
- Chimera-retention fraction matches the reported worst-case 15.6% (16S-10) and 15.5% (ITS-10)
- uparse_filtering_results.csv contains exactly the four expected metrics with values matching the paper within rounding
- Filtered read counts (kept/total) are recorded for every pipeline × dataset combination before chimera counting

## Limitations

- Validation is only possible on simulated data with known chimera content; the article notes that no official and well-studied benchmark datasets exist, so accuracy of pipelines is generally hard to assess
- UPARSE's truncation-based filtering combined with the minimum read length requirement discards a large fraction of sequence data, so retention metrics are strongly filter-dependent
- Synthetic 454 error model may not reflect behaviour on Illumina or other platforms

## Evidence

- [results] UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively): "UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively)"
- [results] the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold: "the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [readme] filters low quality sequences according to the maximum allowed **expected error (EE) rate: "filters low quality sequences according to the maximum allowed **expected error (EE) rate"
- [methods] Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/.: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
