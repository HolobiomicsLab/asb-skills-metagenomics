---
name: ssu-fasta-output-processing
description: Use when you have full-length/nearly full-length 16S/18S SSU sequences
  reconstructed by EMIRGE and MetaRib (e.g. metagenome_SSU_sequences.fasta) plus per-sequence
  abundance counts, and need clustered representatives and consolidated abundances
  prior to taxonomic classification.
when_to_use_negative:
- Input is already a clustered representative FASTA or an abundance table at the cluster
  level — re-clustering would merge distinct taxa.
- You are classifying short Illumina reads directly rather than reconstructed full-length
  SSU sequences; RiboTaxa showed direct short-read classification yields a high rate
  of false positives.
- Input sequences are amplicon ASVs already denoised/merged at 100% (e.g. via EMIRGE
  JOIN_TRESHOLD identity merging) and no abundance consolidation across reconstructions
  is needed.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Clusters the reconstructed MetaRib/EMIRGE SSU sequences at the 97% identity
    threshold and emits cluster representative sequences
- name: BBMap
  role: Maps short reads onto reconstructed SSU sequences to produce the 'Assigned
    reads' counts that are summed per cluster
- name: EMIRGE
  role: Upstream producer of full-length SSU rRNA sequence reconstructions that feed
    the clustering step
- name: MetaRib
  role: Upstream producer of full-length SSU rRNA sequence reconstructions that feed
    the clustering step
- name: RiboTaxa
  role: Pipeline whose configuration (CLUSTER_ID = 0.97) and outputs (clustered_SSU_abundance.tsv,
    clustered_SSU_sequences.fasta) define this processing step
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:87bbf431a8bac2085e7635b73a24539b3220a6d423e690a94e0a90235a7f8fa0
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
  - RiboTaxa
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ssu-fasta-output-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deduplicate reconstructed SSU rRNA gene sequences before taxonomic affiliation by clustering the EMIRGE/MetaRib outputs at 97% identity with VSEARCH and summing BBMap-assigned read counts per cluster so relative abundances reflect clusters rather than redundant reconstructed variants.

## When to use

Use when you have full-length/nearly full-length 16S/18S SSU sequences reconstructed by EMIRGE and MetaRib (e.g. metagenome_SSU_sequences.fasta) plus per-sequence abundance counts, and need clustered representatives and consolidated abundances prior to taxonomic classification.

## When NOT to use

- Input is already a clustered representative FASTA or an abundance table at the cluster level — re-clustering would merge distinct taxa.
- You are classifying short Illumina reads directly rather than reconstructed full-length SSU sequences; RiboTaxa showed direct short-read classification yields a high rate of false positives.
- Input sequences are amplicon ASVs already denoised/merged at 100% (e.g. via EMIRGE JOIN_TRESHOLD identity merging) and no abundance consolidation across reconstructions is needed.

## Inputs

- metagenome_SSU_sequences.fasta — combined full-length/nearly full-length 16S/18S rRNA sequences from EMIRGE and MetaRib
- metagenome_SSU_taxonomy_abundance.tsv — per-sequence 'Assigned reads' counts from BBMap mapping of short reads

## Outputs

- clustered_SSU_sequences.fasta — VSEARCH cluster representative (centroid) sequences at 97% identity
- clustered_SSU_abundance.tsv — per-cluster summed read counts and Relative_Abundance(%) column

## How to apply

Load the final reconstructed SSU sequences from metagenome_SSU_sequences.fasta, which combines the full-length/nearly full-length 16S/18S rRNA sequences output by EMIRGE and MetaRib. Cluster these sequences with VSEARCH v2.17.0 at the 97% identity threshold (CLUSTER_ID = 0.97 in the RiboTaxa configuration), writing cluster representatives to clustered_SSU_sequences.fasta. Parse metagenome_SSU_taxonomy_abundance.tsv to retrieve per-sequence 'Assigned reads' counts obtained by mapping short reads onto the reconstructions with BBMap. Sum the assigned read counts of all members sharing a cluster representative, then compute each cluster's relative abundance as (summed assigned reads / total assigned reads) x 100 and write the per-cluster table to clustered_SSU_abundance.tsv. The rationale is that reconstructed sequence variants from the same organism should contribute a single consolidated abundance before taxonomic affiliation with the QIIME 2 Sklearn classifier.

## Related tools

- **VSEARCH** (Clusters the reconstructed MetaRib/EMIRGE SSU sequences at the 97% identity threshold and emits cluster representative sequences)
- **BBMap** (Maps short reads onto reconstructed SSU sequences to produce the 'Assigned reads' counts that are summed per cluster)
- **EMIRGE** (Upstream producer of full-length SSU rRNA sequence reconstructions that feed the clustering step)
- **MetaRib** (Upstream producer of full-length SSU rRNA sequence reconstructions that feed the clustering step)
- **RiboTaxa** (Pipeline whose configuration (CLUSTER_ID = 0.97) and outputs (clustered_SSU_abundance.tsv, clustered_SSU_sequences.fasta) define this processing step) — https://github.com/oschakoory/RiboTaxa

## Examples

```
vsearch --cluster_fast metagenome_SSU_sequences.fasta --id 0.97 --centroids clustered_SSU_sequences.fasta --uc clusters.uc
```

## Evaluation signals

- Both output files exist: clustered_SSU_abundance.tsv and clustered_SSU_sequences.fasta.
- Every cluster row in clustered_SSU_abundance.tsv carries a non-zero summed read count.
- The Relative_Abundance(%) column sums to 100 across clusters.
- The number of cluster representatives in clustered_SSU_sequences.fasta equals the number of rows in clustered_SSU_abundance.tsv.
- Clustering was run at 0.97 identity, matching the CLUSTER_ID = 0.97 setting in the RiboTaxa configuration.

## Limitations

- No chimera detection was included in the 16S rRNA gene reconstruction workflow, so chimeric reconstructions may form their own clusters and inflate abundance.
- A part of the reconstructed diversity may originate from artificial in silico diversity created during full-length sequence reconstruction, which clustering at 97% may not fully collapse.
- No 16S rRNA gene copy number normalization is applied, so summed read abundances are not corrected for copy number variation.

## Evidence

- [other] other: [evidence span withheld — anchor status: not_found]
