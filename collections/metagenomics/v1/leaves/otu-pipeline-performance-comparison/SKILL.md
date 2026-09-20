---
name: otu-pipeline-performance-comparison
description: Use when comparing how different quality-filtering strategies (MICCA's
  expected-error-based filtering vs UPARSE's truncation-based filtering) change retained-read
  and retained-chimera fractions on simulated 16S or ITS amplicon datasets.
when_to_use_negative:
- Input is real (non-simulated) data with unknown chimeric fraction — chimera-retention
  percentages cannot be validated without ground truth
- The goal is OTU-count accuracy or ecological-index estimation (e.g. RSS of top-20
  OTU abundances) rather than filtering/read-retention behavior
- Data are Illumina paired-end rather than the 454 single-end reads this benchmark's
  error model targets
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: UPARSE
  role: Provides the truncation-based quality filter, chimera detection, and dereplication
    steps whose read-retention (42.3%/66.8%) and chimera-retention (15.6%/15.5%) outcomes
    are reproduced
- name: MICCA
  role: Comparison baseline implementing flexible maximum-expected-error (EE) quality
    filtering that retains more reads than UPARSE
  repo: https://github.com/compmetagen/micca
- name: QIIME
  role: Secondary comparison point — number of reads discarded by MICCA and QIIME
    were very similar
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep09743
    title: 'MICCA: a complete and accurate software for taxonomic profiling of metagenomic
      data'
  source_content_hash: sha256:3c2694e452534eb3fb82747eed4dee8ccd186aa0d72300dbcd647c54b9748699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compmetagen/micca
  tools:
  - MICCA
  - QIIME
  - UPARSE
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# otu-pipeline-performance-comparison

## Summary

Benchmark how alternative amplicon-processing pipelines (MICCA vs UPARSE) differ in quality-filtering stringency by measuring read retention and chimeric-read retention on simulated 16S-10 and ITS-10 454 datasets with 20% known chimeras, and write the comparison metrics to a CSV.

## When to use

Use when comparing how different quality-filtering strategies (MICCA's expected-error-based filtering vs UPARSE's truncation-based filtering) change retained-read and retained-chimera fractions on simulated 16S or ITS amplicon datasets.

## When NOT to use

- Input is real (non-simulated) data with unknown chimeric fraction — chimera-retention percentages cannot be validated without ground truth
- The goal is OTU-count accuracy or ecological-index estimation (e.g. RSS of top-20 OTU abundances) rather than filtering/read-retention behavior
- Data are Illumina paired-end rather than the 454 single-end reads this benchmark's error model targets

## Inputs

- Simulated 16S-10 FASTQ dataset (454 error model, 20% chimeric sequences) from ftp://ftp.fmach.it/metagenomics/micca/scirep/
- Simulated ITS-10 FASTQ dataset (454 error model, 20% chimeric sequences) from the same deposit
- Quality-score threshold and minimum read length parameters for UPARSE filtering
- Maximum allowed expected-error (EE) threshold for MICCA filtering

## Outputs

- uparse_filtering_results.csv with four metrics: retained-read fraction per dataset and chimeric-read retention fraction per dataset
- Kept/total read counts per pipeline and per dataset
- Per-dataset counts of chimeric sequences retained after chimera detection and dereplication

## How to apply

Download the simulated 16S-10 and ITS-10 datasets (realistic 454 error model, 20% chimeric sequences) from the paper's deposit at ftp://ftp.fmach.it/metagenomics/micca/scirep/. Apply UPARSE's stringent quality filter, which truncates each read at the first base whose quality score falls below the specified threshold and enforces a minimum read length, recording kept/total counts; the reported result is 42.3% reads kept on 16S-10 and 66.8% on ITS-10. Run UPARSE's chimera detection and dereplication on the filtered reads and compute chimeric-read retention, reported at 15.6% (16S-10) and 15.5% (ITS-10) — the worst of the tested pipelines. Contrast these against MICCA's more flexible maximum-allowed expected-error (EE) based filtering, which uses sequenced reads more efficiently while discarding a comparable fraction to QIIME. Write the four metrics (16S-10 kept %, ITS-10 kept %, 16S-10 chimera %, ITS-10 chimera %) to uparse_filtering_results.csv.

## Related tools

- **UPARSE** (Provides the truncation-based quality filter, chimera detection, and dereplication steps whose read-retention (42.3%/66.8%) and chimera-retention (15.6%/15.5%) outcomes are reproduced)
- **MICCA** (Comparison baseline implementing flexible maximum-expected-error (EE) quality filtering that retains more reads than UPARSE) — https://github.com/compmetagen/micca
- **QIIME** (Secondary comparison point — number of reads discarded by MICCA and QIIME were very similar)

## Evaluation signals

- Retained-read fraction must equal the reported 42.3% on 16S-10 and 66.8% on ITS-10 after UPARSE quality filtering
- Chimera-retention fraction must match the reported worst-case values of 15.6% (16S-10) and 15.5% (ITS-10)
- uparse_filtering_results.csv contains exactly the four expected metrics
- UPARSE discards more reads than MICCA on both datasets (MICCA's more flexible EE-based filtering uses reads more efficiently)
- MICCA and QIIME discarded read numbers should be very similar, while UPARSE is markedly higher

## Limitations

- No official and well-studied benchmark datasets exist for metagenomic pipelines, so validation relies on simulated data with known OTU composition
- UPARSE's truncation-based filter combined with a minimum read length requirement causes loss of a large fraction of sequence data, which biases downstream OTU estimates (converging to ~64% of real OTU number on average)
- Only simulated 454-read datasets (16S-10, ITS-10, 16S-R) were used for accuracy validation, not real HMP or environmental data

## Evidence

- [results] UPARSE tends to discard a higher number of reads, keeping only 42.3% and 66.8% of the reads on the 16S-10 and ITS-10 datasets: "UPARSE tends to discard a higher number of reads, keeping only 42.3% and 66.8% of the reads on the 16S-10 and ITS-10 datasets"
- [results] the stringent quality filtering approach used by UPARSE, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold: "the stringent quality filtering approach used by UPARSE4, which is based on truncating the reads at the first base position which has a quality score less than a specified threshold"
- [results] UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively): "UPARSE obtained the worst results in terms of number of chimeric reads retained on both 16S-10 and ITS-10 datasets (15.6% and 15.5% respectively)"
- [results] we have analysed the 16S-10 and ITS-10 datasets generated using a realistic error model for 454 reads and including 20% of chimera sequences: "we have analysed the 16S-10 and ITS-10 datasets generated using a realistic error model for 454 reads and including 20% of chimera sequences"
- [readme] Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/.: "Dataset download: ftp://ftp.fmach.it/metagenomics/micca/scirep/."
