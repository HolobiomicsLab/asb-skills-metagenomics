---
name: parent-abundance-fit-curve-visualization
description: Use when you have HashSeq OneMismatchCluster.txt output from 16S rRNA
  amplicon data and need to show how one-mismatch child discovery scales with parent
  read abundance and fit a Poisson error model.
when_to_use_negative:
- Your goal is variant calling itself rather than characterizing error saturation
  — use inferTrueSequences/SvTable.txt instead
- You need to model the mean/variance of one-mismatch children counts rather than
  fraction observed — use HashSeq's makeLoessPlot on childrenProperties.txt, since
  the Poisson assumption of equal mean and variance was not met in that setting
- Long-read data (e.g., PacBio) where multiple-mismatch sequences are not rare, violating
  the one-mismatch Poisson framework
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3168
tools:
- name: HashSeq
  role: Generates OneMismatchCluster.txt (parents and one-mismatch children) via inferTrueSequences
    that feeds the fraction-observed computation
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Fits the single-parameter Poisson model with dpois (P/3 per child) and plots
    the fit curve
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:f1d0e132d528715b9a6b60d8b0109e053b3e34e98396118c3f68483a950b17cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# parent-abundance-fit-curve-visualization

## Summary

Visualize the fraction of all possible one-mismatch children observed per parent sequence against parent abundance, together with a single-parameter Poisson fit (fitted error rate P = 10^-4), reproducing the HashSeq manuscript's Fig. 2 relationship.

## When to use

Use when you have HashSeq OneMismatchCluster.txt output from 16S rRNA amplicon data and need to show how one-mismatch child discovery scales with parent read abundance and fit a Poisson error model.

## When NOT to use

- Your goal is variant calling itself rather than characterizing error saturation — use inferTrueSequences/SvTable.txt instead
- You need to model the mean/variance of one-mismatch children counts rather than fraction observed — use HashSeq's makeLoessPlot on childrenProperties.txt, since the Poisson assumption of equal mean and variance was not met in that setting
- Long-read data (e.g., PacBio) where multiple-mismatch sequences are not rare, violating the one-mismatch Poisson framework

## Inputs

- Pre-processed 16S rRNA gene fastq files (zymo mock communities from NCBI PRJEB24409 via FarnazFouladi/Examples)
- OneMismatchCluster.txt from HashSeq inferTrueSequences output

## Outputs

- poisson_error_rate_fit.png (fraction-observed vs. parent abundance with Poisson fit curve)
- poisson_error_rate.txt (fitted single-parameter error rate, ~10^-4)

## How to apply

Run HashSeq's inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000) on pre-processed zymo mock community fastq files (primers removed, reads paired, trimmed to 250 bases) to produce OneMismatchCluster.txt. For each parent, compute the number of distinct observed one-mismatch children divided by 3 × sequence length (the total number of possible one-mismatch children). Fit a single-parameter Poisson model in R using dpois with per-child error probability P/3, estimating P by maximum likelihood. Plot the fitted Poisson prediction of fraction-observed versus parent abundance overlaid on observed per-parent fractions, saving the figure as poisson_error_rate_fit.png and the fitted rate to poisson_error_rate.txt.

## Related tools

- **HashSeq** (Generates OneMismatchCluster.txt (parents and one-mismatch children) via inferTrueSequences that feeds the fraction-observed computation) — https://github.com/FarnazFouladi/HashSeq
- **R** (Fits the single-parameter Poisson model with dpois (P/3 per child) and plots the fit curve) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
inferTrueSequences(inputDir,outputDir,threshold)
```

## Evaluation signals

- Fitted error rate P falls in the low-percent range (~10^-4) typical of Illumina MiSeq amplicon error
- Predicted fraction-observed curve increases smoothly and monotonically with parent abundance
- Overlay visually reproduces the published Fig. 2 relationship between parent abundance and fraction of possible one-mismatch children observed
- Denominator for each parent equals 3 substitutions per base × sequence length
- abundanceThreshold=1000 removes variants with total reads below 1000 across all samples, consistent with the default

## Limitations

- Fits are approximate: 'reasonably well fit across data sets', with slightly better fits for some data sets at slightly different P values
- The Poisson model of equal mean and variance was not met (overdispersion) for controlling one-mismatch children distribution in HashSeq's inference step
- HashSeq considers only one-mismatch variants and does not model indels or two or more mismatches

## Evidence

- [results] The fraction of possible one-mismatch children as a function of parent abundance is reasonably well fit across data sets by a Poisson model with a fitted error rate of P = 10−4: [evidence span withheld — anchor status: not_found]
- [intro] every possible unique one-mismatch variant: "We demonstrate that with enough sequence depth, every possible unique one-mismatch variant for a sequence will be observed."
- [discussion] Poisson equal mean and variance not met: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [readme] sequence variants that have total reads less than 1000 across all samples are removed: "Here we choose a threshold of 1000 which means sequence
variants that have total reads less than 1000 across all samples are
removed."
- [readme] inferTrueSequences outputs OneMismatchCluster.txt: "OneMismatchCluster.txt: This file contains all parents and one-mismatch
children for all clusters."
