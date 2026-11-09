---
name: metagenomic-abundance-quantification
description: Use when raw FASTA/FASTQ reads from a sample must be classified against
  a reference genome collection and species-level abundances (or false-positive/deviation
  metrics like Σ FP and Σ Dev) are the target output.
when_to_use_negative:
- The analysis goal is a full alignment-level comparison (e.g. variant calling or
  alignment inspection) rather than taxonomic detection/quantification — an aligner
  like BWA is the wrong substitute direction here.
- Input is already a classified abundance table or feature table; no read classification
  is needed.
- The sample requires de novo discovery of organisms absent from the reference database;
  the method only detects taxa present among the indexed reference genomes.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation of the minhashing k-mer classifier used to
    build the AFS reference database and classify/query reads with species-level abundance
    estimation and the remove-overpopulated-features filter.
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Apache Spark implementation used to build (16 executors, HDFS) and query (16
    executors, 4 threads/executor) the classification database on a big data cluster.
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: BWA
  role: Alignment-based baseline of the previous AFS pipeline used for comparison
    (per-genome alignment); replaced by the k-mer classifier in this skill.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:e620eccef82972292cc36419d3a4051cfe23a249e6b40b40884a3810d7a82cc6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - BWA
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metagenomic-abundance-quantification

## Summary

Detect and quantify the species composition of a biological sample (e.g. food) by classifying sequencing reads against a reference genome database with the alignment-free, minhashing-based MetaCache/AFS-MetaCache pipeline, estimating abundance at a selectable taxonomic level instead of running per-genome alignments.

## When to use

Use when raw FASTA/FASTQ reads from a sample must be classified against a reference genome collection and species-level abundances (or false-positive/deviation metrics like Σ FP and Σ Dev) are the target output.

## When NOT to use

- The analysis goal is a full alignment-level comparison (e.g. variant calling or alignment inspection) rather than taxonomic detection/quantification — an aligner like BWA is the wrong substitute direction here.
- Input is already a classified abundance table or feature table; no read classification is needed.
- The sample requires de novo discovery of organisms absent from the reference database; the method only detects taxa present among the indexed reference genomes.

## Inputs

- Sequencing reads in FASTA or FASTQ format (single-end or paired, gzipped supported via zlib)
- Reference genome collection in FASTA format (e.g. the AFS10 reference genomes)
- Taxonomy/mapping information linking reference sequences to taxa for abundance estimation levels

## Outputs

- Read-to-taxon classification results per sample
- Abundance estimates at a selectable taxonomic level (e.g. species)
- Comparison metrics such as Σ FP and Σ Dev, and timings (build time, query speed in millions reads/min)

## How to apply

Build a k-mer/minhash database from the reference genome collection (e.g. the 10-genome AFS10 set) using MetaCache or MetaCacheSpark, optionally enabling the remove-overpopulated-features (-o) filter at build time to drop features that map to too many targets. Classify/query the read sets (e.g. KAL_D calibrator sausage reads, Kal/KLyo samples) with abundance estimation enabled via the -a/--abundance_per option at the desired taxonomic level (species), and select a query mode (precise, threshold, fast, very_fast; default threshold) with -q. On a Spark cluster, run the build with 16 executors storing the database in HDFS and the classification with 16 executors and 4 threads per executor. Rationale: counting k-mer window matches after minhash subsampling avoids per-genome BWA alignment, cutting AFS10 database construction from ~7 h to ~47 m and raising query throughput from 0.04 to 17.1 million reads/min. Validate accuracy by computing the sum of false positives (Σ FP) and sum of deviations (Σ Dev) against known sample compositions, where this approach outperformed CLARK, Kraken2 and Kraken2+Bracken.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation of the minhashing k-mer classifier used to build the AFS reference database and classify/query reads with species-level abundance estimation and the remove-overpopulated-features filter.) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Apache Spark implementation used to build (16 executors, HDFS) and query (16 executors, 4 threads/executor) the classification database on a big data cluster.) — https://github.com/jmabuin/MetaCacheSpark
- **BWA** (Alignment-based baseline of the previous AFS pipeline used for comparison (per-genome alignment); replaced by the k-mer classifier in this skill.)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a species -q threshold -n 4 Kal_D-sample.fastq
```

## Evaluation signals

- Database build time should drop from ~7 h (BWA-based AFS pipeline on AFS10) to ~47 m with AFS-MetaCache, and query speed should reach on the order of 17.1 MR/m vs 0.04 MR/m.
- Σ FP (sum of false positives) and Σ Dev (sum of deviations) computed on the KAL_D/Kal/KLyo samples should match the published Tables 9-10 values.
- The -a/--abundance_per option must be set (e.g. species level) for quantification output; without it only per-read classifications are produced.
- The database must be queried with the same MetaCache variant/data-type build (k-mer size, target/window ID width) used to construct it; mismatches fail.
- In CLARK/Kraken2/Kraken2+Bracken benchmarks, AFS-MetaCache results should show the lowest false-positive rate and best quantification accuracy.

## Limitations

- No software version numbers were reported for benchmarked competitors (Kraken2, CLARK, Bracken, Spark), and no hardware specifications of the big data cluster used for benchmarks were given.
- Default MetaCache compilation supports k-mer sizes up to 16 and needs ~128 GB RAM for full NCBI RefSeq bacterial/viral/archaeal databases; use database partitioning if RAM is insufficient.
- No availability statement exists for the raw sequencing data or reference sample datasets of the admixture benchmarks, complicating direct reproduction.
- No specific versions or accession identifiers of the reference genomes used for database construction are stated in the article.

## Evidence

- [abstract] We introduce an alignment-free k-mer based method for detection and quantification of species composition in food and other complex biological matters.: "We introduce an alignment-free k-mer based method for detection and quantification of species composition in food and other complex biological matters."
- [intro] Selectable taxonomic level and coverage filtering: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [methods] Spark query execution: "Execute **MetaCacheSpark** for classification with 16 executors and 4 threads per executor."
- [methods] remove-overpopulated-features option: "* **-o**: Uses the *remove-overpopulated-features*."
- [readme] abundance option in MetaCacheSpark usage: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
