---
name: silva-database-insertion-screening
description: Use when you need to quantify the prevalence or distribution of large
  (≥10 bp) insertions/interrupts in 16S rRNA gene sequences across a large reference
  database, or to benchmark an observed insertion rate (e.g. the 31% CPR rate) against
  database-wide background.
when_to_use_negative:
- 'The goal is small indels or sequencing errors: the screen is designed for large
  (≥10 bp) gaps, and insertions ≥10 bp are normally removed before phylogenetic alignment'
- The sequences are already intron-free, curated 16S rRNA genes intended for structural
  alignment/phylogeny — remove insertions and align instead of re-screening
- 'Assessing whether insertions are retained in transcripts: that requires mapping
  metatranscriptomic sequences to assembled 16S rRNA genes, not SILVA screening'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3168
tools:
- name: Infernal (cmsearch)
  role: Identifies 16S rRNA gene sequences among SILVA records by covariance-model
    search in hmmonly mode with score threshold T 1
- name: SSUAlign
  role: Provides the manually curated structural alignment of the 16S rRNA used as
    the reference model for alignment and gap-based insertion boundary detection
- name: SILVA version 115
  role: Non-redundant bacterial 16S rRNA reference database screened for large insertions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:0f9d201bc770a74a7d645fae5fef3a75f209ae8eca53ff2de4ddabf4484d68a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal (cmsearch)
  - SILVA version 115
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# SILVA database insertion screening

## Summary

A database-scale survey method that screens bacterial 16S rRNA gene sequences from non-redundant SILVA version 115 for large (≥10 bp) insertions, using covariance-model alignment gap detection to quantify how rare CPR-style intronic insertions are outside the candidate phyla radiation.

## When to use

Use when you need to quantify the prevalence or distribution of large (≥10 bp) insertions/interrupts in 16S rRNA gene sequences across a large reference database, or to benchmark an observed insertion rate (e.g. the 31% CPR rate) against database-wide background.

## When NOT to use

- The goal is small indels or sequencing errors: the screen is designed for large (≥10 bp) gaps, and insertions ≥10 bp are normally removed before phylogenetic alignment
- The sequences are already intron-free, curated 16S rRNA genes intended for structural alignment/phylogeny — remove insertions and align instead of re-screening
- Assessing whether insertions are retained in transcripts: that requires mapping metatranscriptomic sequences to assembled 16S rRNA genes, not SILVA screening

## Inputs

- Non-redundant SILVA version 115 bacterial 16S rRNA gene sequences (FASTA)
- Manually curated 16S rRNA structural alignment seed (Comparative RNA Web seed alignment, via SSUAlign)
- 16S rRNA covariance model (Rfam/SSUAlign CM)
- Optionally: study-assembled and curated 16S rRNA gene sequences (≥800 bp, introns removed) for comparison

## Outputs

- Per-sequence insertion calls with boundary coordinates derived from ≥10 bp alignment gaps
- Insertion length distribution (e.g. maximum 2,004 bp; mean 519 bp; s.d. 372 bp) and per-lineage prevalence
- SILVA-wide prevalence of ≥10 bp insertions inside vs. outside the CPR, for comparison against the 31% CPR rate

## How to apply

Download the non-redundant SILVA version 115 bacterial 16S rRNA gene sequences and identify 16S rRNA records by searching them with Infernal cmsearch in hmmonly mode (score threshold T 1) against a manually curated structural alignment of the 16S rRNA (the sequence-only model built from the Comparative RNA Web seed alignment, used via SSUAlign). Detect insertion boundaries from large gaps (≥10 bp) between each SILVA sequence and the 16S rRNA covariance model alignment, mirroring the gap-based boundary detection applied to the Rifle metagenome assemblies. For each insertion, record its length (SILVA-wide maximum was 2,004 bp; mean 519 bp; s.d. 372 bp in the study set) and lineage/family association, then compute the prevalence of ≥10 bp insertions across all bacterial 16S rRNA genes. Judge correctness by reproducing the rarity of insertions outside the CPR, contrasting with the 31% of CPR 16S rRNA genes encoding a large insertion, and by matching the length distribution reported in Extended Data Fig. 4 and Supplementary Table 6.

## Related tools

- **Infernal (cmsearch)** (Identifies 16S rRNA gene sequences among SILVA records by covariance-model search in hmmonly mode with score threshold T 1)
- **SSUAlign** (Provides the manually curated structural alignment of the 16S rRNA used as the reference model for alignment and gap-based insertion boundary detection)
- **SILVA version 115** (Non-redundant bacterial 16S rRNA reference database screened for large insertions)

## Examples

```
cmsearch --hmmonly -T 1 SSU_bacteria.cm silva115_bacterial_16S.fasta > silva115_16S_hits.tblout
```

## Evaluation signals

- Detected insertion boundaries coincide with gaps of ≥10 bp between each SILVA sequence and the 16S rRNA covariance model alignment
- Computed SILVA-wide insertion prevalence is consistent with the finding that such insertions are rare in bacteria outside the CPR
- Insertion length statistics fall in the reported ranges (maximum 2,004 bp; mean 519 bp; s.d. 372 bp) and match Supplementary Table 6 / Extended Data Fig. 4
- A ~31% large-insertion rate is recovered when restricting to CPR 16S rRNA genes, serving as a positive control
- All SILVA records pass a 16S rRNA identity check at the cmsearch hmmonly score threshold of T 1 before insertion calling

## Limitations

- Only large (≥10 bp) insertions are detected; smaller indels are invisible to the gap-based boundary detection and are removed before phylogenetic analysis
- Divergent CPR 16S rRNA gene sequences may evade detection entirely — 50–100% of organisms from specific CPR phyla would be missed in typical surveys — so SILVA may under-represent insertion-bearing lineages
- Insertions are probably self-splicing introns not retained in transcribed RNAs (they are probably rapidly degraded), so RNA-based surveys will not recover them

## Evidence

- [methods] version 115 of nonredundant SILVA was analysed using the same methods: [evidence span withheld — anchor status: not_found]
