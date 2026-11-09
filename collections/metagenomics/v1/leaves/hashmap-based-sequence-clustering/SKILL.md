---
name: hashmap-based-sequence-clustering
description: Use when Illumina 16S rRNA amplicon FASTQ reads (primer-trimmed, paired,
  trimmed to 250 bases) need de novo variant calling, and one-mismatch child sequences
  must be grouped with abundant parents before error-rate modeling.
when_to_use_negative:
- Detecting true low-abundance sequence variants — HashSeq is not sensitive here;
  use more sensitive algorithms such as DADA2
- Data with two or more mismatches, indel variation, or long reads such as PacBio
  where multiple-mismatch sequences are not rare
- Input is already a feature/count table — this skill operates on raw (preprocessed)
  fastq reads
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: R package providing inferTrueSequences, which performs the HashMap-based one-mismatch
    clustering and generates the count table
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Alternative/more sensitive variant caller recommended for low-abundance variants;
    benchmark for supervised classification behavior
- name: R
  role: Runtime environment for HashSeq; installed via devtools::install_github
- name: rJava
  role: Bridges R to the Java virtual machine whose heap must be sized via options(java.parameters)
    for large data sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:9ce7c1639755deb7931b1ced64b63568243ea06b420debf7dc6d279f7df92ecb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  - rJava
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# hashmap-based-sequence-clustering

## Summary

Cluster one-mismatch 16S rRNA amplicon sequence variants onto their most abundant parent using HashSeq's fast HashMap-based approach, as a precursor to LOESS-based background error estimation and true-sequence inference.

## When to use

Use when Illumina 16S rRNA amplicon FASTQ reads (primer-trimmed, paired, trimmed to 250 bases) need de novo variant calling, and one-mismatch child sequences must be grouped with abundant parents before error-rate modeling.

## When NOT to use

- Detecting true low-abundance sequence variants — HashSeq is not sensitive here; use more sensitive algorithms such as DADA2
- Data with two or more mismatches, indel variation, or long reads such as PacBio where multiple-mismatch sequences are not rare
- Input is already a feature/count table — this skill operates on raw (preprocessed) fastq reads

## Inputs

- Primer-trimmed, paired 16S rRNA gene fastq files trimmed to 250 bases (e.g., zymo mock community, NCBI PRJEB24409)
- Abundance threshold parameter (default 1,000 total reads across all samples)
- JVM heap memory setting via options(java.parameters) for large data sets

## Outputs

- OneMismatchCluster.txt — all parents and one-mismatch children for all clusters
- childrenProperties.txt — mean and standard deviation for each cluster
- InferenceTestSignificantChildren.txt — one-mismatch children significant in the inference test
- sequences.fasta — parents above threshold plus significant one-mismatch children
- SvTable.txt — sequence-variant count table (e.g., 5,643 rows x 10 columns for the zymo data set)

## How to apply

Download preprocessed mock-community fastq files (e.g., the zymo set from NCBI project PRJEB24409 via github.com/FarnazFouladi/Examples) into an input directory. Increase the JVM heap with options(java.parameters="-Xmx1024m") before attaching libraries if the default 512MB is insufficient, then load the HashSeq R package. Run inferTrueSequences(inputDir, outputDir, threshold=1000), which sorts variants by abundance and assigns each one-mismatch child to its most abundant parent; variants with fewer than 1,000 total reads across all samples are filtered out (use 10,000 for the largest data sets). The HashMap clustering writes OneMismatchCluster.txt (parents plus one-mismatch children), childrenProperties.txt (cluster means/SDs), InferenceTestSignificantChildren.txt, sequences.fasta, and SvTable.txt; validate against the reference zymo run's cluster table of 5,643 rows x 10 columns.

## Related tools

- **HashSeq** (R package providing inferTrueSequences, which performs the HashMap-based one-mismatch clustering and generates the count table) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Alternative/more sensitive variant caller recommended for low-abundance variants; benchmark for supervised classification behavior)
- **R** (Runtime environment for HashSeq; installed via devtools::install_github)
- **rJava** (Bridges R to the Java virtual machine whose heap must be sized via options(java.parameters) for large data sets)

## Examples

```
options(java.parameters="-Xmx1024m"); library(HashSeq); inferTrueSequences(inputDir,outputDir,threshold)
```

## Evaluation signals

- The printed cluster-table dimensions for the zymo data set equal 5,643 rows x 10 columns
- SvTable.txt contains parent sequences plus significant one-mismatch children with per-sample counts and can be used for downstream taxonomic assignment
- All five expected output files (OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt) are present in the output directory
- Variants with total reads below the abundance threshold (1,000 by default) are absent from the parent table
- For the zymo run, the reference outputs include an abundance-parents table of 9 rows x 2 columns and a children-properties table of 230 rows x 10 columns

## Limitations

- HashSeq is not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for those variants
- The algorithm considers only one-mismatch variants and does not consider two or more mismatches
- The algorithm does not explicitly model insertions or deletions and treats indel events as separate parent sequences
- The rarity assumption for multiple-mismatch sequences may not hold for long-read technologies such as PacBio
- The default 512MB JVM heap may be insufficient for big data sets and must be raised via options(java.parameters) before attaching libraries

## Evidence

- [readme] HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference.: "HashSeq utilizes a very fast HashMap-based approach to cluster sequence variants with one nucleotide difference."
- [methods] Sequence variants with fewer than 1,000 total reads are removed.: "sequence variants that have total reads less than 1000 across all samples are removed"
- [discussion] Parent abundance threshold set at 1,000 reads except for the largest data set.: "This was set at a parent abundance of 1,000 reads for all of our data sets except the largest one, where it was set to 10,000."
- [readme] SvTable count table is intended for downstream taxonomic assignment.: "This count table can be used for taxonomic assignment and other down-stream analyses."
- [abstract] HashMap-based variant detection applied to six public 16S rRNA data sets.: "we utilized a very fast HashMap-based approach to detect sequence variants in six publicly available 16S rRNA gene data sets."
