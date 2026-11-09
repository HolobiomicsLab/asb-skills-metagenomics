---
name: quantile-based-candidate-genome-filtering
description: Use when a MetaCacheSpark query yields a candidate genome list with spurious
  hits (e.g. shared conserved regions inflating hit counts) and reads must be assigned
  only to genuinely covered references, as in the all-food-sequencing KLyo_C sample.
when_to_use_negative:
- The goal is to reduce database size at build time by pruning k-mer features — that
  is the separate remove-overpopulated-features step (-o flag), not read-coverage
  candidate filtering.
- No candidate genome list exists (raw reads with no prior MetaCacheSpark classification),
  so there are no per-genome coverage profiles to quantile-filter.
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MetaCacheSpark
  role: Performs the query-mode classification producing per-genome candidate hits
    and window coverage information that the quantile filter acts on
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: AFS-MetaCache
  role: C++ workstation implementation whose coverage-filter behavior (deleting lowest-coverage
    quantile of candidate genomes) is being reconstructed
  repo: https://muellan.github.io/metacache/afs.html
- name: Apache Spark
  role: Distributed execution engine for MetaCacheSpark query and filtering runs on
    big data clusters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:45f4b93807ab7a8d36a0b504c47e857bfc024f2693641682f8f3c56eefcfaa85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jmabuin/MetaCacheSpark
  tools:
  - AFS-MetaCache
  - Apache Spark
  - MetaCacheSpark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# quantile-based-candidate-genome-filtering

## Summary

A coverage-based false-positive removal step in the AFS-MetaCache pipeline: candidate target genomes returned by MetaCacheSpark whose fraction of read-covered windows falls in the lowest quantile (e.g. 10%) are deleted before final read assignment, eliminating candidates arising from shared DNA regions.

## When to use

Use when a MetaCacheSpark query yields a candidate genome list with spurious hits (e.g. shared conserved regions inflating hit counts) and reads must be assigned only to genuinely covered references, as in the all-food-sequencing KLyo_C sample.

## When NOT to use

- The goal is to reduce database size at build time by pruning k-mer features — that is the separate remove-overpopulated-features step (-o flag), not read-coverage candidate filtering.
- No candidate genome list exists (raw reads with no prior MetaCacheSpark classification), so there are no per-genome coverage profiles to quantile-filter.

## Inputs

- MetaCacheSpark query-mode output (-m query) mapping paired reads to candidate target genomes
- Reference genome database (e.g. AFS31RS90) with window-partitioned target sequences
- Percentage quantile of lowest-coverage candidates to remove (default 10%)

## Outputs

- Filtered candidate genome list (retained references) used for final taxonomic assignment
- coverage_filter_report.csv recording per-genome window coverage and filter decisions

## How to apply

Run MetaCacheSpark in query mode (-m query) against the reference database (e.g. AFS31RS90) on paired reads (-r), enabling abundance estimation at a selectable level (-a species), and parse the query results to group assigned reads by candidate genome. For each candidate target genome, compute which reference windows are covered by reads, forming per-genome coverage profiles. Sort candidates by percentage of covered windows and delete the lowest-coverage quantile (10% by default) as false positives — rationale: true ingredient genomes show even, dense window coverage, whereas false positives from shared DNA regions cover only sparse, uneven windows. Re-run or finalize the classification against the retained genomes so reads are assigned to a filtered candidate set, and record filter decisions (removed vs. retained) in a report such as coverage_filter_report.csv.

## Related tools

- **MetaCacheSpark** (Performs the query-mode classification producing per-genome candidate hits and window coverage information that the quantile filter acts on) — https://github.com/jmabuin/MetaCacheSpark
- **AFS-MetaCache** (C++ workstation implementation whose coverage-filter behavior (deleting lowest-coverage quantile of candidate genomes) is being reconstructed) — https://muellan.github.io/metacache/afs.html
- **Apache Spark** (Distributed execution engine for MetaCacheSpark query and filtering runs on big data clusters)

## Examples

```
spark-submit --class com.github.metachachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m query -r -a species -q threshold
```

## Evaluation signals

- coverage_filter_report.csv places Actinoalloteichus (sparse, uneven window coverage) in the filtered-out set and Brochothrix thermosphacta (even coverage) in the retained set, matching Figs. 5-6 and Table 11
- The number of removed candidates equals the chosen quantile (e.g. 10%) of the ranked candidate list
- Final taxonomic assignment uses only the retained-genome candidate set — no reads map to deleted references
- Retained genomes show high, uniform percentages of covered windows across their reference sequences

## Limitations

- Genuine low-abundance ingredients could be wrongly deleted if their covered-window percentage falls in the removed quantile
- Coverage is computed over candidate genomes returned in the query phase, so candidates never surfaced cannot be recovered by this filter

## Evidence

- [intro] estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage: "MetaCache is augmented with the ability to estimate the abundance of organisms at a selectable taxonomic level as well as the possibility to filter out target references based on sequence coverage"
- [results] quantile removal of lowest-coverage candidates: [evidence span withheld — anchor status: not_found]
- [discussion] window-based candidate matching: "We apply an intelligent subsampling technique based on minhashing within local windows to reduce the database size."
- [readme] query-mode flags in README: " -o,--remove_overpopulated_features   [BUILD] Uses remove overpopulated features when building."
