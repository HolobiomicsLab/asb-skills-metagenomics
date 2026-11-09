---
name: amplicon-sequence-variant-cluster-analysis
description: Use when evaluating or reproducing an inference test on one-mismatch
  children in HashSeq cluster output (OneMismatchCluster.txt, childrenProperties.txt),
  e.g. to benchmark Poisson-based variant-calling error models.
when_to_use_negative:
- You want a conservative variant caller for production use — the Poisson test is
  anticonservative because its equal mean-variance assumption is violated by overdispersion,
  so it calls nearly all one-mismatch variants as true
- Your goal is de novo variant calling itself — use HashSeq's inferTrueSequences (normal
  distribution + LOESS regression) instead of the Poisson-based inference test
- The data contain true low-abundance variants or two-or-more-mismatch variants —
  HashSeq/Poisson cluster analysis does not address these cases
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3168
tools:
- name: HashSeq
  role: R package that produces the OneMismatchCluster.txt, childrenProperties.txt
    and InferenceTestSignificantChildren.txt files used as inputs and templates for
    the Poisson inference test
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Environment in which the per-child one-sided poisson.test and Benjamini-Hochberg
    FDR correction are run
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:18d8343c798bd0891f382b684b9ae5eaac87947acbc17bb9bd7a53534ec2d9b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# amplicon-sequence-variant-cluster-analysis

## Summary

Analyzes HashSeq's parent/one-mismatch children cluster structure by running a one-sided Poisson test per one-mismatch child against the LOESS-estimated background error rate, with Benjamini-Hochberg FDR correction, to assess whether the Poisson model can discriminate true one-mismatch variants from sequencing-error artifacts.

## When to use

Use when evaluating or reproducing an inference test on one-mismatch children in HashSeq cluster output (OneMismatchCluster.txt, childrenProperties.txt), e.g. to benchmark Poisson-based variant-calling error models.

## When NOT to use

- You want a conservative variant caller for production use — the Poisson test is anticonservative because its equal mean-variance assumption is violated by overdispersion, so it calls nearly all one-mismatch variants as true
- Your goal is de novo variant calling itself — use HashSeq's inferTrueSequences (normal distribution + LOESS regression) instead of the Poisson-based inference test
- The data contain true low-abundance variants or two-or-more-mismatch variants — HashSeq/Poisson cluster analysis does not address these cases

## Inputs

- OneMismatchCluster.txt (parent and one-mismatch children cluster assignments from HashSeq inferTrueSequences)
- childrenProperties.txt (per-cluster mean and standard deviation of one-mismatch children)
- Poisson-model estimated background error rate per cluster

## Outputs

- InferenceTestSignificantChildren.txt (one-mismatch children whose BH-adjusted P values pass the 5% FDR threshold)
- Per-child one-sided Poisson-test P values adjusted by Benjamini-Hochberg at 5% FDR
- Rejection-rate statistic (fraction of one-mismatch children for which the null is rejected, expected >83%)

## How to apply

Load the parent/one-mismatch children cluster assignments from OneMismatchCluster.txt and the per-cluster mean and standard deviation statistics from childrenProperties.txt in the HashSeq output directory. For each one-mismatch child, run a one-sided poisson.test using the Poisson-model error rate as the expected background rate and the child's observed read count as the test statistic. Adjust all per-child P values with the Benjamini-Hochberg procedure at a 5% false discovery rate and write the passing children to InferenceTestSignificantChildren.txt, matching the format produced by inferTrueSequences. The rationale is to test whether the Poisson assumption of equal mean and variance holds: because the one-mismatch children are overdispersed, the test is anticonservative and rejects the null of random sequencing error for more than 83% of one-mismatch children (Fig. S1), explaining why HashSeq instead models background error with a normal distribution combined with LOESS regression.

## Related tools

- **HashSeq** (R package that produces the OneMismatchCluster.txt, childrenProperties.txt and InferenceTestSignificantChildren.txt files used as inputs and templates for the Poisson inference test) — https://github.com/FarnazFouladi/HashSeq
- **R** (Environment in which the per-child one-sided poisson.test and Benjamini-Hochberg FDR correction are run) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
pvals <- sapply(children, function(c) poisson.test(c$reads, r = c$backgroundRate, alternative = "greater")$p.value); sig <- p.adjust(pvals, method = "BH") < 0.05
```

## Evaluation signals

- The null hypothesis of random sequencing error is rejected for more than 83% of one-mismatch children after BH correction at 5% FDR, matching Fig. S1
- InferenceTestSignificantChildren.txt follows the same format as the file produced by inferTrueSequences
- The number of children flagged as significant is very large relative to the total one-mismatch children count (e.g. most of the 230 children in the zymo example cluster-properties table), reflecting anticonservativeness
- Adjusted P values are computed jointly across all one-mismatch children, not per cluster, to reproduce the multiple-hypothesis correction

## Limitations

- The Poisson model is anticonservative because its equal mean-variance assumption is not met (overdispersion), so it fails to control the distribution of one-mismatch children in each cluster
- A negative binomial alternative was rejected because it is undefined when the variance is less than the mean and does not work on log-transformed data
- HashSeq considers only one-mismatch variants and does not model indels, so this cluster-based test cannot assess multi-mismatch or indel artifacts

## Evidence

- [results] The Poisson test rejected the null hypothesis of random sequencing error for more than 83% of one-mismatch children even after multiple-hypothesis correction: [evidence span withheld — anchor status: not_found]
- [discussion] the Poisson model failed to control the distribution of one-mismatch children in each cluster: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [readme] OneMismatchCluster.txt: This file contains all parents and one-mismatch children for all clusters.: "OneMismatchCluster.txt: This file contains all parents and one-mismatch children for all clusters."
- [discussion] Adjusting P values with Benjamini-Hochberg FDR multiple-hypothesis correction: [evidence span withheld — anchor status: not_found]
- [discussion] the negative binomial distribution is not defined when the variance is less than the mean: "the negative binomial distribution is not defined when the variance is less than the mean, and although for the majority of sequence variants the variance is greater than the mean"
