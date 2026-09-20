---
name: one-mismatch-variant-enumeration
description: Use when you have pre-processed 16S rRNA gene fastq reads (primers removed,
  paired, trimmed to 250 bases) and need to enumerate one-mismatch children per parent
  and model their observation rate versus parent abundance.
when_to_use_negative:
- You need to detect true low-abundance variants — HashSeq with a 1,000-read threshold
  is deliberately conservative and insensitive; the article recommends DADA2 for low-abundance
  sequences
- Your data contain indel variation or multi-mismatch variants relative to a parent
  — HashSeq only considers exactly one-mismatch children and treats indels as separate
  parent sequences; Deblur or alignment-based methods are suggested instead
- Long-read data (e.g. PacBio), where the assumption that multiple-mismatch sequences
  are rare may not hold
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: R package providing inferTrueSequences for HashMap-based one-mismatch clustering
    of 16S rRNA sequences and LOESS/normal-distribution background error estimation
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment for HashSeq and for the dpois-based Poisson fit and poisson.test
    inference testing
- name: Illumina MiSeq
  role: Instrument generating the 16S rRNA amplicon reads whose substitution error
    rate (P ~ 10^-4) the Poisson model estimates
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:26b0e1d0e7810548e882cee45c18ff4a4e4c1d81bf096119e2430b1453003ae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - Illumina MiSeq
  - R
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# one-mismatch-variant-enumeration

## Summary

Clusters 16S rRNA amplicon sequences into parent sequences and their one-nucleotide-difference children with HashSeq, then models the fraction of all possible one-mismatch children observed per parent as a single-parameter Poisson process with fitted per-nucleotide error rate P (P = 10^-4 in the paper's mock communities).

## When to use

Use when you have pre-processed 16S rRNA gene fastq reads (primers removed, paired, trimmed to 250 bases) and need to enumerate one-mismatch children per parent and model their observation rate versus parent abundance.

## When NOT to use

- You need to detect true low-abundance variants — HashSeq with a 1,000-read threshold is deliberately conservative and insensitive; the article recommends DADA2 for low-abundance sequences
- Your data contain indel variation or multi-mismatch variants relative to a parent — HashSeq only considers exactly one-mismatch children and treats indels as separate parent sequences; Deblur or alignment-based methods are suggested instead
- Long-read data (e.g. PacBio), where the assumption that multiple-mismatch sequences are rare may not hold

## Inputs

- Pre-processed 16S rRNA gene fastq files (primer-removed, forward/reverse paired, trimmed to 250 bases), e.g. zymo mock communities from NCBI PRJEB24409
- HashSeq output OneMismatchCluster.txt (parent and one-mismatch children per cluster)
- HashSeq output childrenProperties.txt (per-cluster mean and standard deviation of one-mismatch children)

## Outputs

- OneMismatchCluster.txt listing parents and one-mismatch children for all clusters
- childrenProperties.txt with mean and standard deviation per cluster
- Fitted single-parameter Poisson error rate (P, expected ~10^-4 for Illumina MiSeq) and figure of fraction of possible one-mismatch children observed versus parent abundance
- Poisson-test result: fraction of one-mismatch children with rejected error null after BH FDR correction at 5% (paper reports >83%)

## How to apply

Download the pre-processed zymo mock community fastq files (NCBI project PRJEB24409) from the FarnazFouladi/Examples repository, then run HashSeq's inferTrueSequences(inputDir, outputDir, threshold=1000), which clusters sequences that differ by one nucleotide using a fast HashMap-based approach and emits OneMismatchCluster.txt (parent/child assignments) and childrenProperties.txt (per-cluster mean and standard deviation). From OneMismatchCluster.txt, compute for each parent the number of distinct observed one-mismatch children divided by the total number of possible one-mismatch children (3 substitutions per base times the sequence length). Fit the single-parameter Poisson model in R with dpois, using per-child error probability P/3, estimating the error rate P by maximum likelihood as a function of parent abundance. Visualize the fitted fraction-observed curve against observed per-parent fractions to reproduce the paper's Fig. 2 relationship; optionally test each child against the background rate with a one-sided poisson.test and Benjamini-Hochberg FDR correction at 5%.

## Related tools

- **HashSeq** (R package providing inferTrueSequences for HashMap-based one-mismatch clustering of 16S rRNA sequences and LOESS/normal-distribution background error estimation) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment for HashSeq and for the dpois-based Poisson fit and poisson.test inference testing)
- **Illumina MiSeq** (Instrument generating the 16S rRNA amplicon reads whose substitution error rate (P ~ 10^-4) the Poisson model estimates)

## Examples

```
inferTrueSequences(inputDir, outputDir, threshold=1000)
```

## Evaluation signals

- The maximum-likelihood fitted error rate P falls in the low-percent/sub-percent range consistent with Illumina MiSeq amplicon error (~10^-4 in the paper)
- The predicted fraction-of-possible-one-mismatch-children-observed curve increases smoothly and monotonically with parent abundance, visually matching the published Fig. 2 relationship
- The one-sided poisson.test with BH FDR correction at 5% rejects the random-error null for more than 83% of one-mismatch children (Fig. S1), confirming anticonservatism of the Poisson test
- inferTrueSequences output tables appear with expected structure (e.g. OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt)
- Per-cluster children mean and standard deviation plotted via makeLoessPlot confirm the LOESS regression fits well above the chosen abundance threshold (1,000 reads; 10,000 for the largest data set)

## Limitations

- The Poisson test is anticonservative: it rejected the null for >83% of one-mismatch children because the Poisson equal mean-variance assumption is violated by overdispersion, which motivated HashSeq's normal-distribution plus LOESS approach
- The negative binomial alternative is undefined when variance is less than the mean, restricting distribution choices for the background error model
- HashSeq is insensitive for true low-abundance sequences, does not model two or more mismatches, indels, or compositionality, and the rarity-of-multiple-mismatches assumption may fail for long-read technologies such as PacBio
- The default JVM heap size (512MB) may be insufficient for large data sets; increase via options(java.parameters) before attaching libraries

## Evidence

- [results] The fraction of possible one-mismatch children as a function of parent abundance is reasonably well fit across data sets by a Poisson model with a fitted error rate of P = 10−4: [evidence span withheld — anchor status: not_found]
- [readme] HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference.: "HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference."
- [readme] sequence variants that have total reads less than 1000 across all samples are removed: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [discussion] the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met.: "the Poisson model failed to control the distribution of one-mismatch children in each cluster, presumably because the Poisson assumption of equal mean and variance was not met."
- [discussion] The number of distinct one-mismatch children for each parent sequence can be modeled by a simple Poisson process: "the number of distinct one-mismatch children for each parent sequence can be well modeled by a simple Poisson process, suggesting that when sequence depth is high enough, every possible one-mismatch"
