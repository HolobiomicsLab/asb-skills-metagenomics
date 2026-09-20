---
name: 16s-amplicon-sequence-clustering
description: Use when Illumina MiSeq 16S rRNA amplicon FASTQ data (primer-removed,
  paired, trimmed to 250 bases) must be de novo clustered into true sequence variants
  with one-mismatch children assessed against sequencing error.
when_to_use_negative:
- Goal is detection of true low-abundance sequence variants — HashSeq is not sensitive
  there; the article recommends more sensitive algorithms such as DADA2
- Variants involve insertions/deletions or indel variation relative to a parent —
  HashSeq considers only one-mismatch variants and treats indel events as separate
  parent sequences; the article suggests Deblur or other multiple-sequence-alignment
  methods instead
- Data come from long-read technologies such as PacBio, where the assumption of rarity
  of multiple-mismatch sequences may not hold
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: HashSeq
  role: R package providing inferTrueSequences (HashMap-based one-mismatch clustering,
    normal + LOESS background error estimation) and makeLoessPlot for threshold diagnostics
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R
  role: Runtime environment; installed via devtools::install_github, requires Java
    JDK and options(java.parameters) for heap adjustment
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Alternative/error-model comparator — HashSeq is more conservative and calls
    fewer sequence variants as true; recommended instead for low-abundance variants
- name: Deblur
  role: Alternative method suggested for capturing indel variation via multiple-sequence
    alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:c61f95334768a0366af2232bd9c8a8c47f2af3ad5f908e95e25732cabf26db23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - Deblur
  - HashSeq
  - R
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# 16s-amplicon-sequence-clustering

## Summary

Cluster pre-processed 16S rRNA gene FASTQ reads into parent sequences and one-mismatch children using HashSeq's HashMap-based de novo variant caller, then infer true biological sequences against a normally distributed background error model estimated by LOESS regression.

## When to use

Use when Illumina MiSeq 16S rRNA amplicon FASTQ data (primer-removed, paired, trimmed to 250 bases) must be de novo clustered into true sequence variants with one-mismatch children assessed against sequencing error.

## When NOT to use

- Goal is detection of true low-abundance sequence variants — HashSeq is not sensitive there; the article recommends more sensitive algorithms such as DADA2
- Variants involve insertions/deletions or indel variation relative to a parent — HashSeq considers only one-mismatch variants and treats indel events as separate parent sequences; the article suggests Deblur or other multiple-sequence-alignment methods instead
- Data come from long-read technologies such as PacBio, where the assumption of rarity of multiple-mismatch sequences may not hold

## Inputs

- Pre-processed 16S rRNA gene FASTQ files (primers removed, forward/reverse reads paired, trimmed to 250 bases), e.g. zymo mock communities from NCBI PRJEB24409 via github.com/FarnazFouladi/Examples
- Input directory of FASTQ files (inputDir) and output directory (outputDir)
- Abundance threshold (default 1000 total reads across all samples; 10000 for the largest data sets)

## Outputs

- OneMismatchCluster.txt (all parents and one-mismatch children for all clusters)
- childrenProperties.txt (mean and standard deviation of one-mismatch children per cluster)
- InferenceTestSignificantChildren.txt (one-mismatch children significant in the inference test)
- sequences.fasta (parents above threshold plus significant children)
- SvTable.txt (count table of inferred sequence variants for downstream taxonomic assignment)
- LOESS diagnostic plot from makeLoessPlot

## How to apply

First pre-process reads by removing primers, pairing forward and reverse reads, and trimming to 250 bases, as done for the zymo mock communities from NCBI project PRJEB24409. Install HashSeq from GitHub with devtools::install_github("FarnazFouladi/HashSeq") (Java JDK required; increase heap via options(java.parameters="-Xmx1024m") before loading libraries if the 512MB default is too small). Run inferTrueSequences(inputDir, outputDir, threshold) with an abundance threshold of 1,000 total reads (10,000 for the largest data sets) so variants below the threshold are removed. The function clusters sequences differing by one nucleotide with a fast HashMap approach, estimates each cluster's background error rate as a function of sequencing depth using a normal distribution combined with LOESS regression, and performs an inference test to call significant one-mismatch children as true variants. Use makeLoessPlot(outputDir, fileName="childrenProperties.txt") to visualize cluster mean/standard deviation versus parent abundance and confirm the threshold sits where the LOESS fit is good. HashSeq is conservative relative to DADA2, calling fewer sequence variants as true and providing explicit P values per variant.

## Related tools

- **HashSeq** (R package providing inferTrueSequences (HashMap-based one-mismatch clustering, normal + LOESS background error estimation) and makeLoessPlot for threshold diagnostics) — https://github.com/FarnazFouladi/HashSeq
- **R** (Runtime environment; installed via devtools::install_github, requires Java JDK and options(java.parameters) for heap adjustment) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Alternative/error-model comparator — HashSeq is more conservative and calls fewer sequence variants as true; recommended instead for low-abundance variants)
- **Deblur** (Alternative method suggested for capturing indel variation via multiple-sequence alignments)

## Examples

```
inferTrueSequences(inputDir,outputDir,threshold)  # e.g. inputDir=file.path(seqDir,"zymo"), threshold=1000
```

## Evaluation signals

- inferTrueSequences runs to completion printing cluster-table dimensions (e.g. zymo example: 5643-row cluster table, 9 parents, 230 children-properties rows, 26 significant children)
- All five expected output files are produced (OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt)
- SvTable.txt contains only sequences with total reads at or above the chosen threshold (e.g. 1000) across all samples
- makeLoessPlot shows the LOESS regression fits the mean/standard-deviation-vs-parent-abundance relationship above the chosen threshold
- Detected variants match well against SILVA reference variants in the article's validation, and HashSeq calls are conservative (fewer variants than DADA2)

## Limitations

- Not sensitive for true low-abundance sequences; DADA2 recommended for those
- Considers only one-mismatch variants; two or more mismatches are not considered
- Does not explicitly model insertions or deletions (indels); indel events are treated as a separate parent sequence
- Assumption of rarity of multiple-mismatch sequences may not hold for long-read technologies such as PacBio
- Does not address compositionality or compositional artifacts affecting variant calling
- Default 512MB Java heap may be insufficient for large data sets; increase via options(java.parameters) after restarting R and before attaching libraries
- A Poisson model of one-mismatch children failed due to overdispersion (mean not equal to variance), motivating the normal-distribution-plus-LOESS approach; the negative binomial was rejected because it is undefined when variance is less than the mean

## Evidence

- [readme] HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference. It further uses the normal distribution combined with LOESS regression to estimate background error rates as a function of sequencing depth: "HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference."
- [methods] Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases"
- [methods] sequence variants that have total reads less than 1000 across all samples are removed: "sequence variants that have total reads less than 1000 across all samples are removed"
- [discussion] This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [discussion] our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true"
- [discussion] This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database: "This conservative approach detected sequence variants that had a good match to existing variants in the SILVA database"
- [readme] Note: The default heap memory size for the java virtual machine invoked by R is 512MB.: "Note: The default heap memory size for the java virtual machine invoked by R is 512MB."
