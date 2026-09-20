---
name: ssu-rna-sequence-clustering-vsearch
description: Use when you have full-length/nearly full-length 16S/18S SSU rRNA sequences
  reconstructed from metagenomic reads (e.g. metagenome_SSU_sequences.
when_to_use_negative:
- Input is already a non-redundant cluster representative set or a feature/abundance
  table rather than reconstructed SSU FASTA sequences
- You need to cluster raw short reads or amplicon ASVs rather than full-length reconstructed
  SSU rRNA sequences
- You are applying 16S rRNA gene copy number normalization — RiboTaxa explicitly chooses
  not to correct for copy number, so abundance adjustment at this stage would conflict
  with the pipeline's design
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Clusters the reconstructed SSU sequences at 97% identity (v2.17.0, CLUSTER_ID
    = 0.97) and outputs cluster representatives
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBMap
  role: Provides the per-sequence 'Assigned reads' abundance counts that are summed
    across cluster members
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE
  role: Produces reconstructed full-length SSU rRNA sequences that form the clustering
    input
  repo: https://github.com/oschakoory/RiboTaxa
- name: MetaRib
  role: Produces reconstructed full-length SSU rRNA sequences that form the clustering
    input
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBTools
  role: Part of the RiboTaxa pipeline toolset from which abundances derive; used for
    QC/trimming upstream
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:8473331e3d6c484e71afb12cfa53b08470e2edab59a4f64e10b51f1bfa006152
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
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ssu-rna-sequence-clustering-vsearch

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deduplicate reconstructed SSU rRNA gene sequences (from EMIRGE and MetaRib within RiboTaxa) by clustering at 97% identity with VSEARCH, summing the BBMap-derived 'Assigned reads' abundances of all cluster members before taxonomic affiliation so that relative abundances reflect consolidated clusters rather than redundant reconstructions.

## When to use

Use when you have full-length/nearly full-length 16S/18S SSU rRNA sequences reconstructed from metagenomic reads (e.g. metagenome_SSU_sequences.fasta from EMIRGE/MetaRib) with per-sequence read abundances, and need to collapse sequences sharing ≥97% identity into cluster representatives with summed read counts prior to taxonomic classification.

## When NOT to use

- Input is already a non-redundant cluster representative set or a feature/abundance table rather than reconstructed SSU FASTA sequences
- You need to cluster raw short reads or amplicon ASVs rather than full-length reconstructed SSU rRNA sequences
- You are applying 16S rRNA gene copy number normalization — RiboTaxa explicitly chooses not to correct for copy number, so abundance adjustment at this stage would conflict with the pipeline's design

## Inputs

- metagenome_SSU_sequences.fasta — full-length/nearly full-length 16S/18S rRNA sequences reconstructed by EMIRGE and MetaRib
- metagenome_SSU_taxonomy_abundance.tsv — per-sequence 'Assigned reads' counts obtained by BBMap mapping of short reads onto the reconstructed SSU sequences
- RiboTaxa configuration with CLUSTER_ID = 0.97

## Outputs

- clustered_SSU_sequences.fasta — cluster representative sequences at 97% identity
- clustered_SSU_abundance.tsv — per-cluster table with summed read counts and Relative_Abundance(%) column

## How to apply

Load the reconstructed SSU sequences from metagenome_SSU_sequences.fasta, which combines the full-length 16S/18S rRNA sequences output by EMIRGE and MetaRib. Cluster them with VSEARCH v2.17.0 using the 97% identity threshold (CLUSTER_ID = 0.97 in the RiboTaxa configuration), consistent with the species-level clustering convention. Retrieve per-sequence 'Assigned reads' counts from metagenome_SSU_taxonomy_abundance.tsv (obtained by mapping short reads onto the reconstructed SSU sequences with BBMap), map each sequence to its cluster representative, and sum the counts of all cluster members. Compute each cluster's relative abundance as (summed assigned reads / total assigned reads) × 100, write the per-cluster table to clustered_SSU_abundance.tsv, and write the cluster representative sequences to clustered_SSU_sequences.fasta. The rationale is that EMIRGE and MetaRib can produce near-identical SSU reconstructions; clustering at 97% consolidates these redundancies so downstream taxonomic profiling (QIIME 2 Sklearn classifier) operates on non-redundant representatives with correct combined abundances.

## Related tools

- **VSEARCH** (Clusters the reconstructed SSU sequences at 97% identity (v2.17.0, CLUSTER_ID = 0.97) and outputs cluster representatives) — https://github.com/oschakoory/RiboTaxa
- **BBMap** (Provides the per-sequence 'Assigned reads' abundance counts that are summed across cluster members) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE** (Produces reconstructed full-length SSU rRNA sequences that form the clustering input) — https://github.com/oschakoory/RiboTaxa
- **MetaRib** (Produces reconstructed full-length SSU rRNA sequences that form the clustering input) — https://github.com/oschakoory/RiboTaxa
- **BBTools** (Part of the RiboTaxa pipeline toolset from which abundances derive; used for QC/trimming upstream) — https://github.com/oschakoory/RiboTaxa

## Examples

```
vsearch --cluster_size metagenome_SSU_sequences.fasta --id 0.97 --centroids clustered_SSU_sequences.fasta --uc clusters.uc
```

## Evaluation signals

- Both output files exist: clustered_SSU_abundance.tsv and clustered_SSU_sequences.fasta
- Every cluster row in clustered_SSU_abundance.tsv carries a non-zero summed read count
- The Relative_Abundance(%) column in clustered_SSU_abundance.tsv sums to 100 across clusters
- Each cluster representative in clustered_SSU_sequences.fasta was clustered at ≥97% identity (VSEARCH id 0.97), so no two representatives should exceed 97% mutual identity
- The total of summed assigned reads across clusters equals the total 'Assigned reads' in metagenome_SSU_taxonomy_abundance.tsv

## Limitations

- RiboTaxa did not include chimera detection since no chimeric sequences were detected during 16S rRNA gene reconstruction, so clustering at 97% will not remove chimeric reconstructions if they arise
- Artificial in silico diversity created during full-length sequence reconstruction cannot be excluded and may propagate into cluster counts
- RiboTaxa chooses not to apply 16S rRNA gene copy number correction, so cluster relative abundances do not represent gene-copy-normalized organism abundance

## Evidence
