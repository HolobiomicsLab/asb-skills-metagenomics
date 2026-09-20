---
name: proportional-read-redistribution-across-taxonomy
description: Use when per-read classifications from MetaCacheSpark/AFS-MetaCache must
  be converted into per-organism read counts and abundance percentages at a chosen
  taxonomic rank, and reads sitting on higher-level taxa need redistribution.
when_to_use_negative:
- The goal is raw per-read taxonomic labels rather than rank-level abundance estimation
  (plain MetaCacheSpark query output already provides these).
- The input is already a per-organism abundance/feature table — redistribution operates
  on per-read classifications, not aggregated counts.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaCacheSpark
  role: Produces the per-read classifications whose taxon counts feed the abundance
    estimation, and exposes the -a/--abundance_per parameter to trigger quantification
    at a chosen rank; writes results to HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: AFS-MetaCache
  role: C++ workstation variant augmented with the same selectable-level abundance
    estimation used in the quantification module
  repo: https://muellan.github.io/metacache/afs.html
- name: Apache Spark
  role: Distributed engine on which the quantification computation is executed across
    the cluster with results stored in HDFS
  repo: https://github.com/jmabuin/MetaCacheSpark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:cd22eebbbef10ded52d9c14cf194ae06b16700362fa634a41c585eef4ca46928
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Proportional read redistribution across taxonomy

## Summary

Estimates organism abundances at a selectable taxonomic rank (e.g. species, via -a/--abundance_per) by pruning the taxonomy below the requested rank, propagating pruned read counts to parents, and redistributing reads assigned to higher-level taxa among children proportionally to sub-tree weights.

## When to use

Use when per-read classifications from MetaCacheSpark/AFS-MetaCache must be converted into per-organism read counts and abundance percentages at a chosen taxonomic rank, and reads sitting on higher-level taxa need redistribution.

## When NOT to use

- The goal is raw per-read taxonomic labels rather than rank-level abundance estimation (plain MetaCacheSpark query output already provides these).
- The input is already a per-organism abundance/feature table — redistribution operates on per-read classifications, not aggregated counts.

## Inputs

- Classified read results from a MetaCacheSpark query run (per-read taxonomic assignments)
- Taxonomy (NCBI taxonomic tree) supporting rank selection and sub-tree weights
- Target taxonomic rank for abundance estimation (e.g. species via -a/--abundance_per)

## Outputs

- Table of estimated read counts per organism at the selected rank
- Abundance percentages per organism written to the output file in HDFS (Output_File)

## How to apply

Load the classified read results from the MetaCacheSpark query run and count reads assigned to each taxon. Prune the taxonomy at the requested rank (selected with the -a/--abundance_per query parameter, e.g. species), adding the read counts of pruned taxa to their parent taxa so no signal is lost. Redistribute reads assigned to higher-level taxa among their children in proportion to sub-tree weights, then compute abundance percentages per organism from the estimated reads. Finally, write the estimated reads and abundance percentages per organism to the output file in HDFS (Output_File). The rationale is that alignment-free classification frequently assigns reads to higher-level taxa, and sub-tree-weighted proportional splitting recovers rank-specific abundance estimates that outperformed Kraken2+Bracken in quantification accuracy.

## Related tools

- **MetaCacheSpark** (Produces the per-read classifications whose taxon counts feed the abundance estimation, and exposes the -a/--abundance_per parameter to trigger quantification at a chosen rank; writes results to HDFS) — https://github.com/jmabuin/MetaCacheSpark
- **AFS-MetaCache** (C++ workstation variant augmented with the same selectable-level abundance estimation used in the quantification module) — https://muellan.github.io/metacache/afs.html
- **Apache Spark** (Distributed engine on which the quantification computation is executed across the cluster with results stored in HDFS) — https://github.com/jmabuin/MetaCacheSpark

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -a species -p <num_partitions> -q threshold <db> <reads.fastq>
```

## Evaluation signals

- The output table lists every organism with its estimated read count and abundance percentage.
- Abundances are consistent with the read counts after redistribution (percentages recomputable from estimated reads).
- Taxa below the requested rank (e.g. below species) no longer appear as separate rows; their counts are folded into parent taxa.
- Reads originally assigned to higher-level taxa are fully redistributed among children — total assigned read mass is conserved.
- Rank-level abundance profiles compare favorably in quantification accuracy against Kraken2+Bracken on admixed food samples (chicken, turkey, pork, beef, horse, sheep).

## Limitations

- Quantification accuracy depends on the quality of per-read classifications; false-positive classifications propagate into abundance estimates.
- The selectable level is limited to ranks present in the taxonomy supplied to MetaCacheSpark/AFS-MetaCache.
- Comparison tools (Kraken2, CLARK, Bracken, Spark) were reported without software version numbers, complicating exact replication of the benchmark.

## Evidence

- [intro] Augment MetaCache with abundance estimation at a selectable taxonomic level and filtering of target references based on sequence coverage: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references"
- [results] pruning and proportional redistribution definition: [evidence span withheld — anchor status: not_found]
- [readme] abundance_per CLI parameter: "-a,--abundance_per <arg>             [QUERY] Indicates if use the
                                          abundance estimation feature and at
                                          which level."
- [abstract] superior quantification accuracy: "In comparison to the established tools CLARK, Kraken2, and Kraken2+Bracken it is superior in terms of false-positive rate and quantification accuracy."
