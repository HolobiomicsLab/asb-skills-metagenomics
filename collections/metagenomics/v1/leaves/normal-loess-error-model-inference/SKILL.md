---
name: normal-loess-error-model-inference
description: Use when comparing the number of sequence variants called from 16S rRNA
  amplicon FASTQ data by HashSeq's normal/LOESS error model at 5% FDR versus DADA2
  defaults, especially on mock communities of known taxa.
when_to_use_negative:
- The goal is to detect true low-abundance sequence variants — HashSeq is not sensitive
  there; the paper recommends more sensitive algorithms such as DADA2 for low-abundance
  variants.
- Input reads require modeling of indels or two-or-more-mismatch differences — HashSeq
  only considers one-mismatch variants and treats indel events as separate parent
  sequences.
- Data come from long-read technologies such as PacBio, where the assumption of rarity
  of multiple-mismatch sequences may not hold.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: HashSeq
  role: R package that performs the HashMap-based clustering, normal/LOESS error modeling,
    and 5% FDR variant inference via inferTrueSequences and makeLoessPlot
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Comparison baseline run with default parameters on the same trimmed 250-base
    reads to obtain competing sequence-variant counts
- name: R
  role: Environment for running HashSeq (installed with devtools::install_github)
    and the comparison/tabulation scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:c67a3d1ace6afe0917a917e061a43173479fd17e689b7a7bc9c9cd9afddd398c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# normal-loess-error-model-inference

## Summary

Reproduces HashSeq's variant calling, in which one-mismatch children of parent 16S rRNA sequences are tested against a normally distributed background error estimated with LOESS regression as a function of parent abundance, and compares the resulting variant counts to DADA2 defaults on mock-community (zymo) FASTQ data.

## When to use

Use when comparing the number of sequence variants called from 16S rRNA amplicon FASTQ data by HashSeq's normal/LOESS error model at 5% FDR versus DADA2 defaults, especially on mock communities of known taxa.

## When NOT to use

- The goal is to detect true low-abundance sequence variants — HashSeq is not sensitive there; the paper recommends more sensitive algorithms such as DADA2 for low-abundance variants.
- Input reads require modeling of indels or two-or-more-mismatch differences — HashSeq only considers one-mismatch variants and treats indel events as separate parent sequences.
- Data come from long-read technologies such as PacBio, where the assumption of rarity of multiple-mismatch sequences may not hold.

## Inputs

- Pre-processed mock-community 16S rRNA FASTQ files (zymo.zip from github.com/FarnazFouladi/Examples; primers removed, paired, trimmed to 250 bases)
- Abundance threshold (default 1,000 reads total across all samples)
- FDR level (0.05) for Benjamini-Hochberg-corrected inference tests

## Outputs

- SvTable.txt sequence-variant count table from HashSeq
- childrenProperties.txt with per-cluster mean and standard deviation of one-mismatch children
- OneMismatchCluster.txt, InferenceTestSignificantChildren.txt, and sequences.fasta from inferTrueSequences
- variant_counts_table.csv comparing HashSeq vs DADA2 variant counts per mock community

## How to apply

Download the pre-processed zymo mock-community FASTQ files (project PRJEB24409; primers removed, reads paired and trimmed to 250 bases) from github.com/FarnazFouladi/Examples and unzip them into an input directory. Run HashSeq's inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000, fdr=0.05): it clusters sequences one nucleotide apart with a HashMap approach, models the mean and standard deviation of one-mismatch children as a function of parent abundance using a normal distribution plus LOESS regression, and removes variants with fewer than 1,000 total reads across samples (10,000 for the largest data set). Verify the LOESS fit with makeLoessPlot on childrenProperties.txt before trusting the inference. Independently run the DADA2 default pipeline on the same trimmed 250-base reads, then tabulate variant counts per mock community and confirm the expected direction: HashSeq is more conservative than DADA2 and calls fewer variants, while its parents still recover the eight bacterial taxa in the mock.

## Related tools

- **HashSeq** (R package that performs the HashMap-based clustering, normal/LOESS error modeling, and 5% FDR variant inference via inferTrueSequences and makeLoessPlot) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Comparison baseline run with default parameters on the same trimmed 250-base reads to obtain competing sequence-variant counts)
- **R** (Environment for running HashSeq (installed with devtools::install_github) and the comparison/tabulation scripts)

## Examples

```
inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000, fdr=0.05); makeLoessPlot(outputDir, fileName="childrenProperties.txt")
```

## Evaluation signals

- The LOESS plot of children mean and standard deviation vs parent abundance (from makeLoessPlot on childrenProperties.txt) shows a good fit above the abundance threshold.
- The comparison table reproduces the reported direction: HashSeq calls a lower number of sequence variants as true than DADA2 on the mock communities.
- Parent sequences in the HashSeq output include the eight bacterial taxa present in the zymo mock community.
- inferTrueSequences output dimensions are consistent with the README example (e.g. abundance parents table with 9 rows, significant children table with 26 rows for the zymo data set).

## Limitations

- HashSeq is not sensitive for detecting true low-abundance sequences; use DADA2 for low-abundance variants.
- Only one-mismatch variants are considered; two or more mismatches are ignored, and indels are not explicitly modeled (treated as separate parent sequences).
- The assumption of rarity of multiple-mismatch sequences may fail for long-read technologies such as PacBio; compositionality artifacts are not addressed.
- The default Java heap of 512MB may be insufficient for large data sets; increase it with options(java.parameters) after restarting R and before attaching libraries.

## Evidence

- [discussion] Conservativeness direction: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
- [abstract] Error model: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth"
- [methods] Mock data set: "three mock communities of 16S rRNA gene sequences that are publicly available at NCBI under project# PRJEB24409"
- [discussion] Threshold: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [readme] LOESS plot usage: "This visualization helps us to set a filtering threshold at above which the LOESS regression is a good fit for our data."
- [readme] Java heap caveat: "The default heap memory size for the java virtual machine invoked by R is 512MB. This may be not enough for a big data set."
