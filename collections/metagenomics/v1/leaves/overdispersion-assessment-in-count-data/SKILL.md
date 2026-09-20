---
name: overdispersion-assessment-in-count-data
description: Use when evaluating whether per-cluster count data violate Poisson mean-variance
  assumptions, e.g., testing if one-mismatch children exceed the background sequencing-error
  rate estimated by LOESS.
when_to_use_negative:
- When the goal is to call true sequence variants — this test is a diagnostic for
  Poisson anticonservatism, not a variant caller; use HashSeq's normal-distribution/LOESS
  inference instead
- When count variance is not expected to exceed the mean — the negative binomial alternative
  is undefined when variance is less than the mean, but the Poisson may still be inadequate
  for other reasons; verify overdispersion before concluding
- When counts come from technologies with abundant multiple-mismatch or indel errors
  (e.g., PacBio long reads) where the one-mismatch-only cluster model does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3277
tools:
- name: R
  role: Runs the one-sided poisson.test per one-mismatch child and the Benjamini-Hochberg
    FDR adjustment at 5%
  repo: https://github.com/FarnazFouladi/HashSeq
- name: HashSeq
  role: Generates OneMismatchCluster.txt and childrenProperties.txt inputs and the
    reference inferTrueSequences output format for InferenceTestSignificantChildren.txt
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:d6795ef62b1e4d047d44d2dc803548dcf1e81736ecc6081ff7d28f93242141e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3277
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# overdispersion-assessment-in-count-data

## Summary

Assess whether count data (e.g., one-mismatch child read counts in 16S rRNA amplicon clusters) violate the Poisson equal mean-variance assumption by running a one-sided Poisson test against an estimated background error rate; systematic over-rejection of the null indicates overdispersion and that the Poisson model is anticonservative.

## When to use

Use when evaluating whether per-cluster count data violate Poisson mean-variance assumptions, e.g., testing if one-mismatch children exceed the background sequencing-error rate estimated by LOESS.

## When NOT to use

- When the goal is to call true sequence variants — this test is a diagnostic for Poisson anticonservatism, not a variant caller; use HashSeq's normal-distribution/LOESS inference instead
- When count variance is not expected to exceed the mean — the negative binomial alternative is undefined when variance is less than the mean, but the Poisson may still be inadequate for other reasons; verify overdispersion before concluding
- When counts come from technologies with abundant multiple-mismatch or indel errors (e.g., PacBio long reads) where the one-mismatch-only cluster model does not apply

## Inputs

- OneMismatchCluster.txt (parent/one-mismatch children cluster assignments from HashSeq inferTrueSequences)
- childrenProperties.txt (per-cluster mean and standard deviation statistics from HashSeq inferTrueSequences)
- Per-child observed read counts and Poisson-model expected background error rates

## Outputs

- InferenceTestSignificantChildren.txt (one-mismatch children whose BH-adjusted P values pass the 5% FDR threshold, formatted to match inferTrueSequences output)
- Rejection-rate statistic (fraction of one-mismatch children for which the Poisson null is rejected after BH correction)

## How to apply

Load the parent/one-mismatch children cluster assignments from OneMismatchCluster.txt and the per-cluster mean and standard deviation statistics from childrenProperties.txt in the HashSeq output directory. For each one-mismatch child, run a one-sided poisson.test (equation 2) using the Poisson-model error rate as the expected background rate and the child's observed read count as the test statistic. Adjust all per-child P values with the Benjamini-Hochberg procedure at a 5% false discovery rate. If the null hypothesis of random sequencing error is rejected for a large majority of children (more than 83% in the HashSeq evaluation, Fig. S1), conclude the Poisson model is anticonservative because the equal mean-variance assumption is violated by overdispersion — this motivates HashSeq's normal-distribution-plus-LOESS error model instead.

## Related tools

- **R** (Runs the one-sided poisson.test per one-mismatch child and the Benjamini-Hochberg FDR adjustment at 5%) — https://github.com/FarnazFouladi/HashSeq
- **HashSeq** (Generates OneMismatchCluster.txt and childrenProperties.txt inputs and the reference inferTrueSequences output format for InferenceTestSignificantChildren.txt) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
pvals <- sapply(children, function(c) poisson.test(c$count, r = c$backgroundRate, alternative = "greater")$p.value); sig <- p.adjust(pvals, method = "BH") < 0.05
```

## Evaluation signals

- Null hypothesis of random sequencing error is rejected for more than 83% of one-mismatch children after BH correction, reproducing the Fig. S1 anticonservatism result
- InferenceTestSignificantChildren.txt format matches the file produced by HashSeq inferTrueSequences
- Every one-mismatch child receives a one-sided Poisson P value based on the child's observed read count versus the cluster's estimated background error rate
- BH adjustment is applied across all per-child P values at 5% FDR, not per-cluster

## Limitations

- The Poisson test is anticonservative because the equal mean-variance assumption is violated by overdispersion, so it calls nearly all one-mismatch variants as true
- The negative binomial alternative is undefined when variance is less than the mean and does not work on log-transformed data, motivating the normal-distribution/LOESS approach
- The test operates on one-mismatch children only; HashSeq does not consider two or more mismatches and does not model indels, which are treated as separate parent sequences

## Evidence

- [other] Load the parent/one-mismatch children cluster assignments from OneMismatchCluster.txt and the per-cluster mean and standard deviation statistics from childrenProperties.txt: [evidence span withheld — anchor status: not_found]
- [other] The Poisson test rejected the null hypothesis of random sequencing error for more than 83% of one-mismatch children even after multiple-hypothesis correction: [evidence span withheld — anchor status: not_found]
- [discussion] A Poisson-based model failed to control the distribution of one-mismatch children because the Poisson assumption of equal mean and variance was not met (overdispersion): "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [readme] OneMismatchCluster.txt and childrenProperties.txt are standard outputs of inferTrueSequences: "OneMismatchCluster.txt: This file contains all parents and one-mismatch children for all clusters."
- [readme] childrenProperties.txt contains mean and standard deviation for each cluster: "childrenProperties.txt: This file contains mean and standard deviation for each cluster."
