---
name: metagenomic-read-clustering
description: Use when metagenomic Illumina short reads must be split into strain/species-coherent
  read groups before per-cluster assembly, e.g. as the preprocessing step of an overlap-layout-consensus
  strain-aware assembly pipeline.
when_to_use_negative:
- Input is long noisy third-generation reads, whose high error rate StrainXpress's
  short-read workflow does not resolve
- Reads are already partitioned into strain/species groups or a reference-guided binning
  already exists, making de novo overlap-based clustering redundant
- The goal is long contigs spanning whole strain genomes from Illumina data, which
  the article notes is constrained by short read length regardless of clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: StrainXpress
  role: Implements the single-linkage overlap-score clustering; invoked via strainxpress.py
    with the -fast option recommended for big data sets
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: 'Overlap stage: computes read-pair overlap scores from the interleaved FASTQ
    input that feed the clustering'
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency of StrainXpress installation, linked as prior overlap-graph-based
    tooling (SAVAGE/POLYTE) from which StrainXpress's OLC approach builds
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:73f521f4f5f3e0e94270c11034bc54507c609649f56387790d5baa7113d2bc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metagenomic-read-clustering

## Summary

Clusters metagenomic short reads into small, species-coherent groups via single-linkage merging of Minimap2 overlap scores, serving as the 'Clustering Reads' preprocessing stage of StrainXpress's OLC divide-and-conquer assembly workflow.

## When to use

Use when metagenomic Illumina short reads must be split into strain/species-coherent read groups before per-cluster assembly, e.g. as the preprocessing step of an overlap-layout-consensus strain-aware assembly pipeline.

## When NOT to use

- Input is long noisy third-generation reads, whose high error rate StrainXpress's short-read workflow does not resolve
- Reads are already partitioned into strain/species groups or a reference-guided binning already exists, making de novo overlap-based clustering redundant
- The goal is long contigs spanning whole strain genomes from Illumina data, which the article notes is constrained by short read length regardless of clustering

## Inputs

- Interleaved paired-end FASTQ file (all_reads.fq, e.g. Illumina MiSeq reads)
- Minimap2 read-overlap file with per-pair overlap scores (PAF-style overlap output)

## Outputs

- cluster_assignments.tsv: one row per input read mapping read ID to a non-empty final cluster ID
- Cluster-level output in the stageb folder used by subsequent Local Assembly and Global Assembly stages

## How to apply

Run Minimap2 on the interleaved FASTQ input (all_reads.fq) to produce a read-pair overlap-score file, which is the overlap stage of the StrainXpress OLC pipeline. Sort all read-pair overlap-score entries in descending score order so the most compatible pairs appear first. Perform a single top-to-bottom pass over the sorted list: whenever a pair links two distinct clusters, merge them (single-linkage, where inter-cluster distance is the shortest distance between any member of one cluster and any member of the other); overall complexity is O(n(log n + n)) = O(n^2). For large data sets, use StrainXpress's fast cluster method via the -fast flag. Write cluster assignments as a TSV mapping each read ID to its final cluster ID so reads from the same species fall into the same cluster; results are deposited in the stageb folder.

## Related tools

- **StrainXpress** (Implements the single-linkage overlap-score clustering; invoked via strainxpress.py with the -fast option recommended for big data sets) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Overlap stage: computes read-pair overlap scores from the interleaved FASTQ input that feed the clustering) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency of StrainXpress installation, linked as prior overlap-graph-based tooling (SAVAGE/POLYTE) from which StrainXpress's OLC approach builds) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- cluster_assignments.tsv exists and is well-formed TSV
- The file contains exactly one assignment row per input read, each with a non-empty cluster ID
- Reads from the same species fall into the same cluster (species coherence check)
- Sorted-list pass processes pairs in strictly descending overlap-score order; cluster count is comparable to the article's reported scale (e.g. 899 clusters vs 568 in the XC comparison)
- stageb folder is produced containing the clustered output for downstream Local/Global Assembly

## Limitations

- Single-linkage merge is O(n^2) overall, so the -fast cluster mode is recommended for big data sets
- Short Illumina read length constrains contig length downstream, preventing strain-specific contigs spanning entire genomes
- The clustering ('XC') stage alone as preprocessing yields fewer clusters (568) than full StrainXpress (899), which limits downstream strain resolution

## Evidence

- [results] Workflow stage listing: [evidence span withheld — anchor status: not_found]
- [results] Clustering speed: [evidence span withheld — anchor status: not_found]
- [results] Cluster count comparison: "StrainXpress yields 899 clusters, instead of only 568 clusters"
- [readme] Interleaved FASTQ input requirement: "The input file must be interleaved FASTQ and format like below"
- [readme] Fast cluster flag for big data sets: "When a data set is big, we recommend to use the fast cluster method:"
