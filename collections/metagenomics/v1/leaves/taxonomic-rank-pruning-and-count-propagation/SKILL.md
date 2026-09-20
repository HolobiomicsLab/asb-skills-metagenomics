---
name: taxonomic-rank-pruning-and-count-propagation
description: Use when you have per-read taxonomic assignments from a MetaCacheSpark/AFS-MetaCache
  query run and need read counts and abundance percentages per organism at a chosen
  rank via the -a/--abundance_per parameter.
when_to_use_negative:
- Input is raw FASTA/FASTQ reads that have not yet been classified by a MetaCache/MetaCacheSpark
  query — this skill operates on per-read classifications, not on sequences
- You only need a ranked list of candidate taxa per read (classification), not sample-level
  abundance quantification
- The analysis does not require a selectable taxonomic level and raw per-taxon read
  counts from the classifier suffice
edam_operation: http://edamontology.org/operation_3234
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3307
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation whose abundance estimation module prunes taxa
    at the requested rank, propagates counts to parents, and redistributes higher-level
    reads by sub-tree weights
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark implementation exposing the abundance estimation feature via the -a/--abundance_per
    QUERY option; writes estimated reads and abundance percentages to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Execution engine used to run the MetaCacheSpark query/quantification job on
    a big data cluster with HDFS storage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:72c75801ccc0558f07c74130aaa8af30bcc001a0d07c9e207d707d995106e741
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3234
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomic-rank-pruning-and-count-propagation

## Summary

Estimates organism abundances at a selectable taxonomic rank (e.g. species) from per-read classifications by pruning the taxonomy below the requested rank, propagating read counts upward to parent taxa, and redistributing reads from higher-level taxa to children proportionally to sub-tree weights.

## When to use

Use when you have per-read taxonomic assignments from a MetaCacheSpark/AFS-MetaCache query run and need read counts and abundance percentages per organism at a chosen rank via the -a/--abundance_per parameter.

## When NOT to use

- Input is raw FASTA/FASTQ reads that have not yet been classified by a MetaCache/MetaCacheSpark query — this skill operates on per-read classifications, not on sequences
- You only need a ranked list of candidate taxa per read (classification), not sample-level abundance quantification
- The analysis does not require a selectable taxonomic level and raw per-taxon read counts from the classifier suffice

## Inputs

- Classified per-read results from a MetaCacheSpark query run (classification maps of reads to taxa)
- Taxonomy tree covering the reference database organisms
- Target taxonomic rank for -a/--abundance_per (e.g. species)

## Outputs

- Per-organism table of estimated read counts at the requested rank
- Per-organism abundance percentages
- Output file written to HDFS (Output_File)

## How to apply

Load the classified read results from the MetaCacheSpark query run (stored in HDFS) and count reads assigned to each taxon. Prune the taxonomy at the rank selected with the -a/--abundance_per QUERY option (e.g. species), adding the read counts of pruned taxa to their parent taxa so no reads are lost. Then redistribute reads assigned to higher-level taxa among their children proportionally to sub-tree weights, which recovers rank-specific estimates for reads classified only at a coarser level. Compute abundance percentages per organism from the estimated read counts and write the table of estimated reads and abundance percentages to the Output_File in HDFS. This abundance estimation is a MetaCache extension introduced for all-food-sequencing, where food samples can span various kingdoms of life and a fixed species-level tally is insufficient.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation whose abundance estimation module prunes taxa at the requested rank, propagates counts to parents, and redistributes higher-level reads by sub-tree weights) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark implementation exposing the abundance estimation feature via the -a/--abundance_per QUERY option; writes estimated reads and abundance percentages to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Execution engine used to run the MetaCacheSpark query/quantification job on a big data cluster with HDFS storage)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a species -q threshold -n 4 -p 16 myReads.fa
```

## Evaluation signals

- The output table lists every organism at the requested rank with both an estimated read count and an abundance percentage
- Total read count is conserved: pruning adds pruned taxa's counts to parents and redistribution splits higher-level counts among children, so no reads are dropped or duplicated
- Abundance percentages are consistent with the estimated read counts after redistribution (percentages sum to ~100% across organisms at the selected rank)
- The requested -a/--abundance_per rank is honored: no taxa below the selected level (e.g. subspecies-level entries when rank=species) appear in the output
- Redistribution of higher-level taxa's reads follows sub-tree weights, so children within one clade receive shares proportional to their sub-tree read masses

## Limitations

- Reads assigned to higher-level taxa are only redistributed proportionally to sub-tree weights — an approximation that can misattribute reads within clades rather than resolving them taxonomically
- Quantification accuracy and false-positive rate should be benchmarked against alternatives (CLARK, Kraken2, Kraken2+Bracken were used in the paper); results depend on reference database completeness for the sample's kingdoms
- MetaCacheSpark output targets HDFS/Hadoop clusters and requires the same database variant used at build time; the C++ and Spark versions differ in runtime environment

## Evidence

- [intro] Selectable abundance level: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references"
- [abstract] Quantification superiority: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
- [readme] abundance_per parameter: " -a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [intro] Food ingredient detection scope: "detection and quantification of ingredients in food samples that can span various kingdoms"
