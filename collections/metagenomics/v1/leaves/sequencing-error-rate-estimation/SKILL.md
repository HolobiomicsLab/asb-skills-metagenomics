---
name: sequencing-error-rate-estimation
description: Use when you have parent sequences and their one-mismatch children from
  amplicon fastq data and need to estimate the Illumina substitution error rate or
  verify that one-mismatch child accumulation follows a Poisson process.
when_to_use_negative:
- Your goal is variant calling itself rather than error-rate characterization; HashSeq's
  own inference uses a normal distribution with LOESS regression for background error
  instead of the Poisson model
- Data come from long-read technologies such as PacBio, where the assumption of rarity
  of multiple-mismatch sequences may not hold
- You need indel error modeling; HashSeq does not explicitly model insertions or deletions
  and treats indel events as separate parent sequences
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3168
tools:
- name: HashSeq
  role: R package whose inferTrueSequences function clusters sequences into parents
    and one-mismatch children and emits OneMismatchCluster.txt used for the Poisson
    error-rate fit
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Environment for fitting the single-parameter Poisson model with dpois and
    producing the fraction-observed versus parent abundance plot
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:44a52d57104edff8f4194419834eb86bddad8f77283a926e22ff612c7d92d717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sequencing-error-rate-estimation

## Summary

Fits a single-parameter Poisson model to the fraction of all possible one-mismatch children observed per parent 16S rRNA sequence versus parent abundance, yielding a per-nucleotide sequencing error rate (approximately P = 10^-4 on Illumina MiSeq zymo mock community data). This quantifies the background error process that HashSeq uses to infer true sequence variants.

## When to use

Use when you have parent sequences and their one-mismatch children from amplicon fastq data and need to estimate the Illumina substitution error rate or verify that one-mismatch child accumulation follows a Poisson process.

## When NOT to use

- Your goal is variant calling itself rather than error-rate characterization; HashSeq's own inference uses a normal distribution with LOESS regression for background error instead of the Poisson model
- Data come from long-read technologies such as PacBio, where the assumption of rarity of multiple-mismatch sequences may not hold
- You need indel error modeling; HashSeq does not explicitly model insertions or deletions and treats indel events as separate parent sequences

## Inputs

- Pre-processed 16S rRNA gene fastq files (primers removed, reads paired, trimmed to 250 bases; e.g. zymo mock communities from PRJEB24409)
- OneMismatchCluster.txt output from HashSeq inferTrueSequences (parents and one-mismatch children per cluster)

## Outputs

- Fitted per-nucleotide error rate P (e.g. saved to poisson_error_rate.txt)
- Plot of Poisson-predicted fraction of possible one-mismatch children observed vs parent abundance with observed points (e.g. poisson_error_rate_fit.png)

## How to apply

Obtain pre-processed 16S rRNA gene fastq reads (zymo mock communities, NCBI PRJEB24409; primers removed, forward/reverse reads paired, trimmed to 250 bases) and run HashSeq's inferTrueSequences(inputDir, outputDir, threshold=1000) to cluster parents and one-mismatch children and emit OneMismatchCluster.txt. For each parent, compute the number of distinct observed one-mismatch children divided by the total possible (3 substitutions per base times sequence length). Fit a Poisson model in R with dpois using per-child error probability P/3, estimating P by maximum likelihood as a function of parent abundance. Plot the fitted curve against observed per-parent fractions and report the fitted P to a text file.

## Related tools

- **HashSeq** (R package whose inferTrueSequences function clusters sequences into parents and one-mismatch children and emits OneMismatchCluster.txt used for the Poisson error-rate fit) — https://github.com/FarnazFouladi/HashSeq
- **R** (Environment for fitting the single-parameter Poisson model with dpois and producing the fraction-observed versus parent abundance plot) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
inferTrueSequences(inputDir, outputDir, threshold=1000); fit <- optim(par=1e-4, fn=function(P) -sum(dpois(observed, P/3 * abundance, log=TRUE)), method="Brent", lower=0, upper=1)
```

## Evaluation signals

- Fitted error rate P falls around 10^-4, in the low-percent range typical of Illumina MiSeq amplicon error
- The predicted fraction-of-possible-children-observed curve increases smoothly with parent abundance and visually reproduces the published Fig. 2 relationship
- Slightly different P values (e.g. per data set) may fit some data sets marginally better; deviations should be small, not orders of magnitude
- OneMismatchCluster.txt must show parents with children counts that rise with parent abundance toward saturation at 3 x sequence-length possible children

## Limitations

- The Poisson model describes distinct one-mismatch child accumulation but the Poisson assumption of equal mean and variance was not met for cluster-level child distributions (overdispersion), which is why HashSeq itself uses a normal distribution with LOESS
- Slightly better fits for some data sets occur at slightly different P values, so P = 10^-4 is approximate, not universal
- The approach considers only one-mismatch variants and assumes multiple-mismatch sequences are rare, which may fail for long-read technologies such as PacBio

## Evidence

- [results] The fraction of possible one-mismatch children observed per parent follows a Poisson process with fitted error rate P = 10^-4: [evidence span withheld — anchor status: not_found]
- [methods] Mock community data pre-processing and source: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases"
- [methods] Abundance filtering threshold before inference: "threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed"
- [discussion] Poisson modeling of one-mismatch children: "the number of distinct one-mismatch children for each parent sequence can be well modeled by a simple Poisson process"
- [discussion] Why Poisson was not used for HashSeq's error model: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
