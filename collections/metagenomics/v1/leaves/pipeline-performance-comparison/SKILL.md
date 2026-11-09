---
name: pipeline-performance-comparison
description: Use when you need to empirically compare wall-clock runtime and peak
  memory of two amplicon sequence-variant pipelines (e.g., HashSeq vs DADA2) on the
  same 16S rRNA gene FASTQ data sets.
when_to_use_negative:
- The goal is variant-calling sensitivity on true low-abundance sequences — HashSeq
  is explicitly not sensitive there and DADA2 is recommended instead
- Input data are long reads (e.g., PacBio), where HashSeq's one-mismatch rarity assumption
  may not hold and its performance advantage is uncharacterized
- The inputs differ between pipelines (different samples, trimming, or primer handling),
  making runtime/memory comparisons invalid
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: HashSeq
  role: Pipeline under test; run inferTrueSequences with threshold=1000 on the zymo
    FASTQ files while profiling time and memory
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Comparison baseline; run its default pipeline (filterAndTrim, learnErrors,
    dada, mergePairs, makeSequenceTable) on identical inputs
- name: R
  role: Execution environment; used for devtools installation, system.time timing,
    and memory profiling of both pipelines
  repo: https://github.com/FarnazFouladi/HashSeq
- name: devtools
  role: Installs the HashSeq package from GitHub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:caf93a54265411644989bf548b15c627e8a80073272f13b6bf240602016abf01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FarnazFouladi/HashSeq
  tools:
  - DADA2
  - HashSeq
  - R
  - devtools
  - rJava
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pipeline-performance-comparison

## Summary

Benchmarks the runtime and peak memory of the HashSeq R pipeline against the DADA2 default pipeline on identical pre-processed 16S rRNA gene FASTQ inputs, demonstrating ~43x average speedup and ~3x memory reduction for HashSeq.

## When to use

Use when you need to empirically compare wall-clock runtime and peak memory of two amplicon sequence-variant pipelines (e.g., HashSeq vs DADA2) on the same 16S rRNA gene FASTQ data sets.

## When NOT to use

- The goal is variant-calling sensitivity on true low-abundance sequences — HashSeq is explicitly not sensitive there and DADA2 is recommended instead
- Input data are long reads (e.g., PacBio), where HashSeq's one-mismatch rarity assumption may not hold and its performance advantage is uncharacterized
- The inputs differ between pipelines (different samples, trimming, or primer handling), making runtime/memory comparisons invalid

## Inputs

- Pre-processed paired 16S rRNA gene FASTQ files (zymo mock community, 3 samples, primers removed, trimmed to 250 bases) from github.com/FarnazFouladi/Examples
- HashSeq R package installed via devtools::install_github("FarnazFouladi/HashSeq")
- DADA2 R package (default pipeline functions)

## Outputs

- benchmark_results.csv compiling wall-clock runtime and peak memory for HashSeq and DADA2 on the same inputs
- HashSeq output directory files (OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt)
- Timing/memory profiling records (e.g., system.time output, profmem or /usr/bin/time -v logs)

## How to apply

Download the pre-processed zymo mock community FASTQ files (3 samples from NCBI project PRJEB24409, primers removed, reads paired and trimmed to 250 bases) from github.com/FarnazFouladi/Examples into an input directory. Run HashSeq via library(HashSeq) with inferTrueSequences(inputDir, outputDir, threshold=1000), wrapping the call with system.time for wall-clock time and peak-memory profiling (e.g., profmem or /usr/bin/time -v); note the JVM default heap is 512MB, so options(java.parameters="-Xmx1024m") must be set after restarting R and before attaching libraries if more memory is needed. Run the DADA2 default pipeline (filterAndTrim, learnErrors, dada, mergePairs, makeSequenceTable) on the same FASTQ files under identical timing and memory recording. Compile both pipelines' elapsed times and peak memory into a single table (e.g., benchmark_results.csv) and check that HashSeq values are lower, consistent with the reported ~43x average speedup and ~3x memory reduction.

## Related tools

- **HashSeq** (Pipeline under test; run inferTrueSequences with threshold=1000 on the zymo FASTQ files while profiling time and memory) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Comparison baseline; run its default pipeline (filterAndTrim, learnErrors, dada, mergePairs, makeSequenceTable) on identical inputs)
- **R** (Execution environment; used for devtools installation, system.time timing, and memory profiling of both pipelines) — https://github.com/FarnazFouladi/HashSeq
- **devtools** (Installs the HashSeq package from GitHub)

## Examples

```
options(java.parameters="-Xmx1024m"); library(HashSeq); system.time(inferTrueSequences(inputDir, outputDir, threshold=1000))
```

## Evaluation signals

- Recorded HashSeq wall-clock time is lower than DADA2's on the same FASTQ inputs, consistent in direction with the reported ~43x average speedup
- Recorded HashSeq peak memory is lower than DADA2's, consistent with the reported ~3x memory reduction
- HashSeq run completes without JVM heap errors, confirming the 512MB default heap was sufficient or options(java.parameters) was set before attaching libraries
- HashSeq output directory contains all five expected files (OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt)
- Both pipelines were run on byte-identical input FASTQ files (same trimming to 250 bases and primer removal), so the comparison is apples-to-apples

## Limitations

- The JVM default heap size invoked by R is 512MB, which may be insufficient for big data sets; options(java.parameters) must be set after restarting R and before attaching any libraries
- Benchmark conclusions are specific to paired, primer-removed, 250-base-trimmed Illumina 16S rRNA gene data (zymo mock communities) and may not transfer to other read lengths or technologies
- HashSeq is not sensitive for detecting true low-abundance sequences; performance advantage should not be read as a sensitivity advantage over DADA2

## Evidence

- [results] Averaged across the data sets, HashSeq is 43 times faster than DADA2 and uses 3 times less memory.: [evidence span withheld — anchor status: not_found]
- [readme] JVM default heap is 512MB and must be increased via options(java.parameters) before attaching libraries.: "The default heap memory size for the java virtual machine invoked by R is 512MB. This may be not enough for a big data set."
- [readme] Pre-processed samples are primer-removed, paired, and trimmed to 250 bases.: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases."
- [readme] Threshold of 1000 removes sequence variants with fewer than 1000 total reads across all samples.: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
- [readme] options(java.parameters) must be set after restarting R and before attaching libraries.: "we need to increase memory using the options(java.parameters). This step must be done after restarting the R session and before attaching any libraries."
