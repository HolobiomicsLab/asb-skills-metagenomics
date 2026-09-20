---
name: genome-coverage-profile-computation-from-assigned-reads
description: Use when metagenomic reads (e.g., paired-end FASTQ such as KLyo_C) have
  been classified against a reference database and you need per-genome window coverage
  profiles to identify false-positive candidate genomes before final taxonomic assignment.
when_to_use_negative:
- Input is a raw read file with no prior classification — coverage profiles require
  query output assigning reads to candidate genomes first
- The classification task has no suspected false-positive candidates from shared DNA
  regions and no target-reference filtering step is desired (the filter step is optional
  in the AFS pipeline)
- You need alignment-level base-pair coverage (e.g., BWA/Bowtie2 pileups); this skill
  uses MetaCache's window-level minhashing hits, not per-base alignment coverage
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: AFS-MetaCache
  role: C++ workstation implementation whose query output supplies candidate-genome
    assignments and window hits for coverage profile computation
  repo: https://muellan.github.io/metacache/afs.html
- name: MetaCacheSpark
  role: Spark implementation run in query mode (-m query) with abundance estimation
    (-a species) and paired reads (-r) to produce the classified reads from which
    coverage profiles are computed
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Big data engine executing MetaCacheSpark query and classification phases on
    a cluster
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:6a8a32f3ee02b66924c1c0991d06ebcfcf7b1c1654a8ff0dfadd96af12449f5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# Genome coverage profile computation from assigned reads

## Summary

Groups MetaCache/MetaCacheSpark query-assigned reads by candidate reference genome and computes, for each genome, the fraction of reference windows covered by reads — the per-genome coverage profile used downstream by the coverage-based false-positive filter in AFS-MetaCache.

## When to use

Use when metagenomic reads (e.g., paired-end FASTQ such as KLyo_C) have been classified against a reference database and you need per-genome window coverage profiles to identify false-positive candidate genomes before final taxonomic assignment.

## When NOT to use

- Input is a raw read file with no prior classification — coverage profiles require query output assigning reads to candidate genomes first
- The classification task has no suspected false-positive candidates from shared DNA regions and no target-reference filtering step is desired (the filter step is optional in the AFS pipeline)
- You need alignment-level base-pair coverage (e.g., BWA/Bowtie2 pileups); this skill uses MetaCache's window-level minhashing hits, not per-base alignment coverage

## Inputs

- MetaCache/MetaCacheSpark query output file mapping reads to candidate reference genomes (from -m query on paired FASTQ, e.g. KLyo_C reads against the AFS31RS90 reference database)
- Reference genome sequences/database targets partitioned into windows (built database, e.g. AFS31RS90)

## Outputs

- Per-candidate-genome window coverage profiles (fraction of reference windows covered by assigned reads)
- Partition of candidate genomes into lowest-coverage quantile (default 10%, e.g. Actinoalloteichus) vs. retained genomes (e.g. Brochothrix thermosphacta)
- coverage_filter_report.csv documenting filter decisions and comparison to published figures

## How to apply

Run MetaCacheSpark (or AFS-MetaCache) in query mode against the built reference database on the sample's paired reads, enabling abundance estimation at species level (-a species); for KLyo_C this is the AFS31RS90 database with remove-overpopulated-features (-o) and paired reads (-r). Parse the query output to group assigned reads by candidate genome and, for each candidate reference sequence, partition the target into small windows (MetaCache's local-window scheme, e.g. default window length and k-mer size of 16) and mark which windows are hit by at least one assigned read. The result is a per-genome profile of covered vs. uncovered windows; genomes whose percentage of covered windows is much lower than others — e.g., sparse and uneven coverage from shared DNA regions — are candidates for removal (Actinoalloteichus in the paper), while genomes with even coverage (Brochothrix thermosphacta) are retained. The lowest-coverage quantile (10% by default) is then deleted before the final read classification is fixed. Write the per-genome coverage decisions to a report such as coverage_filter_report.csv so the filter outcome can be verified against Figs. 5-6 and Table 11.

## Related tools

- **AFS-MetaCache** (C++ workstation implementation whose query output supplies candidate-genome assignments and window hits for coverage profile computation) — https://muellan.github.io/metacache/afs.html
- **MetaCacheSpark** (Spark implementation run in query mode (-m query) with abundance estimation (-a species) and paired reads (-r) to produce the classified reads from which coverage profiles are computed) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Big data engine executing MetaCacheSpark query and classification phases on a cluster)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -r -a species -o -q threshold AFS31RS90 KLyo_C_1.fa KLyo_C_2.fa -out classification.txt
```

## Evaluation signals

- Coverage report shows the false-positive genome (Actinoalloteichus, sparse and uneven window coverage) in the filtered set, matching Figs. 5-6 and Table 11
- Coverage report shows the even-coverage genome (Brochothrix thermosphacta) in the retained set
- Deleted candidate set equals the lowest-coverage quantile (10% by default) ranked by percentage of covered windows
- Per-genome profile groups every assigned read to exactly one candidate genome with no reads dropped during the grouping step
- Final read classification, re-run on the filtered candidate set, changes assignments only for reads previously attributed to deleted genomes

## Limitations

- Genomes sharing DNA regions with abundant true organisms receive sparse, uneven coverage and can be wrongly present as candidates; the quantile filter (default 10%) mitigates but may not catch all such cases
- A fixed 10% quantile is a heuristic: if more than 10% of candidates are false positives from shared regions, some will survive the filter
- MetaCacheSpark only supports build and query modes (plus add/info/annotate per README); coverage profile computation and filtering must be done by parsing the query output outside the tool
- The README notes databases can only be queried with the same MetaCache variant/data-type settings they were built with, so coverage profiles depend on a compatible database build

## Evidence

- [methods] coverage filter on window coverage: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [readme] abundance estimation flag in query mode: "-a,--abundance_per <arg>             [QUERY] Indicates if use the abundance estimation feature and at which level."
- [discussion] minhashing within local windows: "We apply an intelligent subsampling technique based on minhashing within local windows to reduce the database size."
- [readme] paired reads query option: "-r,--paired_reads                    [QUERY] Use paired reads in the"
- [readme] database/query variant compatibility: [evidence span withheld — anchor status: external_doc]
