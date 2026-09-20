---
name: one-mismatch-children-statistics
description: Use when running HashSeq on Illumina 16S rRNA amplicon data and you need
  to check that the LOESS regression of one-mismatch children's mean/SD versus parent
  abundance fits well above the filtering threshold before trusting inferred variants.
when_to_use_negative:
- Your input is already a feature/count table (e.g. SvTable.txt) rather than childrenProperties.txt
  from a HashSeq inferTrueSequences run — no parent-child cluster statistics exist
  to fit
- You need to detect true low-abundance sequence variants or indel variation — HashSeq's
  threshold-based approach and one-mismatch-only model are inappropriate; DADA2 or
  Deblur are recommended instead
- Sequences come from long-read technologies such as PacBio, where the assumption
  that multiple-mismatch sequences are rare may not hold
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: R package providing makeLoessPlot to fit and visualize the LOESS regression
    of one-mismatch children's mean/SD versus parent abundance
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment in which the HashSeq package's makeLoessPlot is executed
- name: DADA2
  role: Alternative/supplementary variant caller recommended for low-abundance variants
    that HashSeq misses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:e2f87929c5db03191714ea2f730286b845f813468370c9f002b4955ee16c8a5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# one-mismatch-children-statistics

## Summary

Model the mean and standard deviation of one-mismatch child variant abundances as a smooth LOESS function of parent sequence abundance, providing the depth-dependent background error model (normal + LOESS) that underlies HashSeq's variant inference for 16S rRNA gene data.

## When to use

Use when running HashSeq on Illumina 16S rRNA amplicon data and you need to check that the LOESS regression of one-mismatch children's mean/SD versus parent abundance fits well above the filtering threshold before trusting inferred variants.

## When NOT to use

- Your input is already a feature/count table (e.g. SvTable.txt) rather than childrenProperties.txt from a HashSeq inferTrueSequences run — no parent-child cluster statistics exist to fit
- You need to detect true low-abundance sequence variants or indel variation — HashSeq's threshold-based approach and one-mismatch-only model are inappropriate; DADA2 or Deblur are recommended instead
- Sequences come from long-read technologies such as PacBio, where the assumption that multiple-mismatch sequences are rare may not hold

## Inputs

- childrenProperties.txt from the HashSeq output directory (mean and standard deviation of one-mismatch children per cluster, with parents' abundances)
- 16S rRNA gene FASTQ files pre-processed by removing primers, pairing forward/reverse reads, and trimming to 250 bases
- Abundance threshold (default 1,000 total reads across all samples; 10,000 for the largest data set)

## Outputs

- loess_plot.png: scatterplot of one-mismatch children mean and standard deviation versus parent abundance with the LOESS fit overlaid
- Visual confirmation that the LOESS regression is a good fit above the filtering threshold, supporting the normal + LOESS background error model

## How to apply

After running HashSeq's inferTrueSequences on paired, primer-removed, 250-base-trimmed FASTQ files, read the generated childrenProperties.txt, which holds the mean and standard deviation of one-mismatch children per cluster along with their parents' abundances. Call makeLoessPlot in R to produce a scatterplot of children's mean and standard deviation versus parent abundance with a LOESS (locally estimated scatterplot smoothing) regression overlaid, rendered with scattermore for efficiency and saved as loess_plot.png. Inspect the fit restricted to parents above the abundance threshold (1,000 total reads by default; 10,000 for the largest data set), because above this threshold the LOESS regression is a good fit to the mean-SD relationship. Use the visual agreement of the LOESS curve with the mean and standard deviation relationship to justify the chosen filtering threshold and to validate the normal-plus-LOESS background error model used by inferTrueSequences to assign explicit P values to one-mismatch children.

## Related tools

- **HashSeq** (R package providing makeLoessPlot to fit and visualize the LOESS regression of one-mismatch children's mean/SD versus parent abundance) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment in which the HashSeq package's makeLoessPlot is executed)
- **DADA2** (Alternative/supplementary variant caller recommended for low-abundance variants that HashSeq misses)

## Examples

```
makeLoessPlot(outputDir, fileName = "childrenProperties.txt")
```

## Evaluation signals

- The loess_plot.png shows the LOESS fit tracking the mean-standard deviation relationship for parents above the 1,000-read threshold, consistent with Fig. 5 of the manuscript
- The relationship is well fit especially for high-abundance parents (>1,000 reads) across all six data sets, supporting the depth-dependent background error rate
- childrenProperties.txt exists in the output directory and contains mean, standard deviation, and parent abundance for each cluster
- makeLoessPlot runs without error given outputDir and fileName = 'childrenProperties.txt', producing the expected plot

## Limitations

- The LOESS fit is only a good model above the abundance threshold (1,000 reads default; 10,000 for the largest data set), so low-abundance parents are outside the validated regime
- HashSeq is conservative and not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for those
- The algorithm considers only one-mismatch variants and does not model indels, which are treated as separate parent sequences

## Evidence

- [other] The mean abundance of children in each cluster is well fit by a LOESS function of parent abundance, especially for high-abundance parents (>1,000 reads) across all six data sets: [evidence span withheld — anchor status: not_found]
- [methods] makeLoessPlot visualization of children mean and SD vs parent abundance: "makeLoessPlot(outputDir,fileName = "childrenProperties.txt")"
- [readme] README explains childrenProperties.txt and the purpose of the visualization: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data."
- [discussion] Filtering threshold default: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
