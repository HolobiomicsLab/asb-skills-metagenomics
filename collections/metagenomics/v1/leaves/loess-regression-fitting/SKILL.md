---
name: loess-regression-fitting
description: Use when you need to model the mean and standard deviation of one-mismatch
  child abundances as a smooth function of parent abundance, or to empirically choose
  the abundance threshold above which a LOESS background error model is a good fit.
when_to_use_negative:
- The input is not HashSeq cluster output (e.g., you already have a count/SvTable
  or feature table), since childrenProperties.txt is the required input.
- You need to detect true low-abundance sequence variants — HashSeq's LOESS-based
  background model is conservative and insensitive below the 1,000-read threshold;
  use more sensitive callers such as DADA2.
- Variants differ by two or more mismatches, contain indels, or come from long-read
  technologies like PacBio where the one-mismatch assumption of the background model
  fails.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0611
tools:
- name: HashSeq
  role: Provides inferTrueSequences (generates childrenProperties.txt) and makeLoessPlot
    (fits and renders the LOESS plot); installed via devtools::install_github("FarnazFouladi/HashSeq")
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which the HashSeq package's makeLoessPlot is invoked
- name: scattermore
  role: Used inside makeLoessPlot for efficient rendering of the scatterplot points
    before saving loess_plot.png
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:598610d965ffc8440654a6b891dae2337f226644f9e1a9fce89cd70a1fd39e43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  - scattermore
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# loess-regression-fitting

## Summary

Fit and visualize a LOESS (locally estimated scatterplot smoothing) regression of one-mismatch child variant mean and standard deviation against parent sequencing depth, as used in HashSeq to justify the depth-dependent normal-plus-LOESS background error model for 16S rRNA gene variant calling.

## When to use

Use when you need to model the mean and standard deviation of one-mismatch child abundances as a smooth function of parent abundance, or to empirically choose the abundance threshold above which a LOESS background error model is a good fit.

## When NOT to use

- The input is not HashSeq cluster output (e.g., you already have a count/SvTable or feature table), since childrenProperties.txt is the required input.
- You need to detect true low-abundance sequence variants — HashSeq's LOESS-based background model is conservative and insensitive below the 1,000-read threshold; use more sensitive callers such as DADA2.
- Variants differ by two or more mismatches, contain indels, or come from long-read technologies like PacBio where the one-mismatch assumption of the background model fails.

## Inputs

- childrenProperties.txt (HashSeq output with mean and standard deviation of one-mismatch children per cluster and parent abundances)
- outputDir (HashSeq output directory containing childrenProperties.txt)

## Outputs

- loess_plot.png (scatterplot of children mean/SD versus parent abundance with overlaid LOESS fit)
- Empirically justified parent-abundance filtering threshold (1,000 reads; 10,000 for the largest data set)

## How to apply

Run HashSeq's inferTrueSequences on the FASTQ input to generate childrenProperties.txt, which holds the mean and standard deviation of one-mismatch children per cluster along with their parents' abundances. Call makeLoessPlot(outputDir, fileName = "childrenProperties.txt") in R to fit the LOESS regression and overlay it on the scatterplot of children mean/standard deviation versus parent abundance. The plot renders points efficiently with scattermore and is saved as loess_plot.png. Read the plot to determine the abundance above which LOESS tracks the data: the paper used 1,000 total reads for all data sets except the largest (10,000 reads), and reports the fit is especially good for high-abundance parents (>1,000 reads) across all six data sets, supporting the normal-distribution-plus-LOESS background model. For large data sets, increase the Java heap via options(java.parameters="-Xmx1024m") before attaching libraries.

## Related tools

- **HashSeq** (Provides inferTrueSequences (generates childrenProperties.txt) and makeLoessPlot (fits and renders the LOESS plot); installed via devtools::install_github("FarnazFouladi/HashSeq")) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which the HashSeq package's makeLoessPlot is invoked)
- **scattermore** (Used inside makeLoessPlot for efficient rendering of the scatterplot points before saving loess_plot.png)

## Examples

```
makeLoessPlot(outputDir, fileName = "childrenProperties.txt")
```

## Evaluation signals

- loess_plot.png is produced in the output directory and shows both mean and standard deviation of one-mismatch children plotted against parent abundance with an overlaid LOESS curve.
- The LOESS curve visibly tracks the mean-SD relationship for parents above the 1,000-read threshold, consistent with the reported good fit in Fig. 5 of the manuscript.
- childrenProperties.txt exists and is non-empty before calling makeLoessPlot (generated by inferTrueSequences; in the zymo example it has 230 rows and 10 columns).
- The chosen filtering threshold from reading the plot matches the paper's setting: 1,000 total reads across all samples (10,000 for the largest data set).
- For large data sets, the Java heap was increased via options(java.parameters) after restarting R and before attaching any libraries, avoiding JVM memory failures.

## Limitations

- The LOESS fit is only reliable above the abundance threshold (1,000 reads default; 10,000 for the largest data set), so the resulting background model is not sensitive for true low-abundance sequences.
- The underlying model considers only one-mismatch children and does not model indels, two-or-more-mismatch variants, or long-read (e.g., PacBio) error profiles, and does not address compositionality.
- The default 512MB Java heap may be insufficient for large data sets; options(java.parameters) must be set after restarting R and before attaching libraries.

## Evidence

- [readme] We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents: "We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents"
- [readme] This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data"
- [methods] makeLoessPlot(outputDir,fileName = "childrenProperties.txt"): "makeLoessPlot(outputDir,fileName = "childrenProperties.txt")"
- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [results] The mean abundance of children in each cluster is well fit by a LOESS function of parent abundance: [evidence span withheld — anchor status: not_found]
