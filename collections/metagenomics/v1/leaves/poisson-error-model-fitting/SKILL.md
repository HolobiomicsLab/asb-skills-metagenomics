---
name: poisson-error-model-fitting
description: Use when you have HashSeq's OneMismatchCluster.txt output (parents plus
  one-mismatch children from 16S rRNA amplicon data) and want to quantify the sequencing
  error rate or check whether child accumulation follows a Poisson process.
when_to_use_negative:
- The goal is de novo variant inference rather than error-rate characterization —
  HashSeq's own pipeline uses a normal distribution + LOESS (makeLoessPlot) for background
  error, and the paper notes a Poisson model failed to control one-mismatch child
  distributions due to overdispersion.
- Data come from long-read platforms such as PacBio, where the rarity of multiple-mismatch
  sequences underlying the one-mismatch child model may not hold.
- You need to model indel errors — HashSeq treats indel events as separate parent
  sequences and the Poisson substitution model covers only nucleotide differences.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: HashSeq
  role: R package run via inferTrueSequences to cluster parents and one-mismatch children
    and emit OneMismatchCluster.txt used for the Poisson fit
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Environment for maximum-likelihood fitting of the Poisson model with dpois
    and generating the fraction-observed vs parent-abundance figure
  repo: https://github.com/FarnazFouladi/HashSeq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:bf438b93e4050d76473d271a29567d70302c1d770bf7117588d6621da2ed8193
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - HashSeq
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# poisson-error-model-fitting

## Summary

Fit a single-parameter Poisson model of the fraction of all possible one-mismatch children observed per parent sequence as a function of parent abundance, estimating the per-nucleotide error rate P (found to be ~10^-4 for Illumina MiSeq 16S data). This reproduces the HashSeq paper's Fig. 2 error-model characterization.

## When to use

Use when you have HashSeq's OneMismatchCluster.txt output (parents plus one-mismatch children from 16S rRNA amplicon data) and want to quantify the sequencing error rate or check whether child accumulation follows a Poisson process.

## When NOT to use

- The goal is de novo variant inference rather than error-rate characterization — HashSeq's own pipeline uses a normal distribution + LOESS (makeLoessPlot) for background error, and the paper notes a Poisson model failed to control one-mismatch child distributions due to overdispersion.
- Data come from long-read platforms such as PacBio, where the rarity of multiple-mismatch sequences underlying the one-mismatch child model may not hold.
- You need to model indel errors — HashSeq treats indel events as separate parent sequences and the Poisson substitution model covers only nucleotide differences.

## Inputs

- Pre-processed 16S rRNA gene FASTQ files (e.g. zymo mock communities from NCBI PRJEB24409; primers removed, forward/reverse reads paired, trimmed to 250 bases)
- OneMismatchCluster.txt produced by HashSeq inferTrueSequences (parents and one-mismatch children per cluster)

## Outputs

- Fitted per-nucleotide error rate P (e.g. poisson_error_rate.txt, expected on the order of 10^-4)
- Plot of fitted Poisson fraction-observed versus parent abundance with observed per-parent fractions overlaid (poisson_error_rate_fit.png), reproducing Fig. 2

## How to apply

Run HashSeq's inferTrueSequences (e.g. abundance threshold=1000) on pre-processed zymo mock-community FASTQ files (primers removed, reads paired, trimmed to 250 bases) to emit OneMismatchCluster.txt. For each parent, compute the number of distinct observed one-mismatch children divided by the total possible (3 substitutions per base times sequence length). Fit the Poisson prediction with per-child error probability P/3, estimating P by maximum likelihood with dpois, and plot predicted fraction-observed versus parent abundance alongside the observed per-parent fractions. The rationale is the paper's finding that child accumulation is 'well modeled by a simple Poisson process', so a single fitted parameter P captures Illumina MiSeq substitution error.

## Related tools

- **HashSeq** (R package run via inferTrueSequences to cluster parents and one-mismatch children and emit OneMismatchCluster.txt used for the Poisson fit) — https://github.com/FarnazFouladi/HashSeq
- **R** (Environment for maximum-likelihood fitting of the Poisson model with dpois and generating the fraction-observed vs parent-abundance figure) — https://github.com/FarnazFouladi/HashSeq

## Examples

```
inferTrueSequences(inputDir,outputDir,threshold)  # then in R: fit <- optim(par=1e-4, fn=function(P) -sum(dpois(obs_kids, lambda=parent_abun*(len*3)*(P/3), log=TRUE)))
```

## Evaluation signals

- The maximum-likelihood fitted error rate P falls near 10^-4, in the low-percent range typical of Illumina MiSeq amplicon error.
- The predicted fraction-observed curve increases smoothly with parent abundance and visually matches the observed per-parent fractions, reproducing Fig. 2.
- The denominator for each parent's fraction-observed equals 3 x sequence length (three substitutions per base).
- Slightly different P values may fit some data sets marginally better; a large deviation from 10^-4 suggests a preprocessing or clustering problem.

## Limitations

- The Poisson assumption of equal mean and variance was not met for the full child-count distribution (overdispersion), which is why HashSeq's main inference uses a normal distribution with LOESS instead.
- Slightly better fits for some data sets occur at slightly different P values, so P is data-set dependent rather than universal.
- Fits depend on the abundance threshold (1,000 reads default; 10,000 for the largest data set) used by inferTrueSequences, and low-JVM-memory settings may require options(java.parameters="-Xmx1024m") for large data sets.

## Evidence

- [discussion] Poisson model well describes child accumulation: "the number of distinct one-mismatch children for each parent sequence can be well modeled by a simple Poisson process, suggesting that when sequence depth is high enough"
- [results] Fitted error rate P = 10^-4 reproduces Fig. 2: [evidence span withheld — anchor status: not_found]
- [methods] Input preprocessing of zymo mock communities: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases"
- [readme] Abundance threshold filtering before fitting: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [readme] OneMismatchCluster.txt contains parents and one-mismatch children: "OneMismatchCluster.txt: This file contains all parents and one-mismatch children for all clusters."
