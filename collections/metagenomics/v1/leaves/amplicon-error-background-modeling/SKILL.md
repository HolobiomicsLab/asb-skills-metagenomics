---
name: amplicon-error-background-modeling
description: 'Use when building or validating a background error model for amplicon
  variant calling: one-mismatch child abundances (mean/SD per cluster) must be related
  to parent sequencing depth to justify an abundance threshold.'
when_to_use_negative:
- The input is not a HashSeq run output — e.g. you only have a finalized feature/count
  table with no one-mismatch cluster statistics
- You need to capture indel variation relative to a parent, since HashSeq does not
  explicitly model insertions or deletions (methods like Deblur that use multiple-sequence
  alignments are suggested)
- The target variants are true low-abundance sequences below the 1,000-read threshold
  — the article recommends more sensitive algorithms such as DADA2 for those
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HashSeq
  role: Provides makeLoessPlot and inferTrueSequences that generate childrenProperties.txt
    and fit/plot the LOESS background error model
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which the HashSeq package's LOESS plotting and regression
    functions are executed
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:32e00c0d9fc06a53d34f9c099acc432d0b2f62d6a060571a97d2ccea24d0480a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# amplicon-error-background-modeling

## Summary

Estimate a depth-dependent background sequencing-error model for 16S rRNA amplicon data by regressing the mean and standard deviation of one-mismatch child variant abundances against parent sequence abundance with LOESS, as implemented in the HashSeq package.

## When to use

Use when building or validating a background error model for amplicon variant calling: one-mismatch child abundances (mean/SD per cluster) must be related to parent sequencing depth to justify an abundance threshold.

## When NOT to use

- The input is not a HashSeq run output — e.g. you only have a finalized feature/count table with no one-mismatch cluster statistics
- You need to capture indel variation relative to a parent, since HashSeq does not explicitly model insertions or deletions (methods like Deblur that use multiple-sequence alignments are suggested)
- The target variants are true low-abundance sequences below the 1,000-read threshold — the article recommends more sensitive algorithms such as DADA2 for those

## Inputs

- childrenProperties.txt (mean and standard deviation of one-mismatch children per cluster, with parent abundances) from HashSeq inferTrueSequences
- outputDir: HashSeq output directory containing childrenProperties.txt

## Outputs

- loess_plot.png: scatterplot (rendered with scattermore) of children mean/SD versus parent abundance with overlaid LOESS fit
- Validated parent-abundance threshold (1,000 reads default; 10,000 for the largest data set) at which the normal-plus-LOESS error model fits

## How to apply

Run HashSeq's inferTrueSequences on paired, primer-removed, 250-base-trimmed Illumina MiSeq FASTQ files to produce childrenProperties.txt, which holds the mean and standard deviation of one-mismatch children for each cluster along with their parents' total abundances. In R, call makeLoessPlot(outputDir, fileName="childrenProperties.txt") to fit LOESS regression of children's mean and standard deviation versus parent abundance and overlay it on a scatterplot rendered with scattermore. Use the resulting plot to set the filtering threshold: the LOESS regression is a good fit for parents above 1,000 total reads in most data sets, and 10,000 reads for the largest data set. The fitted normal-plus-LOESS background model supplies the error rates (with explicit P values) used to infer true sequence variants against sequencing-error children.

## Related tools

- **HashSeq** (Provides makeLoessPlot and inferTrueSequences that generate childrenProperties.txt and fit/plot the LOESS background error model) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which the HashSeq package's LOESS plotting and regression functions are executed) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
makeLoessPlot(outputDir, fileName = "childrenProperties.txt")
```

## Evaluation signals

- loess_plot.png is produced and shows the LOESS fit tracking the mean–standard deviation relationship for parents above the 1,000-read threshold (Fig. 5 behavior)
- childrenProperties.txt exists in outputDir and contains one row per cluster with mean and standard deviation of one-mismatch children plus parent abundance
- The chosen threshold matches the article's guidance: 1,000 reads for all data sets except the largest, where 10,000 is used
- For high-abundance parents (>1,000 reads), the LOESS fit is a good approximation of the child mean/SD across all six evaluated 16S rRNA data sets

## Limitations

- The LOESS fit is only reliable above the parent-abundance threshold; for the largest data set a 10,000-read threshold was required instead of 1,000
- The Poisson assumption of equal mean and variance was not met (overdispersion) for one-mismatch children, which motivated the normal-plus-LOESS model; the negative binomial was rejected because it is undefined when variance is less than the mean
- This modeling pipeline is conservative and not sensitive for detecting true low-abundance variants; it also considers only one-mismatch children and does not model indels or two-plus mismatches

## Evidence

- [abstract] use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth for individual clusters: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth for individual clusters"
- [methods] makeLoessPlot output: "makeLoessPlot(outputDir,fileName = "childrenProperties.txt")"
- [readme] childrenProperties.txt contains mean and standard deviation: "This file contains mean and standard deviation for each cluster"
- [discussion] threshold 1000 vs 10000: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
