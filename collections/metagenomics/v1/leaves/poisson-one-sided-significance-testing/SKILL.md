---
name: poisson-one-sided-significance-testing
description: Use when evaluating whether a Poisson error model with equal mean-variance
  assumption adequately controls one-mismatch child distributions in amplicon variant
  calling, or when reproducing the Fig. S1 anticonservativeness benchmark.
when_to_use_negative:
- When the goal is to call true variants in a production pipeline — the Poisson test
  is anticonservative (calls nearly all one-mismatch variants as true) and HashSeq
  instead uses a normal-distribution-plus-LOESS error model
- When the child count data show variance less than the mean (underdispersion), where
  alternate models like the negative binomial are undefined and a normal distribution
  is preferred
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Language used to run one-sided poisson.test per child and apply Benjamini-Hochberg
    FDR correction
  repo: https://github.com/FarnazFouladi/HashSeq
- name: HashSeq
  role: R package producing OneMismatchCluster.txt and childrenProperties.txt consumed
    by this test; its inferTrueSequences output format is matched
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:63ccd68c8db520a22c7158c088fba4e1933186e93751e153aaa8549b6d601804
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# poisson-one-sided-significance-testing

## Summary

A diagnostic procedure from the HashSeq paper that runs one-sided poisson.test calls on one-mismatch children, with Benjamini-Hochberg FDR correction at 5%, to demonstrate that the Poisson model is anticonservative (rejecting the null for >83% of children) because of overdispersion.

## When to use

Use when evaluating whether a Poisson error model with equal mean-variance assumption adequately controls one-mismatch child distributions in amplicon variant calling, or when reproducing the Fig. S1 anticonservativeness benchmark.

## When NOT to use

- When the goal is to call true variants in a production pipeline — the Poisson test is anticonservative (calls nearly all one-mismatch variants as true) and HashSeq instead uses a normal-distribution-plus-LOESS error model
- When the child count data show variance less than the mean (underdispersion), where alternate models like the negative binomial are undefined and a normal distribution is preferred

## Inputs

- OneMismatchCluster.txt (HashSeq output listing all parents and one-mismatch children per cluster)
- childrenProperties.txt (HashSeq output with per-cluster mean and standard deviation of one-mismatch children)
- Poisson-model estimated background error rate per child cluster

## Outputs

- Per-child one-sided Poisson test P values with Benjamini-Hochberg 5% FDR adjustment
- InferenceTestSignificantChildren.txt (one-mismatch children passing the 5% FDR threshold, formatted like inferTrueSequences output)

## How to apply

Load the parent/one-mismatch children cluster assignments from OneMismatchCluster.txt and the per-cluster mean and standard deviation statistics from childrenProperties.txt in the HashSeq output directory. For each one-mismatch child, run a one-sided poisson.test (equation 2) using the Poisson-model error rate as the expected background rate and the child's observed read count as the test statistic. Adjust all per-child P values with the Benjamini-Hochberg procedure at a 5% false discovery rate. Write children whose adjusted P values pass the 5% FDR threshold to InferenceTestSignificantChildren.txt, matching the format produced by inferTrueSequences. Validate by confirming the null hypothesis of random sequencing error is rejected for more than 83% of one-mismatch children, which demonstrates the Poisson model is anticonservative because its equal mean-variance assumption is violated by overdispersion.

## Related tools

- **R** (Language used to run one-sided poisson.test per child and apply Benjamini-Hochberg FDR correction) — https://github.com/FarnazFouladi/HashSeq
- **HashSeq** (R package producing OneMismatchCluster.txt and childrenProperties.txt consumed by this test; its inferTrueSequences output format is matched) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
pvals <- sapply(childCounts, function(n) poisson.test(n, r = bgRate, alternative = "greater")$p.value); padj <- p.adjust(pvals, method = "BH"); sum(padj < 0.05) / length(padj)
```

## Evaluation signals

- More than 83% of one-mismatch children have their null hypothesis of random sequencing error rejected even after Benjamini-Hochberg 5% FDR correction (matching Fig. S1)
- InferenceTestSignificantChildren.txt matches the format of the file produced by inferTrueSequences
- Observed overdispersion (variance exceeding the Poisson mean) is present in the one-mismatch child counts, explaining the anticonservativeness
- Adjusted P values are computed across all per-child tests jointly, not per-cluster, before thresholding at 5% FDR

## Limitations

- The Poisson test is anticonservative: it rejected the null for more than 83% of one-mismatch children because the Poisson assumption of equal mean and variance is not met (overdispersion)
- The negative binomial alternative is undefined when variance is less than the mean, which is why HashSeq's production model uses a normal distribution with LOESS regression instead

## Evidence

- [results] The Poisson test rejected the null hypothesis of random sequencing error for more than 83% of one-mismatch children even after multiple-hypothesis correction: [evidence span withheld — anchor status: not_found]
- [discussion] Overdispersion violates equal mean-variance: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [readme] HashSeq output files described in README: "OneMismatchCluster.txt: This file contains all parents and one-mismatch children for all clusters."
- [discussion] Negative binomial undefined below unit dispersion: "the negative binomial distribution is not defined when the variance is less than the mean"
