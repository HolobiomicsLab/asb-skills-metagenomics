---
name: benjamini-hochberg-fdr-correction
description: Use when evaluating one-mismatch children from HashSeq's OneMismatchCluster.txt
  against a background error rate with poisson.test, and per-child P values must be
  multiple-hypothesis corrected with BH at 5% FDR before writing InferenceTestSignificantChildren.txt.
when_to_use_negative:
- 'You need a well-controlled variant caller for production: the Poisson test is anticonservative
  (>83% rejection) because of overdispersion; use HashSeq''s normal-distribution +
  LOESS inference instead'
- 'True low-abundance variant detection: HashSeq-based inference is not sensitive
  there, and more sensitive algorithms such as DADA2 are recommended'
- The data involve multiple mismatches or indels, which the one-mismatch Poisson framework
  does not model
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Runs the one-sided poisson.test per child and the Benjamini-Hochberg adjustment
    via p.adjust
  repo: https://github.com/FarnazFouladi/HashSeq
- name: HashSeq
  role: Produces OneMismatchCluster.txt, childrenProperties.txt, and the inferTrueSequences-format
    InferenceTestSignificantChildren.txt that this skill replicates
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:fe2136f7e8b45475e10b0b162272f020d2d2c89f04e39d7c384a11bc76c5a135
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benjamini-hochberg-fdr-correction

## Summary

Applies one-sided Poisson tests with Benjamini-Hochberg FDR correction (5% threshold) to classify one-mismatch child sequences in HashSeq clusters as true variants versus sequencing-error artifacts, demonstrating the anticonservatism of the Poisson error model.

## When to use

Use when evaluating one-mismatch children from HashSeq's OneMismatchCluster.txt against a background error rate with poisson.test, and per-child P values must be multiple-hypothesis corrected with BH at 5% FDR before writing InferenceTestSignificantChildren.txt.

## When NOT to use

- You need a well-controlled variant caller for production: the Poisson test is anticonservative (>83% rejection) because of overdispersion; use HashSeq's normal-distribution + LOESS inference instead
- True low-abundance variant detection: HashSeq-based inference is not sensitive there, and more sensitive algorithms such as DADA2 are recommended
- The data involve multiple mismatches or indels, which the one-mismatch Poisson framework does not model

## Inputs

- OneMismatchCluster.txt (parents and one-mismatch children for all clusters from HashSeq)
- childrenProperties.txt (per-cluster mean and standard deviation statistics from HashSeq)
- Poisson-model estimated background error rate per cluster (equation 2)
- Per-child observed read counts

## Outputs

- InferenceTestSignificantChildren.txt (one-mismatch children passing 5% FDR, in inferTrueSequences format)
- Per-child BH-adjusted P values
- Fraction of one-mismatch children for which the null hypothesis of random sequencing error is rejected

## How to apply

Load the parent/one-mismatch children cluster assignments from OneMismatchCluster.txt and the per-cluster mean and standard deviation statistics from childrenProperties.txt in the HashSeq output directory. For each one-mismatch child, run a one-sided poisson.test (equation 2) in R, using the Poisson-model background error rate as the expected rate and the child's observed read count as the test statistic. Adjust all per-child P values simultaneously with p.adjust(method="BH") so that the correction spans the full family of tests. Write the children whose adjusted P values pass the 5% FDR threshold to InferenceTestSignificantChildren.txt, matching the format produced by inferTrueSequences. The rationale is diagnostic: the Poisson test assumes equal mean and variance, but one-mismatch children are overdispersed, so the test is expected to reject the null for more than 83% of one-mismatch children even after BH correction — evidence that the Poisson model calls nearly all one-mismatch variants as true.

## Related tools

- **R** (Runs the one-sided poisson.test per child and the Benjamini-Hochberg adjustment via p.adjust) — https://github.com/FarnazFouladi/HashSeq
- **HashSeq** (Produces OneMismatchCluster.txt, childrenProperties.txt, and the inferTrueSequences-format InferenceTestSignificantChildren.txt that this skill replicates) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
pvals <- sapply(children, function(c) poisson.test(c, r = bgRate, alternative = "greater")$p.value); adjP <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- The null hypothesis of random sequencing error is rejected for more than 83% of one-mismatch children after BH correction (as reported in Fig. S1)
- The rejection fraction remains high even after multiple-hypothesis correction, demonstrating anticonservatism of the Poisson model
- InferenceTestSignificantChildren.txt matches the file format produced by inferTrueSequences
- All per-child P values are adjusted in a single BH call across the full family of one-mismatch children, at a 5% FDR threshold

## Limitations

- The Poisson assumption of equal mean and variance is violated by overdispersion, making the test anticonservative — it calls nearly all one-mismatch variants as true
- The negative binomial alternative is not defined when variance is less than the mean, which constrains overdispersed-model choices on log-transformed data
- Only one-mismatch variants are considered; two or more mismatches, indels, and long-read error profiles (e.g., PacBio) are outside this framework

## Evidence

- [discussion] Poisson model anticonservatism due to overdispersion: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [discussion] P values for downstream FDR correction: "which can be fed into statistical models to produce FDR-corrected P values"
- [readme] InferenceTestSignificantChildren.txt contents: "InferenceTestSignificantChildren.txt: This file includes one-mismatch children that were significant in an inference test."
- [other] BH adjustment at 5% FDR in the reproducibility workflow: [evidence span withheld — anchor status: not_found]
