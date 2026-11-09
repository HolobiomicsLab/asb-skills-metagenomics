---
name: bbmap-read-count-summarization
description: Use when you have multiple reconstructed SSU sequences (from EMIRGE and
  MetaRib) with per-sequence read counts and need de-redundant cluster representatives
  with combined abundances for downstream taxonomic profiling.
when_to_use_negative:
- Input is already a de-redundant set of SSU cluster representatives with cluster-level
  abundances (no further consolidation is needed before classification).
- You are classifying raw short Illumina reads directly rather than reconstructed
  full-length SSU sequences — the article notes direct classification of short reads
  yields a high rate of false positives.
edam_operation: http://edamontology.org/operation_0557
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Clusters the reconstructed SSU sequences at 97% identity (CLUSTER_ID = 0.97)
    and emits cluster representatives
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBMap
  role: Maps short reads onto reconstructed SSU sequences to produce the per-sequence
    'Assigned reads' counts that are summed per cluster
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE
  role: Source of reconstructed full-length SSU rRNA sequences feeding the clustering
    step
  repo: https://github.com/oschakoory/RiboTaxa
- name: MetaRib
  role: Source of reconstructed full-length SSU rRNA sequences feeding the clustering
    step
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:53e70d8620b15ed629643cef6558af53bac99a3cc104cf2c3fb99ca9808e6f6a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - EMIRGE
  - MetaRib
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0557
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# BBMap read-count summarization after VSEARCH clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consolidates RiboTaxa's reconstructed SSU rRNA sequences by clustering the MetaRib/EMIRGE full-length sequences at 97% identity with VSEARCH v2.17.0, then summing the per-sequence 'Assigned reads' counts (obtained by BBMap short-read mapping) for all members of each cluster to obtain per-cluster abundances prior to taxonomic affiliation.

## When to use

Use when you have multiple reconstructed SSU sequences (from EMIRGE and MetaRib) with per-sequence read counts and need de-redundant cluster representatives with combined abundances for downstream taxonomic profiling.

## When NOT to use

- Input is already a de-redundant set of SSU cluster representatives with cluster-level abundances (no further consolidation is needed before classification).
- You are classifying raw short Illumina reads directly rather than reconstructed full-length SSU sequences — the article notes direct classification of short reads yields a high rate of false positives.

## Inputs

- metagenome_SSU_sequences.fasta (combined full-length/nearly full-length 16S/18S rRNA sequences reconstructed by EMIRGE and MetaRib)
- metagenome_SSU_taxonomy_abundance.tsv (per-sequence 'Assigned reads' counts from BBMap short-read mapping)

## Outputs

- clustered_SSU_sequences.fasta (97% cluster representative SSU sequences)
- clustered_SSU_abundance.tsv (per-cluster summed assigned reads and Relative_Abundance(%) column)

## How to apply

Load the final reconstructed SSU sequences from metagenome_SSU_sequences.fasta, which combines the full-length/nearly full-length 16S/18S rRNA sequences output by EMIRGE and MetaRib. Cluster these sequences with VSEARCH v2.17.0 at the 97% identity threshold (CLUSTER_ID = 0.97 in the RiboTaxa configuration), writing cluster representatives to clustered_SSU_sequences.fasta. Parse metagenome_SSU_taxonomy_abundance.tsv to retrieve each reconstructed sequence's 'Assigned reads' count obtained by mapping short reads onto the SSU sequences with BBMap. Map each sequence to its cluster representative, sum the assigned read counts of all cluster members, and compute each cluster's relative abundance as (summed assigned reads / total assigned reads) x 100, writing the per-cluster table to clustered_SSU_abundance.tsv. This consolidation prevents near-identical reconstructions from inflating taxon abundances before classification with QIIME 2's Sklearn classifier.

## Related tools

- **VSEARCH** (Clusters the reconstructed SSU sequences at 97% identity (CLUSTER_ID = 0.97) and emits cluster representatives) — https://github.com/oschakoory/RiboTaxa
- **BBMap** (Maps short reads onto reconstructed SSU sequences to produce the per-sequence 'Assigned reads' counts that are summed per cluster) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE** (Source of reconstructed full-length SSU rRNA sequences feeding the clustering step) — https://github.com/oschakoory/RiboTaxa
- **MetaRib** (Source of reconstructed full-length SSU rRNA sequences feeding the clustering step) — https://github.com/oschakoory/RiboTaxa

## Examples

```
vsearch --cluster_fast metagenome_SSU_sequences.fasta --id 0.97 --centroids clustered_SSU_sequences.fasta
```

## Evaluation signals

- Both output files (clustered_SSU_sequences.fasta and clustered_SSU_abundance.tsv) exist after the step.
- Every cluster row in clustered_SSU_abundance.tsv carries a non-zero summed read count.
- The Relative_Abundance(%) column sums to 100 across clusters.
- The number of cluster representatives in clustered_SSU_sequences.fasta equals the number of rows in clustered_SSU_abundance.tsv.
- Clustering used the 97% identity threshold, matching CLUSTER_ID = 0.97 in the RiboTaxa configuration.

## Limitations

- No 16S rRNA gene copy number normalization is applied in RiboTaxa, so abundances are read-based rather than organism-based.
- Part of the reconstructed diversity may be artificial in silico diversity created during full-length sequence reconstruction, which clustering at 97% may not fully collapse.
- Occasional small indel errors in reconstructed sequences could affect clustering and abundance attribution, though in practice these rare indels have little effect on taxonomic classification.
- Chimera detection was not included in the reconstruction workflow, so chimeric sequences could carry into clusters and inflate abundances.

## Evidence

- [methods] Cluster the reconstructed sequences with VSEARCH (v2.17.0) using the 97% identity threshold, matching the CLUSTER_ID = 0.97 setting defined in the RiboTaxa configuration.: [evidence span withheld — anchor status: not_found]
- [abstract] Compiled tools in RiboTaxa pipeline: "BBTools, FastQC, SortMeRNA, MetaRib, EMIRGE, VSEARCH, BBMap and QIIME 2’s Sklearn classifier"
