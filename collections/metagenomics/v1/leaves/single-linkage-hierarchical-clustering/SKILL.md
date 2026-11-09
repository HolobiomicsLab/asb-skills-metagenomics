---
name: single-linkage-hierarchical-clustering
description: Use when you need to partition large volumes of short metagenomic reads
  (e.
when_to_use_negative:
- Input reads are long third-generation reads whose high error rate the paper notes
  introduces issues that still need to be resolved
- You already have species-level binning or a reference genome — reference-assisted
  assembly avoids this de novo clustering path
- A downstream assembler requires contigs, not read clusters — this step only partitions
  reads and performs no assembly itself
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: StrainXpress
  role: Implements the single-linkage clustering ('Clustering Reads') stage that sorts
    overlap scores and merges read clusters; also supports a -fast cluster mode for
    large data sets
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Overlap stage producing the read-pair overlap-score file that is sorted and
    traversed for cluster merging
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Dependency providing overlap-graph-based assembly components (SAVAGE/POLYTE)
    used by StrainXpress around the clustering stage
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:da1654f2e921105a20f6943dd8e5b1ea60b702027a054be3428d5cc8c5796cf4
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-linkage-hierarchical-clustering

## Summary

Cluster metagenomic reads into species-coherent groups by sorting Minimap2 overlap-score pairs in descending order and merging clusters in a single top-to-bottom single-linkage pass, the 'Clustering Reads' first stage of StrainXpress's divide-and-conquer OLC assembly workflow.

## When to use

Use when you need to partition large volumes of short metagenomic reads (e.g. interleaved Illumina FASTQ) into clusters such that reads from the same species fall into the same cluster, as a pre-processing step for strain-aware assembly or as a stand-alone 'XC' pre-grouping step for other assemblers.

## When NOT to use

- Input reads are long third-generation reads whose high error rate the paper notes introduces issues that still need to be resolved
- You already have species-level binning or a reference genome — reference-assisted assembly avoids this de novo clustering path
- A downstream assembler requires contigs, not read clusters — this step only partitions reads and performs no assembly itself

## Inputs

- Interleaved paired-end FASTQ file (all_reads.fq) of metagenomic short reads
- Minimap2 overlap file with per-pair overlap scores

## Outputs

- cluster_assignments.tsv mapping each read ID to its final cluster ID (one non-empty assignment per read)
- Species-coherent read clusters used as input to StrainXpress local and global assembly

## How to apply

Run minimap2 as the overlap stage on the interleaved FASTQ input (all_reads.fq) to produce a read-pair overlap-score file. Sort all overlap-score entries in descending order so the most compatible read pairs appear at the top, then traverse the sorted list once from top to bottom, merging the clusters of the two reads whenever a pair links two distinct clusters — single linkage means the distance between two clusters is the shortest distance between any member of one cluster and any member of the other. This single-pass merging yields O(n(log n + n)) = O(n^2) overall complexity; for big data sets StrainXpress recommends the fast cluster mode. Write the assignments as a TSV mapping each read ID to its final cluster ID; resulting clusters are species-coherent, feeding StrainXpress's subsequent 'Local Assembly' and 'Global Assembly' stages.

## Related tools

- **StrainXpress** (Implements the single-linkage clustering ('Clustering Reads') stage that sorts overlap scores and merges read clusters; also supports a -fast cluster mode for large data sets) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Overlap stage producing the read-pair overlap-score file that is sorted and traversed for cluster merging) — https://github.com/lh3/minimap2
- **HaploConduct** (Dependency providing overlap-graph-based assembly components (SAVAGE/POLYTE) used by StrainXpress around the clustering stage) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq -fast
```

## Evaluation signals

- cluster_assignments.tsv exists, is well-formed TSV, and contains exactly one assignment row per input read with a non-empty cluster ID
- Reads known to originate from the same species (e.g. synthetic benchmark strains) fall into the same cluster
- Number of clusters stays in the expected range (e.g. StrainXpress yielded 899 clusters versus 568 for earlier work on comparable data)
- Pipeline timing check: 'StrainXpress - Clustering' should be dramatically faster than prior overlap-clustering implementations (reported 46x speedup)
- Downstream Local/Global Assembly stages receive non-empty clusters in the stageb folder

## Limitations

- Overall complexity is O(n(log n + n)) = O(n^2); very large data sets should use the -fast cluster mode
- Short length of Illumina-type reads constrains downstream contig length, preventing full-length strain-specific contigs
- Single linkage can chain clusters together; cluster granularity depends on the quality and sorting of the Minimap2 overlap scores

## Evidence

- [results] Clustering stage rationale: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
- [results] Clustering speedup: "‘StrainXpress - Clustering’ is 46 times faster than the earlier work (22)"
- [results] XC clustering as pre-processing: "We considered the application of StrainXpress’s clustering procedure (referred to as ‘XC’ in Table 1) as a pre-processing step for the other methods"
- [readme] Input format: "The input file must be interleaved FASTQ and format like below:"
- [readme] Fast cluster mode: "When a data set is big, we recommend to use the fast cluster method:"
- [readme] Dependencies: "StrainXpress relies on the following dependencies:
- [minimap2](https://github.com/lh3/minimap2)"
