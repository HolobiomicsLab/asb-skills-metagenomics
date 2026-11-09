---
name: 16s-amplicon-variant-calling
description: Use when calling de novo sequence variants from primer-trimmed, paired
  Illumina 16S rRNA amplicon FASTQ data (e.g., mock communities like PRJEB24409),
  especially when a conservative, fast, scalable caller with explicit P values is
  preferred over DADA2 defaults.
when_to_use_negative:
- Detection of true low-abundance sequences — the article recommends more sensitive
  algorithms such as DADA2 for that purpose
- Long-read data such as PacBio, where the assumption of rarity of multiple-mismatch
  sequences does not hold
- Analyses requiring indel variation relative to a parent, or two-or-more-mismatch
  variants — use Deblur or methods with multiple-sequence alignments instead
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: HashSeq
  role: 'Primary variant caller: HashMap-based clustering of one-mismatch children
    and normal+LOESS error modeling via inferTrueSequences'
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Comparator baseline default pipeline (filterAndTrim, learnErrors, dada, mergePairs,
    makeSequenceTable) for runtime, memory, and variant-count comparison
- name: Deblur
  role: Alternative caller for capturing indel variation via multiple-sequence alignments
- name: R
  role: Execution environment; HashSeq is an R package invoked via devtools install
    and R calls
- name: rJava
  role: Java bridge requiring JVM heap sizing via options(java.parameters) before
    library attachment
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_005
  - task_006
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:55c50c7b81604de941f080da372f8e68ccf1a353a1c9580a67dd71bf472d951b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - Deblur
  - HashSeq
  - R
  - devtools
  - rJava
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# 16s-amplicon-variant-calling

## Summary

Infer true 16S rRNA gene sequence variants from Illumina amplicon reads using HashSeq, a de novo caller that clusters one-mismatch children to parents via a HashMap approach and models background error rates with a normal distribution fitted by LOESS regression, yielding explicit P values per variant.

## When to use

Use when calling de novo sequence variants from primer-trimmed, paired Illumina 16S rRNA amplicon FASTQ data (e.g., mock communities like PRJEB24409), especially when a conservative, fast, scalable caller with explicit P values is preferred over DADA2 defaults.

## When NOT to use

- Detection of true low-abundance sequences — the article recommends more sensitive algorithms such as DADA2 for that purpose
- Long-read data such as PacBio, where the assumption of rarity of multiple-mismatch sequences does not hold
- Analyses requiring indel variation relative to a parent, or two-or-more-mismatch variants — use Deblur or methods with multiple-sequence alignments instead

## Inputs

- Primer-trimmed, paired Illumina 16S rRNA gene FASTQ reads trimmed to 250 bases (e.g., PRJEB24409 mock-community zymo samples from FarnazFouladi/Examples)
- Input directory containing the FASTQ files
- Abundance threshold (default 1000 total reads across all samples; 10,000 for the largest data set)
- JVM heap setting via options(java.parameters) for large data sets

## Outputs

- OneMismatchCluster.txt (all parents and one-mismatch children per cluster; 5,643 rows x 10 columns for the MMC zymo set)
- childrenProperties.txt (mean and standard deviation per cluster; 230 rows x 10 columns)
- InferenceTestSignificantChildren.txt (significant one-mismatch children; 26 rows x 12 columns)
- sequences.fasta (parents above threshold plus significant one-mismatch children)
- SvTable.txt count table of all inferred variants for downstream taxonomic assignment

## How to apply

Start from pre-processed reads with primers removed, forward/reverse reads paired, and sequences trimmed to 250 bases; download such data (e.g., zymo.zip from the FarnazFouladi/Examples repository) into an input directory. Install the HashSeq R package with devtools::install_github("FarnazFouladi/HashSeq") (requires Java JDK), and if the 512MB default JVM heap is insufficient, set options(java.parameters="-Xmx1024m") after restarting R and before attaching libraries. Run inferTrueSequences(inputDir, outputDir, threshold=1000), which sorts variants by abundance, assigns each one-mismatch child to its most abundant parent via HashMap clustering, models one-mismatch children counts with a two-parameter normal distribution combined with LOESS regression, and removes variants with fewer than 1,000 total reads across all samples (10,000 for the largest data sets). Inspect the outputs: OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt (usable for taxonomic assignment); validate with makeLoessPlot(outputDir, fileName='childrenProperties.txt') to confirm the LOESS fit is good above the abundance threshold. For benchmarking against DADA2, run the DADA2 default pipeline (filterAndTrim, learnErrors, dada, mergePairs, makeSequenceTable) on identical inputs under matched wall-clock and peak-memory recording.

## Related tools

- **HashSeq** (Primary variant caller: HashMap-based clustering of one-mismatch children and normal+LOESS error modeling via inferTrueSequences) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Comparator baseline default pipeline (filterAndTrim, learnErrors, dada, mergePairs, makeSequenceTable) for runtime, memory, and variant-count comparison)
- **Deblur** (Alternative caller for capturing indel variation via multiple-sequence alignments)
- **R** (Execution environment; HashSeq is an R package invoked via devtools install and R calls)
- **rJava** (Java bridge requiring JVM heap sizing via options(java.parameters) before library attachment)

## Examples

```
library(HashSeq); inferTrueSequences(inputDir, outputDir, threshold=1000)
```

## Evaluation signals

- Cluster table dimensions match the reference run: 5,643 rows x 10 columns for the zymo MMC data set
- All five output files (OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt) are produced
- makeLoessPlot shows the LOESS regression is a good fit above the abundance threshold
- HashSeq wall-clock time and peak memory are lower than DADA2's on identical inputs (reported ~43x faster, ~3x less memory on average)
- BLAST against SILVA132 shows the great majority (>99%) of inferred variants with >99% percent identity to their top hit

## Limitations

- Not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for low-abundance variants
- Only one-mismatch variants are considered; two or more mismatches are not modeled
- Does not explicitly model indels; indel events are treated as separate parent sequences
- Does not address compositionality or how compositional artifacts might impact variant calling
- The rarity-of-multiple-mismatch assumption may not hold for long-read technologies such as PacBio

## Evidence

- [readme] HashMap-based clustering sorts variants by abundance and assigns one-mismatch children to parents: "HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference."
- [abstract] Normal + LOESS error modeling: "use the normal distribution combined with locally estimated scatterplot smoothing (LOESS) regression to estimate background error rates as a function of sequencing depth"
- [readme] Abundance threshold parameter: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [readme] JVM heap adjustment prerequisite: "This step must be done after restarting the R session and before attaching any libraries."
- [readme] Reference cluster-table dimensions for the zymo set: ""number of rows: 5643""
- [discussion] Conservative calling relative to DADA2: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
- [discussion] Low-abundance sensitivity limitation: "our algorithm is not sensitive to detect true low-abundance sequences. Therefore, we recommend using more sensitive algorithms, such as DADA2"
