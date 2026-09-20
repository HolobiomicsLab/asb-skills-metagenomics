---
name: scatterplot-visualization-of-cluster-properties
description: Use when you have run HashSeq's inferTrueSequences on 16S rRNA amplicon
  data and need to inspect childrenProperties.txt to set or validate an abundance
  threshold (e.g., 1,000 reads) above which the LOESS background error model fits
  well.
when_to_use_negative:
- No childrenProperties.txt exists yet — run inferTrueSequences first to generate
  cluster-level children statistics.
- The goal is detecting true low-abundance sequence variants — HashSeq's threshold-based
  approach is conservative and not sensitive for low-abundance variants; the article
  recommends DADA2 for that purpose.
- Data come from long-read platforms such as PacBio, where the one-mismatch/rarity
  assumptions underlying the parent-child cluster model may not hold.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: HashSeq
  role: Provides makeLoessPlot() which reads childrenProperties.txt, fits and overlays
    the LOESS regression, and renders the scatterplot
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which the HashSeq package's makeLoessPlot function
    is invoked
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Alternative variant caller recommended for low-abundance variants outside
    the scope of this visualization/thresholding workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:427b846225812092cf15d8299ba6a27911a12604fb10d3d70bdae5211010e906
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Scatterplot visualization of cluster properties

## Summary

Generates scatterplots of the mean and standard deviation of one-mismatch children versus parent abundance using HashSeq's makeLoessPlot, overlays a LOESS regression fit, and saves the plot (loess_plot.png) to guide the choice of the abundance filtering threshold.

## When to use

Use when you have run HashSeq's inferTrueSequences on 16S rRNA amplicon data and need to inspect childrenProperties.txt to set or validate an abundance threshold (e.g., 1,000 reads) above which the LOESS background error model fits well.

## When NOT to use

- No childrenProperties.txt exists yet — run inferTrueSequences first to generate cluster-level children statistics.
- The goal is detecting true low-abundance sequence variants — HashSeq's threshold-based approach is conservative and not sensitive for low-abundance variants; the article recommends DADA2 for that purpose.
- Data come from long-read platforms such as PacBio, where the one-mismatch/rarity assumptions underlying the parent-child cluster model may not hold.

## Inputs

- childrenProperties.txt (HashSeq output holding mean and standard deviation of one-mismatch children per cluster plus parent abundances)
- HashSeq output directory containing the inferTrueSequences results

## Outputs

- loess_plot.png — scatterplot of mean and SD of one-mismatch children vs. parent abundance with overlaid LOESS fit
- Visual confirmation of the abundance threshold above which LOESS regression is a good fit

## How to apply

After running inferTrueSequences(inputDir, outputDir, threshold), read the generated childrenProperties.txt, which contains the mean and standard deviation of one-mismatch children per cluster along with their parents' abundances. Call HashSeq's makeLoessPlot(outputDir, fileName = "childrenProperties.txt") in R, which fits a LOESS (locally estimated scatterplot smoothing) regression and overlays it on the scatterplot of mean/standard deviation versus parent abundance. The plot is restricted to parents above the abundance threshold — 1,000 total reads for most data sets and 10,000 for the largest — because above this threshold the LOESS regression is a good fit, especially for high-abundance parents (>1,000 reads) across all six data sets. Use scattermore for efficient rendering of dense point clouds and save the result as loess_plot.png. Inspect whether the LOESS curve tracks the mean–standard deviation relationship to confirm the normal-plus-LOESS background model (as in Fig. 5) is appropriate for the chosen threshold.

## Related tools

- **HashSeq** (Provides makeLoessPlot() which reads childrenProperties.txt, fits and overlays the LOESS regression, and renders the scatterplot) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which the HashSeq package's makeLoessPlot function is invoked) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Alternative variant caller recommended for low-abundance variants outside the scope of this visualization/thresholding workflow)

## Examples

```
library(HashSeq); makeLoessPlot(outputDir, fileName = "childrenProperties.txt")
```

## Evaluation signals

- The rendered plot shows the LOESS fit tracking the mean–standard deviation relationship for parents above the 1,000-read (or 10,000 for the largest data set) threshold, consistent with Fig. 5 of the manuscript.
- loess_plot.png is produced in the output directory without errors from makeLoessPlot.
- childrenProperties.txt used as input contains both mean and standard deviation columns plus parent abundances for each cluster.
- Scatterplot points render efficiently (via scattermore) even for large six-data-set-scale point clouds.
- The relationship is visually strongest for high-abundance parents (>1,000 reads), matching the reported good LOESS fit across all six data sets.

## Limitations

- The LOESS fit is only good above the filtering threshold; below ~1,000 reads (10,000 for the largest data set) the mean–SD relationship is not well modeled.
- This visualization supports HashSeq's conservative one-mismatch cluster model only; HashSeq is not sensitive for true low-abundance sequences and does not model indels, multiple mismatches, or compositionality.
- Default JVM heap size (512MB) may be insufficient for large data sets; increase memory with options(java.parameters) before loading libraries.

## Evidence

- [methods] We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents.: "We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents"
- [readme] This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data.: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data"
- [methods] makeLoessPlot(outputDir,fileName = "childrenProperties.txt"): "makeLoessPlot(outputDir,fileName = "childrenProperties.txt")"
- [results] The mean abundance of children in each cluster is well fit by a LOESS function of parent abundance, especially for high-abundance parents (>1,000 reads) across all six data sets.: [evidence span withheld — anchor status: not_found]
- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
