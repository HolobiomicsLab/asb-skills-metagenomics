---
name: rrna-insertion-prevalence-comparison
description: Use when surveying bacterial 16S rRNA gene collections (e.g., SILVA 115)
  for large interhelical insertions, or when testing whether such insertions are CPR-restricted
  versus broadly distributed in bacteria.
when_to_use_negative:
- When the goal is phylogenetic placement of 16S sequences — introns/insertions ≥10
  bp should first be removed rather than quantified before alignment and phylogeny
- When testing whether insertions are retained in transcripts — that requires mapping
  metatranscriptomic (not genomic) reads to the assembled 16S rRNA genes
- When 16S records cannot be confidently identified by the SSUAlign/infernal structural
  model (score below the T 1 threshold) — gap-based insertion calls are unreliable
  on non-16S sequence
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0659
tools:
- name: Infernal (cmsearch)
  role: Searches SILVA records against the 16S rRNA covariance model in hmmonly mode
    with score threshold T 1 to identify 16S rRNA genes; large gaps in the alignment
    mark insertion boundaries
- name: SSUAlign
  role: Provides the manually curated 16S rRNA structural alignment/seed alignment
    used to build the sequence model for cmsearch
- name: SILVA version 115
  role: The non-redundant bacterial 16S rRNA reference database whose insertion prevalence
    is quantified for the comparison against CPR
- name: Rfam (cmscan)
  role: Classifies insertion sequences (rRNA-insertion overlap ≥25% considered positive)
    to associate insertions with known RNA families
- name: Pfam
  role: Searches insertion-encoded ORFs to associate encoded proteins with known protein
    families (≥90% ORF overlap with insertion criterion)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:3f6aa92d1851cd108e862446f4c6fd596b5f409a79a150a41224707932ee0591
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal (cmsearch)
  - Pfam
  - Rfam (cmscan)
  - SILVA version 115
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rrna-insertion-prevalence-comparison

## Summary

Quantifies the prevalence and length distribution of large (≥10 bp) insertions in 16S rRNA genes by aligning sequences (e.g., non-redundant SILVA version 115) to a curated 16S rRNA covariance model with Infernal cmsearch, and compares the rate outside the candidate phyla radiation (CPR) against the 31% large-insertion rate observed within CPR organisms.

## When to use

Use when surveying bacterial 16S rRNA gene collections (e.g., SILVA 115) for large interhelical insertions, or when testing whether such insertions are CPR-restricted versus broadly distributed in bacteria.

## When NOT to use

- When the goal is phylogenetic placement of 16S sequences — introns/insertions ≥10 bp should first be removed rather than quantified before alignment and phylogeny
- When testing whether insertions are retained in transcripts — that requires mapping metatranscriptomic (not genomic) reads to the assembled 16S rRNA genes
- When 16S records cannot be confidently identified by the SSUAlign/infernal structural model (score below the T 1 threshold) — gap-based insertion calls are unreliable on non-16S sequence

## Inputs

- Non-redundant SILVA version 115 bacterial 16S rRNA gene sequences (FASTA)
- Manually curated 16S rRNA structural alignment / covariance model from SSUAlign (built from the Comparative RNA Web seed alignment)
- Assembled and curated 16S rRNA gene sequences from study metagenome assemblies (≥800 bp aligned sequence)
- Insertion sequences for family assignment (searched against Rfam via cmscan and Pfam)

## Outputs

- Prevalence (fraction) of 16S rRNA genes carrying ≥10 bp insertions, stratified CPR vs. non-CPR
- Insertion length distribution (max, mean, s.d.), e.g. max 2,004 bp, mean 519 bp, s.d. 372 bp in CPR
- Per-insertion lineage/family associations (Rfam/Pfam hits)
- Summary table analogous to Supplementary Table 6 / Extended Data Fig. 4

## How to apply

Download the non-redundant SILVA version 115 bacterial 16S rRNA gene sequences and identify 16S records by searching them against the manually curated structural alignment of the 16S rRNA provided with SSUAlign, using Infernal cmsearch in hmmonly mode with a score threshold of T 1, mirroring the approach used for the study's own 1,543 assembled 16S rRNA genes ≥800 bp. Detect insertion boundaries from large gaps (≥10 bp) between each SILVA sequence and the 16S rRNA covariance model alignment, applying the same gap-based boundary detection used for the Rifle metagenome assemblies. For each detected insertion, quantify its length (CPR insertions ranged up to 2,004 bp with mean 519 bp, s.d. 372 bp) and record the lineage/family association of the insertion sequence (e.g., via Rfam cmscan and Pfam searches with ≥25% rRNA-insertion overlap for positive hits). Compute the prevalence of ≥10 bp insertions across the bacterial 16S rRNA genes, then compare against the 31% rate in CPR 16S rRNA genes to confirm that insertions are rare outside the CPR. Consistent with this, all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis, since retained insertions distort 16S-based phylogenies.

## Related tools

- **Infernal (cmsearch)** (Searches SILVA records against the 16S rRNA covariance model in hmmonly mode with score threshold T 1 to identify 16S rRNA genes; large gaps in the alignment mark insertion boundaries)
- **SSUAlign** (Provides the manually curated 16S rRNA structural alignment/seed alignment used to build the sequence model for cmsearch)
- **SILVA version 115** (The non-redundant bacterial 16S rRNA reference database whose insertion prevalence is quantified for the comparison against CPR)
- **Rfam (cmscan)** (Classifies insertion sequences (rRNA-insertion overlap ≥25% considered positive) to associate insertions with known RNA families)
- **Pfam** (Searches insertion-encoded ORFs to associate encoded proteins with known protein families (≥90% ORF overlap with insertion criterion))

## Examples

```
cmsearch --hmmonly -T 1 ssu_16S.cm silva115_bacterial.fasta > silva115_16S_hits.tblout
```

## Evaluation signals

- The computed SILVA-wide prevalence of ≥10 bp insertions should be far below the 31% rate reported in CPR 16S rRNA genes, confirming rarity outside the CPR
- Insertion lengths from the SILVA survey should reproduce the reported distribution (maximum 2,004 bp; mean 519 bp; s.d. 372 bp)
- Every insertion boundary should correspond to a gap of ≥10 bp in the sequence-to-covariance-model alignment
- Results should be consistent with the study's survey of its own 1,543 assembled 16S rRNA genes ≥800 bp (Extended Data Fig. 4 and Supplementary Table 6)
- Non-16S SILVA records should be excluded via the cmsearch hmmonly T 1 score threshold before any gap-based insertion detection

## Limitations

- Insertions are not retained in transcribed RNAs — mapping metatranscriptomic sequences showed insertions are probably rapidly degraded introns, so prevalence in genomes overstates prevalence in ribosomal RNA
- Divergent CPR 16S rRNA gene sequences mean 50–100% of organisms from specific phyla evade typical cultivation-independent surveys, so database-based prevalence estimates under-represent highly divergent lineages
- Primers such as 515F/806R would probably miss ~20% of bacteria detected in this study, so amplicon-derived databases cannot be used as an unbiased insertion survey

## Evidence

- [methods] SILVA version 115 analysed with the same methods: [evidence span withheld — anchor status: not_found]
