---
name: cluster-assignment-table-formatting
description: Use when you have run Minimap2 overlap scoring plus StrainXpress single-linkage
  clustering on metagenomic short reads and need to write or validate a per-read cluster
  assignment TSV as the deliverable of the 'Clustering Reads' stage.
when_to_use_negative:
- The input is third-generation long-read data (PacBio/Nanopore) rather than Illumina-type
  short reads — the workflow targets interleaved short-read FASTQ and high error rates
  remain unresolved.
- You need strain-resolved contigs rather than read-to-cluster groupings — the assignment
  TSV is only the pre-processing output feeding the subsequent 'Local Assembly' and
  'Global Assembly' stages.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: StrainXpress
  role: performs the single-linkage read clustering ('Clustering Reads' stage) whose
    final assignments are formatted as the TSV
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: produces the read-pair overlap-score file that, once sorted by score, drives
    the single-linkage merge
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:3ed641dbcfec9c1f172b9a000ba5c26585513409a02d1ac5bc6f1ee41df21d62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cluster-assignment-table-formatting

## Summary

Emit and validate the read-to-cluster TSV (cluster_assignments.tsv) produced by StrainXpress's 'Clustering Reads' stage, which single-linkage merges reads from the interleaved FASTQ (all_reads.fq) after Minimap2 overlap scoring so that reads from the same species share a cluster ID.

## When to use

Use when you have run Minimap2 overlap scoring plus StrainXpress single-linkage clustering on metagenomic short reads and need to write or validate a per-read cluster assignment TSV as the deliverable of the 'Clustering Reads' stage.

## When NOT to use

- The input is third-generation long-read data (PacBio/Nanopore) rather than Illumina-type short reads — the workflow targets interleaved short-read FASTQ and high error rates remain unresolved.
- You need strain-resolved contigs rather than read-to-cluster groupings — the assignment TSV is only the pre-processing output feeding the subsequent 'Local Assembly' and 'Global Assembly' stages.

## Inputs

- interleaved FASTQ file of metagenomic short reads (all_reads.fq, @ID/1 @ID/2 pairs)
- Minimap2 read-pair overlap-score file (sorted in descending overlap score)
- final cluster assignments from the single-linkage merge pass over the sorted overlap list

## Outputs

- cluster_assignments.tsv — well-formed TSV mapping each read ID to its final cluster ID, one row per input read, non-empty cluster IDs

## How to apply

Feed the interleaved FASTQ (all_reads.fq) through StrainXpress, which runs Minimap2 to produce the overlap-score file, sorts read-pair entries in descending overlap score so the most compatible pairs come first, and merges clusters in a single top-to-bottom pass (single linkage: distance between clusters is the shortest distance between any member of one and any member of the other, O(n(log n + n)) overall). For big data sets, invoke the fast cluster method with the -fast flag as the README recommends. After the merge pass, write the final assignments as a TSV mapping each read ID to its final cluster ID. Validate that cluster_assignments.tsv exists, parses as well-formed TSV, contains exactly one assignment row per input read, and that no cluster ID is empty; species coherence (same-species reads in one cluster) is the biological expectation.

## Related tools

- **StrainXpress** (performs the single-linkage read clustering ('Clustering Reads' stage) whose final assignments are formatted as the TSV) — https://github.com/HaploKit/StrainXpress
- **minimap2** (produces the read-pair overlap-score file that, once sorted by score, drives the single-linkage merge) — https://github.com/lh3/minimap2

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- cluster_assignments.tsv exists and parses as well-formed TSV with read-ID and cluster-ID columns.
- Row count equals the number of reads in all_reads.fq — exactly one assignment row per input read.
- No assignment row has an empty cluster ID.
- Species coherence: reads originating from the same species carry the same cluster ID.
- Cluster count and runtime are consistent with the method's reported behaviour (e.g. 899 clusters vs 568 for the earlier approach; 'Clustering' 46x faster than prior work).

## Limitations

- Single-linkage merging over the sorted overlap list costs O(n(log n + n)) ≈ O(n²); for large data sets the README recommends the fast cluster method (-fast).
- Clustering is species-coherent, not strain-resolved — separating strains happens downstream in Local/Global Assembly.
- Illumina short-read length constrains downstream contig lengths, and high third-generation read error rates remain unresolved in this pipeline.

## Evidence

- [readme] The input file must be interleaved FASTQ and format like below:: "The input file must be interleaved FASTQ and format like below:"
- [results] 'StrainXpress - Clustering' is 46 times faster than the earlier work (22): "‘StrainXpress - Clustering’ is 46 times faster than the earlier work (22)"
- [results] StrainXpress yields 899 clusters, instead of only 568 clusters: "StrainXpress yields 899 clusters, instead of only 568 clusters"
- [results] the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’: "the parts that refer to the different stages of StrainXpress, ‘Clustering Reads’, ‘Local Assembly’ and ‘Global Assembly’"
