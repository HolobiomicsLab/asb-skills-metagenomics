---
name: mock-community-benchmark-datasets
description: Reference for validating or benchmarking a 16S rRNA gene sequence variant
  caller on data with known composition, or for reproducing runtime, memory, and variant-count
  comparisons between HashSeq and DADA2.
when_to_use_negative:
- The goal is detecting true low-abundance sequence variants — the article recommends
  more sensitive algorithms such as DADA2 for low-abundance variants, so mock-community
  confirmation of low-abundance taxa is the wrong scope
- Input reads are raw and unprocessed — this skill's benchmark data sets were already
  primer-trimmed, paired, and trimmed to 250 bases, so applying it to untrimmed data
  breaks comparability
- Working with long-read technologies such as PacBio — HashSeq assumes multiple-mismatch
  sequences are rare, which may not hold for long reads
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: HashSeq
  role: Variant caller being benchmarked; run via inferTrueSequences on the mock community
    FASTQ inputs
  repo: https://github.com/FarnazFouladi/HashSeq
- name: DADA2
  role: Reference variant caller for comparing variant counts, runtime, and memory
    on the same mock community data
- name: R
  role: Environment for running HashSeq and DADA2 pipelines and for timing/memory
    profiling with system.time
- name: rJava
  role: Java bridge required by HashSeq; heap size adjusted via options(java.parameters)
    before library attachment
- name: devtools
  role: Used to install the HashSeq R package from GitHub
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00697-21
    title: 'HashSeq: a Simple, Scalable, and Conservative <i>De Novo</i> Variant Caller
      for 16S rRNA Gene Data Sets'
  source_content_hash: sha256:2628b28a6b06a50527e78054ff76f3888317312309d34eb2432421ed16dba568
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
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mock-community-benchmark-datasets

## Summary

Use pre-processed 16S rRNA gene mock community FASTQ data sets (e.g., the three zymo communities from NCBI PRJEB24409) as controlled benchmarks for validating variant callers like HashSeq against DADA2, including accuracy of taxon recovery and runtime/memory performance.

## When to use

Reference for validating or benchmarking a 16S rRNA gene sequence variant caller on data with known composition, or for reproducing runtime, memory, and variant-count comparisons between HashSeq and DADA2.

## When NOT to use

- The goal is detecting true low-abundance sequence variants — the article recommends more sensitive algorithms such as DADA2 for low-abundance variants, so mock-community confirmation of low-abundance taxa is the wrong scope
- Input reads are raw and unprocessed — this skill's benchmark data sets were already primer-trimmed, paired, and trimmed to 250 bases, so applying it to untrimmed data breaks comparability
- Working with long-read technologies such as PacBio — HashSeq assumes multiple-mismatch sequences are rare, which may not hold for long reads

## Inputs

- Pre-processed zymo mock community FASTQ files (3 mock community samples, primers removed, paired reads trimmed to 250 bases) from FarnazFouladi/Examples zymo.zip
- NCBI PRJEB24409 mock community 16S rRNA gene sequence data

## Outputs

- HashSeq outputs: OneMismatchCluster.txt, childrenProperties.txt, InferenceTestSignificantChildren.txt, sequences.fasta, SvTable.txt
- benchmark_results.csv comparing wall-clock runtime and peak memory of HashSeq vs DADA2
- variant_counts_table.csv comparing sequence variant counts per mock community between methods

## How to apply

Download the pre-processed zymo mock community FASTQ files from github.com/FarnazFouladi/Examples (zymo.zip); these three mock communities come from NCBI project PRJEB24409 and were pre-processed by removing primers, pairing forward and reverse reads, and trimming sequences to 250 bases. Unzip them into a local input directory and run HashSeq's inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000) — set options(java.parameters) before attaching libraries if the 512MB default JVM heap is insufficient. For accuracy benchmarks, confirm that HashSeq parent sequences include the eight bacterial taxa in the mock and compare variant counts against the DADA2 default pipeline (filterAndTrim, learnErrors, dada, mergePairs, makeSequenceTable). For performance benchmarks, wrap both pipelines with R timing (system.time) and peak-memory profiling, then tabulate results; the reference finding is a ~43x average speedup and ~3x memory reduction for HashSeq, and the zymo (MMC) data set yields 2,002 parent/child clusters and a 5,643 x 10 cluster table.

## Related tools

- **HashSeq** (Variant caller being benchmarked; run via inferTrueSequences on the mock community FASTQ inputs) — https://github.com/FarnazFouladi/HashSeq
- **DADA2** (Reference variant caller for comparing variant counts, runtime, and memory on the same mock community data)
- **R** (Environment for running HashSeq and DADA2 pipelines and for timing/memory profiling with system.time)
- **rJava** (Java bridge required by HashSeq; heap size adjusted via options(java.parameters) before library attachment)
- **devtools** (Used to install the HashSeq R package from GitHub)

## Examples

```
options(java.parameters="-Xmx1024m"); library(HashSeq); inferTrueSequences(inputDir, outputDir, abundanceThreshold=1000)
```

## Evaluation signals

- The zymo (MMC) data set produces a cluster table with exactly 5,643 rows and 10 columns, and 2,002 parent/one-mismatch-child clusters
- Recorded HashSeq wall-clock time and peak memory are lower than DADA2's, consistent with the reported ~43x average speedup and ~3x memory reduction
- HashSeq parent sequences on the MMC data set include the eight bacterial taxa present in the mock community
- HashSeq calls fewer sequence variants than DADA2 on the mock communities (conservative direction)
- makeLoessPlot on childrenProperties.txt shows the LOESS regression fits well above the abundance threshold of 1,000 reads

## Limitations

- HashSeq is not sensitive for detecting true low-abundance sequences; more sensitive algorithms such as DADA2 are recommended for low-abundance variants
- The algorithm considers only one-mismatch variants and does not consider two or more mismatches, does not explicitly model indels (treating them as separate parents), and does not address compositionality
- The default 512MB JVM heap may be insufficient for big data sets; options(java.parameters) must be set after restarting R and before attaching any libraries

## Evidence

- [methods] three mock communities of 16S rRNA gene sequences that are publicly available at NCBI under project# PRJEB24409: "three mock communities of 16S rRNA gene sequences that are publicly available at NCBI under project# PRJEB24409"
- [methods] pre-processing description: "Primers from these sequences were removed, forward and reverse reads were paired and sequences were trimmed to 250 bases"
- [readme] default heap memory: "The default heap memory size for the java virtual machine invoked by R is 512MB. This may be not enough for a big data set."
- [results] runtime/memory comparison: [evidence span withheld — anchor status: not_found]
- [discussion] conservativeness: "our algorithm, compared to the popular algorithm DADA2, is fairly conservative and calls a lower number of sequence variants as true."
- [readme] abundance threshold: "Here we choose a threshold of 1000 which means sequence variants that have total reads less than 1000 across all samples are removed."
