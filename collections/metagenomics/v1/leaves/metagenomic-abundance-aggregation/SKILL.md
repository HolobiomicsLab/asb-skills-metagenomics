---
name: metagenomic-abundance-aggregation
description: Use when you have reconstructed SSU rRNA sequences with per-sequence
  read-count abundances from a metagenomic pipeline and need to deduplicate near-identical
  sequences at a 97% identity threshold while consolidating their abundances.
when_to_use_negative:
- Input reads are short Illumina reads rather than reconstructed full-length SSU sequences
  — direct classification of short reads yields a high rate of false positives, which
  is what RiboTaxa's reconstruction step avoids
- The input is already a clustered/consolidated abundance table with no per-sequence
  read counts to aggregate
- Chimera filtering is the goal — RiboTaxa did not include chimera detection because
  no chimeric sequences were detected during 16S rRNA gene reconstruction, so this
  step does not remove chimeras
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Clusters reconstructed SSU sequences at 97% identity (CLUSTER_ID = 0.97),
    producing cluster representatives
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBMap
  role: Maps short reads onto reconstructed SSU sequences to obtain per-sequence 'Assigned
    reads' counts that are summed per cluster
  repo: https://github.com/oschakoory/RiboTaxa
- name: EMIRGE
  role: Source of reconstructed full-length SSU rRNA sequences combined into metagenome_SSU_sequences.fasta
  repo: https://github.com/oschakoory/RiboTaxa
- name: MetaRib
  role: Source of reconstructed full-length SSU rRNA sequences combined into metagenome_SSU_sequences.fasta
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:0a0e5da32392988163f3fce6b557cf1b6a737321ad09b23d2a8e94ca0e161d7f
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-abundance-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Clusters full-length/nearly full-length SSU rRNA sequences reconstructed by EMIRGE and MetaRib at 97% identity with VSEARCH and sums their BBMap-derived assigned-read counts into per-cluster abundances prior to taxonomic affiliation in the RiboTaxa pipeline.

## When to use

Use when you have reconstructed SSU rRNA sequences with per-sequence read-count abundances from a metagenomic pipeline and need to deduplicate near-identical sequences at a 97% identity threshold while consolidating their abundances.

## When NOT to use

- Input reads are short Illumina reads rather than reconstructed full-length SSU sequences — direct classification of short reads yields a high rate of false positives, which is what RiboTaxa's reconstruction step avoids
- The input is already a clustered/consolidated abundance table with no per-sequence read counts to aggregate
- Chimera filtering is the goal — RiboTaxa did not include chimera detection because no chimeric sequences were detected during 16S rRNA gene reconstruction, so this step does not remove chimeras

## Inputs

- metagenome_SSU_sequences.fasta (combined EMIRGE and MetaRib full-length/nearly full-length 16S/18S rRNA reconstructed sequences)
- metagenome_SSU_taxonomy_abundance.tsv (per-sequence 'Assigned reads' counts from BBMap short-read mapping onto reconstructed SSU sequences)
- RiboTaxa configuration with CLUSTER_ID = 0.97

## Outputs

- clustered_SSU_sequences.fasta (cluster representative sequences)
- clustered_SSU_abundance.tsv (per-cluster summed assigned reads and Relative_Abundance(%) column)

## How to apply

Load the combined EMIRGE/MetaRib SSU sequences from metagenome_SSU_sequences.fasta and cluster them with VSEARCH v2.17.0 at CLUSTER_ID = 0.97, the identity threshold set in the RiboTaxa configuration, to handle redundant reconstructions of the same organism. Parse metagenome_SSU_taxonomy_abundance.tsv to retrieve each sequence's 'Assigned reads' count obtained by mapping Illumina short reads onto the reconstructed SSU sequences with BBMap. Map each reconstructed sequence to its cluster representative and sum the assigned read counts of all cluster members. Compute each cluster's relative abundance as (summed assigned reads / total assigned reads) x 100, writing the per-cluster table to clustered_SSU_abundance.tsv and cluster representative sequences to clustered_SSU_sequences.fasta; these clustered representatives then feed into QIIME 2's Sklearn classifier for taxonomic affiliation. RiboTaxa deliberately applies no 16S rRNA gene copy-number correction to these abundances.

## Related tools

- **VSEARCH** (Clusters reconstructed SSU sequences at 97% identity (CLUSTER_ID = 0.97), producing cluster representatives) — https://github.com/oschakoory/RiboTaxa
- **BBMap** (Maps short reads onto reconstructed SSU sequences to obtain per-sequence 'Assigned reads' counts that are summed per cluster) — https://github.com/oschakoory/RiboTaxa
- **EMIRGE** (Source of reconstructed full-length SSU rRNA sequences combined into metagenome_SSU_sequences.fasta) — https://github.com/oschakoory/RiboTaxa
- **MetaRib** (Source of reconstructed full-length SSU rRNA sequences combined into metagenome_SSU_sequences.fasta) — https://github.com/oschakoory/RiboTaxa

## Examples

```
vsearch --cluster_fast metagenome_SSU_sequences.fasta --id 0.97 --centroids clustered_SSU_sequences.fasta
```

## Evaluation signals

- Both output files (clustered_SSU_abundance.tsv and clustered_SSU_sequences.fasta) exist and are non-empty
- Every cluster row in clustered_SSU_abundance.tsv carries a non-zero summed read count
- The Relative_Abundance(%) column in clustered_SSU_abundance.tsv sums to 100 across clusters
- Each cluster representative corresponds to sequences sharing >= 97% identity (CLUSTER_ID = 0.97) as produced by VSEARCH
- Summed per-cluster read counts equal the total of 'Assigned reads' values of all member sequences in metagenome_SSU_taxonomy_abundance.tsv

## Limitations

- No 16S rRNA gene copy number normalization is applied, so abundances reflect reads mapped per reconstructed sequence rather than per genome
- Artificial in silico diversity created during full-length sequence reconstruction cannot be excluded and may inflate the number of clusters
- Small indel errors may occur in reconstructed sequences, though these rare indels have little effect on taxonomic classification

## Evidence

- [methods] reconstructed sequences clustered at 97% identity with abundances summed: [evidence span withheld — anchor status: not_found]
- [methods] BBMap-derived assigned reads summed per cluster: [evidence span withheld — anchor status: not_found]
