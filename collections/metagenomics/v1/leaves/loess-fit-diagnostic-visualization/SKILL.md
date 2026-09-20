---
name: loess-fit-diagnostic-visualization
description: Use when running HashSeq's inferTrueSequences on 16S rRNA gene FASTQ
  data and you need to verify the LOESS regression is a good fit above the abundance
  threshold before trusting variant calls.
when_to_use_negative:
- Input is a raw FASTQ directory that has not yet been processed by inferTrueSequences,
  so childrenProperties.txt does not exist.
- The goal is low-abundance variant detection; HashSeq (and this diagnostic) is not
  sensitive for true low-abundance sequences and DADA2 is recommended instead.
- Data are long-read (e.g. PacBio) or contain frequent indels/multiple mismatches,
  which violate HashSeq's one-mismatch, indel-as-separate-parent assumptions.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: HashSeq
  role: R package providing inferTrueSequences (to generate childrenProperties.txt)
    and makeLoessPlot (LOESS diagnostic visualization)
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which HashSeq is installed via devtools and the diagnostic
    plots are generated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:a263f8d9416ddaa3573338710c9cb4e03b009cf9e23e1e3cd39da5a31c0ddfc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# loess-fit-diagnostic-visualization

## Summary

Visualize the relationship between the mean and standard deviation of one-mismatch children and parent abundances from HashSeq output to confirm the LOESS regression used in the normal-distribution error model fits above the filtering threshold.

## When to use

Use when running HashSeq's inferTrueSequences on 16S rRNA gene FASTQ data and you need to verify the LOESS regression is a good fit above the abundance threshold before trusting variant calls.

## When NOT to use

- Input is a raw FASTQ directory that has not yet been processed by inferTrueSequences, so childrenProperties.txt does not exist.
- The goal is low-abundance variant detection; HashSeq (and this diagnostic) is not sensitive for true low-abundance sequences and DADA2 is recommended instead.
- Data are long-read (e.g. PacBio) or contain frequent indels/multiple mismatches, which violate HashSeq's one-mismatch, indel-as-separate-parent assumptions.

## Inputs

- childrenProperties.txt output from HashSeq inferTrueSequences (mean and standard deviation of one-mismatch children per cluster)
- HashSeq output directory from a completed inferTrueSequences run on trimmed 250-base paired 16S rRNA FASTQ files

## Outputs

- LOESS fit diagnostic plot of children mean and standard deviation versus parent abundance, used to set or validate the filtering threshold

## How to apply

After running inferTrueSequences (with a filtering threshold such as 1,000 reads, e.g. 10,000 for the largest data sets), inspect the generated childrenProperties.txt, which contains the mean and standard deviation of one-mismatch children for each cluster. Call makeLoessPlot on the output directory and this file to plot children mean and standard deviation versus parent abundance. Examine the plot to judge whether the LOESS regression is a good fit at abundances above the chosen threshold; if not, adjust the threshold and re-run inference. This diagnostic underpins HashSeq's normal-distribution background error estimation as a function of sequencing depth, so a poor fit invalidates downstream variant inference.

## Related tools

- **HashSeq** (R package providing inferTrueSequences (to generate childrenProperties.txt) and makeLoessPlot (LOESS diagnostic visualization)) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which HashSeq is installed via devtools and the diagnostic plots are generated)

## Examples

```
library(HashSeq); inferTrueSequences(inputDir, outputDir, threshold=1000); makeLoessPlot(outputDir, fileName = "childrenProperties.txt")
```

## Evaluation signals

- The LOESS regression curve on the makeLoessPlot output visibly fits the mean and standard deviation of one-mismatch children above the chosen abundance threshold (e.g. 1,000 reads).
- childrenProperties.txt exists in the output directory with the expected cluster-level mean and standard deviation columns (README shows 230 rows × 10 columns for the zymo example).
- The chosen filtering threshold sits in the abundance region where the LOESS fit is good; if the plot shows poor fit near the threshold, the threshold is raised and inference re-run.
- Downstream SvTable.txt variant counts remain consistent with the conservative expectation that HashSeq calls fewer variants than DADA2 on mock communities such as PRJEB24409.

## Limitations

- The default Java heap size invoked by R is 512MB, which may be insufficient for big data sets; options(java.parameters="-Xmx1024m") must be set after restarting R and before attaching libraries.
- The diagnostic only validates the one-mismatch normal/LOESS error model; HashSeq does not consider two or more mismatches and does not model indels explicitly.
- A poor LOESS fit at low parent abundances means the threshold must be adjusted; at default thresholds true low-abundance sequences will not be detected.

## Evidence

- [readme] We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents.: "We can visualize the relationship between the mean and standard deviation of one-mismatch children and the abundances of their parents."
- [methods] makeLoessPlot on childrenProperties.txt: "makeLoessPlot(outputDir,fileName = "childrenProperties.txt")"
- [readme] Filtering threshold rationale: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data."
- [abstract] Normal distribution plus LOESS error estimation: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth for individual clusters of"
- [discussion] Low-abundance sensitivity limitation: "First, our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2, to detect low-abundance sequence variants."
