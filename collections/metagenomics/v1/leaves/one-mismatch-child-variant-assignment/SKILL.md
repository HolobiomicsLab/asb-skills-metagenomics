---
name: one-mismatch-child-variant-assignment
description: Use when processing 16S rRNA amplicon FASTQ data to form parent/one-mismatch-child
  clusters with HashSeq and you need a verified cluster table before LOESS-based error
  inference.
when_to_use_negative:
- When the input is already a variant count table or when the analysis only needs
  taxonomic assignment — this step is a de novo clustering of raw preprocessed FASTQ
  variants.
- 'When true low-abundance sequence variants are the target: HashSeq is not sensitive
  for low-abundance sequences and more sensitive algorithms such as DADA2 are recommended
  in that case.'
- When variants with two or more mismatches, indels relative to a parent, or long-read
  (e.g. PacBio) data are in scope — HashSeq considers only one-mismatch children and
  treats indel events as separate parent sequences.
tools:
- name: HashSeq
  role: Provides inferTrueSequences, which performs the HashMap-based clustering of
    one-mismatch children to parents and writes OneMismatchCluster.txt, childrenProperties.txt,
    InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt.
  repo: https://github.com/FarnazFouladi/HashSeq
- name: R / rJava
  role: Runtime environment for HashSeq; rJava invokes the JVM, so heap size must
    be raised via options(java.parameters) before attaching libraries.
- name: DADA2
  role: Alternative/more sensitive variant caller mentioned as a comparison and for
    detecting low-abundance variants outside this skill's scope.
- name: FarnazFouladi/Examples
  role: Source of the preprocessed zymo mock-community FASTQ files used as the benchmark
    input.
  repo: https://github.com/FarnazFouladi/Examples
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:88f8540d05703f36adf0a7fda33b918fced4043c32a66f7112b887f452e96e70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - FarnazFouladi/Examples
  - HashSeq
  - R
  - R / rJava
  - rJava
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# one-mismatch-child-variant-assignment

## Summary

Assign each one-mismatch sequence variant (child) to its most abundant parent using HashSeq's fast HashMap-based clustering, then filter clusters by a parent-abundance threshold (default 1,000 reads) to build the parent/child cluster table used for downstream variant inference.

## When to use

Use when processing 16S rRNA amplicon FASTQ data to form parent/one-mismatch-child clusters with HashSeq and you need a verified cluster table before LOESS-based error inference.

## When NOT to use

- When the input is already a variant count table or when the analysis only needs taxonomic assignment — this step is a de novo clustering of raw preprocessed FASTQ variants.
- When true low-abundance sequence variants are the target: HashSeq is not sensitive for low-abundance sequences and more sensitive algorithms such as DADA2 are recommended in that case.
- When variants with two or more mismatches, indels relative to a parent, or long-read (e.g. PacBio) data are in scope — HashSeq considers only one-mismatch children and treats indel events as separate parent sequences.

## Inputs

- Preprocessed paired 16S rRNA FASTQ files (primers removed, reads paired, trimmed to 250 bases), e.g. the zymo mock community data set (NCBI project PRJEB24409)
- Parent abundance filtering threshold (default 1,000 total reads across all samples; 10,000 for the largest data set)
- Input and output directory paths; optionally a JVM heap setting (e.g. -Xmx1024m) set before loading R libraries

## Outputs

- OneMismatchCluster.txt — all parents and one-mismatch children for all clusters
- childrenProperties.txt — mean and standard deviation for each cluster
- InferenceTestSignificantChildren.txt — one-mismatch children significant in the inference test
- sequences.fasta — parents above threshold plus significant one-mismatch children
- SvTable.txt — count table of all retained sequences with per-sample counts (reference dimensions for zymo: cluster table 5643 rows x 10 columns)

## How to apply

Preprocess paired Illumina reads (primer removal, forward/reverse pairing, trim to 250 bases), then download the zymo mock-community FASTQs from FarnazFouladi/Examples into an input directory. Before attaching libraries, optionally enlarge the Java heap with options(java.parameters="-Xmx1024m") because the JVM default of 512MB may be insufficient. Run HashSeq's inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000): it sorts variants by abundance and clusters each one-mismatch child with its most abundant parent via the HashMap approach, removing variants with total reads below 1,000 across all samples (10,000 for the largest data set). Validate the run by checking that the printed cluster-table dimensions match the reference (5,643 rows x 10 columns for the zymo data set; the smallest MMC data set benchmark is 2,002 parent-plus-child clusters) and that SvTable.txt contains parent sequences plus significant one-mismatch children with per-sample counts.

## Related tools

- **HashSeq** (Provides inferTrueSequences, which performs the HashMap-based clustering of one-mismatch children to parents and writes OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt.) — https://github.com/FarnazFouladi/HashSeq
- **R / rJava** (Runtime environment for HashSeq; rJava invokes the JVM, so heap size must be raised via options(java.parameters) before attaching libraries.)
- **DADA2** (Alternative/more sensitive variant caller mentioned as a comparison and for detecting low-abundance variants outside this skill's scope.)
- **FarnazFouladi/Examples** (Source of the preprocessed zymo mock-community FASTQ files used as the benchmark input.) — https://github.com/FarnazFouladi/Examples

## Examples

```
options(java.parameters="-Xmx1024m"); library(HashSeq); threshold=1000; inferTrueSequences(inputDir, outputDir, threshold)
```

## Evaluation signals

- Printed cluster-table dimensions from inferTrueSequences equal the reference 5,643 rows x 10 columns for the zymo data set.
- The benchmark count of 2,002 parent-plus-child clusters is reproduced for the smallest MMC (mock community) data set.
- SvTable.txt contains parent sequences plus significant one-mismatch children with per-sample counts, and OneMismatchCluster.txt lists every parent and its one-mismatch children per cluster.
- The abundance filter removed all variants with total reads below the threshold (1,000 by default; 10,000 for the largest data set), and the Abundance Parents Table reflects surviving parents (reference: 9 rows x 2 columns for zymo).
- Downstream makeLoessPlot on childrenProperties.txt renders the mean/standard-deviation vs parent-abundance relationship used to check where the LOESS fit is valid.

## Limitations

- The algorithm considers only one-mismatch variants and does not consider two or more mismatches.
- The algorithm does not explicitly model insertions or deletions (indels) and treats indel events as separate parent sequences; indel variation requires methods like Deblur with multiple-sequence alignments.
- The assumption that multiple-mismatch sequences are rare may not hold for long-read technologies such as PacBio.
- Not sensitive for detecting true low-abundance sequences; DADA2 is recommended for low-abundance variants.
- The default JVM heap of 512MB invoked by R may be insufficient for big data sets; increase via options(java.parameters) after restarting R and before attaching libraries.

## Evidence

- [readme] HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference.: "HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants
with one nucleotide difference."
- [methods] Threshold removal of low-abundance variants before clustering.: "sequence variants that have total reads less than 1000 across all samples are removed"
- [discussion] Parent abundance threshold varies by data set size.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [readme] Cluster output file contents.: "OneMismatchCluster.txt: This file contains all parents and one-mismatch
children for all clusters."
- [readme] Reference cluster-table dimensions printed by the run.: ""number of rows: 5643"
#> [1] "number of columns: 10""
- [methods] Memory precondition for large data sets.: "we need to increase memory using the options(java.parameters)"
