---
name: metagenomic-read-counting-per-taxon
description: Use when you already have per-read taxonomic assignments from a MetaCache/MetaCacheSpark
  query run and need organism-level read counts and abundance percentages at a chosen
  rank such as species.
when_to_use_negative:
- You only need per-read taxonomic labels from classification and not aggregated abundance
  per organism
- No taxonomic tree is available to support pruning at the requested rank and proportional
  sub-tree redistribution
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaCacheSpark
  role: Runs the query producing per-read classifications and exposes the -a/--abundance_per
    parameter that triggers abundance estimation at the requested level; output written
    to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: AFS-MetaCache
  role: C++ workstation implementation whose query step produces the per-read classifications
    that feed the abundance quantification
  repo: https://muellan.github.io/metacache/afs.html
- name: Apache Spark
  role: Big data engine underlying MetaCacheSpark, used to scale counting and aggregation
    of per-read classifications on a cluster
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:5a73495d2ed67babfecf721da2984ef3f5b93d05d42b2dfb14f0a181798f5be4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metagenomic-read-counting-per-taxon

## Summary

Estimates the abundance of organisms at a selectable taxonomic level (e.g. species) by aggregating per-read classifications produced by AFS-MetaCache/MetaCacheSpark: counts are pooled up from pruned ranks and redistributed down from higher-level taxa using sub-tree weights.

## When to use

Use when you already have per-read taxonomic assignments from a MetaCache/MetaCacheSpark query run and need organism-level read counts and abundance percentages at a chosen rank such as species.

## When NOT to use

- You only need per-read taxonomic labels from classification and not aggregated abundance per organism
- No taxonomic tree is available to support pruning at the requested rank and proportional sub-tree redistribution

## Inputs

- Per-read classification results from a MetaCacheSpark query run
- Taxonomy with parent/child relationships and sub-tree weights
- Target taxonomic rank for abundance aggregation (e.g. species)

## Outputs

- Estimated read count per organism
- Abundance percentage per organism, written to Output_File in HDFS

## How to apply

Load the classified read results from the MetaCacheSpark query run and count reads assigned to each taxon. Set the target rank via the -a/--abundance_per parameter (e.g. species), which activates the abundance estimation feature. Prune the taxonomy at the requested rank by adding the read counts of pruned taxa to their parent taxa, then redistribute reads assigned to higher-level taxa among their children proportionally to sub-tree weights. Compute abundance percentages per organism from the estimated reads and write the estimated read counts and percentages to the output file in HDFS (Output_File).

## Related tools

- **MetaCacheSpark** (Runs the query producing per-read classifications and exposes the -a/--abundance_per parameter that triggers abundance estimation at the requested level; output written to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **AFS-MetaCache** (C++ workstation implementation whose query step produces the per-read classifications that feed the abundance quantification) — https://muellan.github.io/metacache/afs.html
- **Apache Spark** (Big data engine underlying MetaCacheSpark, used to scale counting and aggregation of per-read classifications on a cluster) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a species -n 4 queryReads.fa
```

## Evaluation signals

- Output table lists every organism with its estimated read count and abundance percentage
- Abundance percentages are consistent with the estimated read counts after up- and down-redistribution across the taxonomy
- Read counts pruned at the requested rank are fully conserved in their parent taxa (no reads lost or duplicated)
- Sum of redistributed reads assigned to higher-level taxa equals the original count for those taxa

## Limitations

- Reads assigned to higher-level taxa are only estimates after proportional sub-tree redistribution; exact per-organism provenance of those reads is lost
- Abundance accuracy depends on correct classification; the paper reports superiority in false-positive rate and quantification accuracy versus CLARK, Kraken2, and Kraken2+Bracken but does not eliminate misassignment
- MetaCacheSpark supports only build and query modes, so quantification relies on parameters available in query mode (e.g. -a/--abundance_per)

## Evidence

- [intro] Selectable-rank abundance estimation: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [readme] Abundance_per parameter in query mode: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [abstract] Superior quantification accuracy versus competitors: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
