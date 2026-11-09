---
name: relative-abundance-computation-metagenomics
description: 'Use when you need relative abundances for reconstructed full-length
  SSU rRNA sequences from metagenomics data: map reads with BBMap, sum counts within
  97% VSEARCH clusters, and report percent abundances.'
when_to_use_negative:
- When the goal is to assign taxonomy directly to short-length Illumina reads without
  reconstruction — the article shows this yields a high rate of false positives
- When abundances must be corrected for 16S rRNA gene copy number — RiboTaxa explicitly
  chose not to use gene copy number correction
- When the input is already a taxonomic abundance table rather than FASTQ reads plus
  reconstructed SSU FASTA
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BBMap
  role: Maps trimmed metagenomic short reads back onto reconstructed SSU sequences
    to count assigned reads per sequence
- name: VSEARCH
  role: Clusters reconstructed SSU sequences at the 97% identity threshold (CLUSTER_ID
    = 0.97) before abundances are summed per cluster
- name: EMIRGE
  role: Produces the reconstructed full-length SSU rRNA sequences whose abundances
    are computed
- name: MetaRib
  role: Produces the reconstructed full-length SSU rRNA sequences whose abundances
    are computed
- name: QIIME 2 Sklearn classifier
  role: Assigns taxonomic lineages with a confidence threshold of 0.7 that are combined
    with the abundance columns
- name: RiboTaxa
  role: The pipeline that implements clustering, read mapping, and relative abundance
    computation end-to-end
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:e429c026108bf11aa78c5c44f6e708970d158b933a6a0719402d4118fbacfed5
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
  - QIIME 2 Sklearn classifier
  - RiboTaxa
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# relative-abundance-computation-metagenomics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-sequence and per-cluster relative abundances of reconstructed SSU rRNA sequences by mapping short metagenomic reads with BBMap, summing abundances of VSEARCH 97% clusters, and reporting percentages without 16S rRNA gene copy-number correction, as done in the RiboTaxa pipeline.

## When to use

Use when you need relative abundances for reconstructed full-length SSU rRNA sequences from metagenomics data: map reads with BBMap, sum counts within 97% VSEARCH clusters, and report percent abundances.

## When NOT to use

- When the goal is to assign taxonomy directly to short-length Illumina reads without reconstruction — the article shows this yields a high rate of false positives
- When abundances must be corrected for 16S rRNA gene copy number — RiboTaxa explicitly chose not to use gene copy number correction
- When the input is already a taxonomic abundance table rather than FASTQ reads plus reconstructed SSU FASTA

## Inputs

- metagenome_SSU_sequences.fasta: full-length/nearly full-length SSU rRNA sequences reconstructed by EMIRGE and MetaRib
- trimmed metagenomic short reads (Illumina FASTQ, single-end or paired-end)
- metagenome_SSU_taxonomy_abundance.tsv with per-sequence 'Assigned reads' counts
- QIIME 2 SILVA 138 trained classifier (silva-138-99-nb-classifier.qza) for taxonomic lineages

## Outputs

- SSU_taxonomy_abundance.tsv: twelve-column table (Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%))
- clustered_SSU_abundance.tsv: per-cluster summed read counts and Relative_Abundance(%)
- SSU_sequences.fasta / clustered_SSU_sequences.fasta: clustered representative SSU sequences

## How to apply

Map the trimmed metagenomic short reads back onto the reconstructed SSU sequences (metagenome_SSU_sequences.fasta) using BBMap and count assigned reads per reconstructed sequence. Compute relative abundance in percent as the number of assigned reads per sequence divided by the sum of all assigned reads multiplied by 100, deliberately without 16S rRNA gene copy-number correction, as RiboTaxa does. To consolidate redundancies, cluster the EMIRGE/MetaRib reconstructed sequences with VSEARCH v2.17.0 at the CLUSTER_ID = 0.97 setting, map each sequence to its cluster representative, and sum the BBMap assigned read counts of all cluster members. Combine the abundances with taxonomic lineages assigned by the QIIME 2 sklearn classifier (SILVA 138, confidence threshold 0.7) and write SSU_taxonomy_abundance.tsv with the twelve columns Sequence_ID, Domain, Phylum, Class, Order, Family, Genus, Species, Confidence, Length(bp), Assigned reads, Relative_Abundance(%). The rationale is that reconstructed full-length sequences are correctly assigned, whereas assigning taxa directly to short Illumina reads produces a high rate of false positives.

## Related tools

- **BBMap** (Maps trimmed metagenomic short reads back onto reconstructed SSU sequences to count assigned reads per sequence)
- **VSEARCH** (Clusters reconstructed SSU sequences at the 97% identity threshold (CLUSTER_ID = 0.97) before abundances are summed per cluster)
- **EMIRGE** (Produces the reconstructed full-length SSU rRNA sequences whose abundances are computed)
- **MetaRib** (Produces the reconstructed full-length SSU rRNA sequences whose abundances are computed)
- **QIIME 2 Sklearn classifier** (Assigns taxonomic lineages with a confidence threshold of 0.7 that are combined with the abundance columns)
- **RiboTaxa** (The pipeline that implements clustering, read mapping, and relative abundance computation end-to-end) — https://github.com/oschakoory/RiboTaxa

## Examples

```
vsearch --cluster_fast metagenome_SSU_sequences.fasta --id 0.97 --centroids clustered_SSU_sequences.fasta --sizeout
```

## Evaluation signals

- SSU_taxonomy_abundance.tsv contains all twelve named columns including Assigned reads and Relative_Abundance(%)
- Relative abundances are percentages in the 0–100 range and sum to ~100 across all reconstructed sequences
- Every cluster row in clustered_SSU_abundance.tsv carries a non-zero summed read count
- The Relative_Abundance(%) column sums to 100 across clusters in clustered_SSU_abundance.tsv
- Every reconstructed sequence has a taxonomy lineage with a Confidence value at or above the 0.7 threshold, and no 16S rRNA gene copy-number correction was applied

## Limitations

- No 16S rRNA gene copy number normalization is applied, so abundances reflect mapped reads rather than cell counts
- A part of the observed diversity may originate from artificial diversity created in silico during full-length sequence reconstruction
- Occasional small indel errors can occur in reconstructed sequences, though they have little effect on taxonomic classification
- High conservation of 16S rRNA sequence means some species cannot be discriminated (e.g. certain Enterobacteriaceae and Lachnospiraceae) and classification may stop at genus level

## Evidence

- [methods] Abundance without copy-number correction: [evidence span withheld — anchor status: not_found]
- [methods] 97% clustering and abundance summing: [evidence span withheld — anchor status: not_found]
