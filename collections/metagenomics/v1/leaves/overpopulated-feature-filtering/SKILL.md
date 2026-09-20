---
name: overpopulated-feature-filtering
description: Use when building a large partitioned MetaCacheSpark reference database
  on a Spark/HDFS cluster and k-mer/minhash features are shared by so many sequences
  that they risk misclassification; enable via the -o build flag.
when_to_use_negative:
- Building a very small reference database where feature overpopulation is unlikely
  — the README frames the option for cases where 'the user is dealing with a lot of
  data'
- You are running AFS-MetaCache (the single-workstation C++ version) rather than the
  MetaCacheSpark distributed build — this flag belongs to the MetaCacheSpark build
  parameter set
- You are running MetaCacheSpark in query mode (-m query); -o is a BUILD-mode option
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MetaCacheSpark
  role: Provides the -o / --remove_overpopulated_features build-mode flag that triggers
    deletion of overpopulated features during the map-reduce key-count merge across
    Spark executors
  repo: https://github.com/jmabuin/MetaCacheSpark
- name: Apache Spark
  role: Executes the distributed build pipeline (map-reduce global key-count merge)
    in which overpopulated features are filtered
- name: AFS-MetaCache
  role: C++ workstation counterpart of the minhashing classifier; the feature-filtering
    concept is described for the MetaCacheSpark build mode
  repo: https://muellan.github.io/metacache/afs.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-3429-6
    title: A big data approach to metagenomics for all-food-sequencing
  source_content_hash: sha256:8da30b6c9fde2aa1758f04582a832ed2372d5500fb0beb131618a397c79efffb
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
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# overpopulated-feature-filtering

## Summary

Removes overpopulated features (minhash sketches shared by too many reference locations) during MetaCacheSpark database construction, so each reference sequence's items reside in exactly one executor-local C++ hash table and classification errors are avoided.

## When to use

Use when building a large partitioned MetaCacheSpark reference database on a Spark/HDFS cluster and k-mer/minhash features are shared by so many sequences that they risk misclassification; enable via the -o build flag.

## When NOT to use

- Building a very small reference database where feature overpopulation is unlikely — the README frames the option for cases where 'the user is dealing with a lot of data'
- You are running AFS-MetaCache (the single-workstation C++ version) rather than the MetaCacheSpark distributed build — this flag belongs to the MetaCacheSpark build parameter set
- You are running MetaCacheSpark in query mode (-m query); -o is a BUILD-mode option

## Inputs

- Reference genome sequences uploaded to HDFS (FASTA)
- Taxonomy files in HDFS (path given via -t)
- Executor-local C++ hash tables produced by the per-executor minhashing step
- Deletion threshold for overpopulated features (default 254)

## Outputs

- Per-executor partitioned hash table files written to HDFS (e.g. Database_16 at the user home)
- A database in which overpopulated features are removed and each reference sequence's items map to exactly one hash table, loadable in query mode (-m query)

## How to apply

During the build phase (spark-submit -m build), pass the -o / --remove_overpopulated_features flag together with -p (number of partitions, e.g. 16) and -t (HDFS taxonomy path). After executors locally minhash-sketch their distributed reference sequences into C++ hash tables, a map-reduce global key-count merge counts feature occurrences across all executors and deletes features whose occurrence count exceeds the deletion threshold (default 254). Rationale: on big-data-scale reference collections, features present in very many sequences carry little taxonomic signal and can cause errors during the query/classification phase, so removal is recommended whenever MetaCacheSpark processes large inputs. The merge step also guarantees each reference sequence's items reside in exactly one hash table of the partitioned database.

## Related tools

- **MetaCacheSpark** (Provides the -o / --remove_overpopulated_features build-mode flag that triggers deletion of overpopulated features during the map-reduce key-count merge across Spark executors) — https://github.com/jmabuin/MetaCacheSpark
- **Apache Spark** (Executes the distributed build pipeline (map-reduce global key-count merge) in which overpopulated features are filtered)
- **AFS-MetaCache** (C++ workstation counterpart of the minhashing classifier; the feature-filtering concept is described for the MetaCacheSpark build mode) — https://muellan.github.io/metacache/afs.html

## Examples

```
spark-submit --class com.github.jmabuin.metacachespark.MetaCacheSpark MetaCacheSpark-0.4.0.jar -m build -o -p 16 -t /path/to/taxonomy/in/hdfs/ Database_16 /path/to/input/sequences/in/hdfs/
```

## Evaluation signals

- The build job completes with -o set and produces the named database directory (e.g. Database_16) in HDFS at the user home
- The database loads successfully in query mode (spark-submit -m query against Database_16)
- Each reference sequence's items are found in exactly one executor-local hash table after the global key-count merge
- No features with occurrence counts above the deletion threshold (default 254) remain in the merged database

## Limitations

- If -o is omitted on very large inputs, overpopulated features can cause errors during the classification phase, per the README recommendation
- Feature distribution across executors affects which features are counted as overpopulated; distribution is by sequence length unless the -e Spark repartition option is used
- The deletion threshold (default 254) may need adjustment for differently sized or taxonomically redundant reference collections

## Evidence

- [methods] Uses the remove-overpopulated-features flag during build: "* **-o**: Uses the *remove-overpopulated-features*."
- [readme] Overpopulated-feature removal is recommended for large data to avoid classification errors: "this option should be used in order to avoid errors during the classification phase"
- [other] Deletion threshold of 254 during the global key-count merge: [evidence span withheld — anchor status: not_found]
- [other] Executors apply minhashing into local C++ hash tables before the merge: [evidence span withheld — anchor status: not_found]
