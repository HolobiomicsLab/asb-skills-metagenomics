---
name: comparative-rna-structural-alignment
description: Use when placing 16S/SSU rRNA sequences (e.g. assembled metagenome contigs
  or SILVA records) into a reference structural alignment, detecting insertions as
  alignment gaps, or preparing intron-stripped rRNA sequences for phylogeny.
when_to_use_negative:
- Aligning protein-coding or non-rRNA sequences — the covariance model and gap-based
  insertion logic are specific to SSU rRNA structure
- When the goal is amplicon primer coverage estimation rather than structural placement
  or insertion detection (use primer binding analysis instead)
- Sequences shorter than the ≥800 bp aligned-residue cutoff used for phylogenetic
  analysis in this study
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: Infernal (cmsearch)
  role: Searches all SILVA 115 records against the 16S rRNA covariance model in hmmonly
    mode with score threshold T 1 to identify 16S rRNA genes
- name: SSUAlign
  role: Provides the manually curated structural alignment of the 16S rRNA used as
    the reference for alignment and gap-based insertion boundary detection
- name: Comparative RNA Web seed alignment
  role: Source of the manually curated seed alignment from which the sequence-only
    covariance model is built
- name: SILVA version 115
  role: Non-redundant bacterial 16S rRNA database searched to quantify insertion rarity
    outside the CPR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:bab616aec7a660d34ac543f5bfaee60b42b1f82f32e235885832704125f6f244
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Comparative RNA Web seed alignment
  - Infernal (cmsearch)
  - SILVA version 115
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# comparative-rna-structural-alignment

## Summary

Align rRNA gene sequences against a manually curated structural (covariance-model) alignment to place them accurately despite divergent or intron-disrupted sequences. In this study it underpins both reliable CPR phylogeny and the survey of ≥10 bp insertions in bacterial 16S rRNA genes.

## When to use

Use when placing 16S/SSU rRNA sequences (e.g. assembled metagenome contigs or SILVA records) into a reference structural alignment, detecting insertions as alignment gaps, or preparing intron-stripped rRNA sequences for phylogeny.

## When NOT to use

- Aligning protein-coding or non-rRNA sequences — the covariance model and gap-based insertion logic are specific to SSU rRNA structure
- When the goal is amplicon primer coverage estimation rather than structural placement or insertion detection (use primer binding analysis instead)
- Sequences shorter than the ≥800 bp aligned-residue cutoff used for phylogenetic analysis in this study

## Inputs

- Non-redundant SILVA version 115 bacterial 16S rRNA gene sequences (FASTA)
- Assembled and curated bacterial 16S rRNA gene sequences ≥800 bp from metagenome assemblies
- Manually curated structural seed alignment of the 16S rRNA (Comparative RNA Web seed alignment, as provided with SSUAlign)

## Outputs

- cmsearch hit table identifying 16S rRNA gene records (T 1 threshold, hmmonly mode)
- Insertion boundary coordinates and lengths (gaps ≥10 bp) per sequence, with lineage/family associations
- Intron-stripped 16S rRNA alignment suitable for phylogenetic reconstruction

## How to apply

Build a sequence-only covariance model from the manually curated Comparative RNA Web seed alignment, then search all input rRNA records with Infernal cmsearch in hmmonly mode using a score threshold of T 1 to identify 16S rRNA genes. Detect insertions as large gaps (≥10 bp) between each query sequence and the 16S rRNA covariance model alignment, recording insertion length and lineage/family association. Before multiple sequence alignment and phylogenetic analysis, remove all insertions ≥10 bp from the rRNA sequences, and restrict phylogenetic analysis to sequences with ≥800 bp of aligned sequence (optionally clustering at 97% identity with USEARCH cluster_smallmem to remove redundancy). Because divergent CPR 16S sequences can evade detection, verify sensitivity against known divergent lineages; insertions detected this way are candidate self-splicing introns and are not retained in transcribed RNAs.

## Related tools

- **Infernal (cmsearch)** (Searches all SILVA 115 records against the 16S rRNA covariance model in hmmonly mode with score threshold T 1 to identify 16S rRNA genes)
- **SSUAlign** (Provides the manually curated structural alignment of the 16S rRNA used as the reference for alignment and gap-based insertion boundary detection)
- **Comparative RNA Web seed alignment** (Source of the manually curated seed alignment from which the sequence-only covariance model is built)
- **SILVA version 115** (Non-redundant bacterial 16S rRNA database searched to quantify insertion rarity outside the CPR)

## Examples

```
cmsearch --hmmonly -T 1 ssu_16S.cm silva115_bacterial.fa > silva115_cmsearch.out
```

## Evaluation signals

- The computed SILVA-wide prevalence of ≥10 bp 16S insertions should reproduce the reported survey (Extended Data Fig. 4 and Supplementary Table 6): rare outside the CPR versus 31% of CPR 16S rRNA genes (maximum 2,004 bp; mean 519 bp; s.d. 372 bp)
- Every detected insertion must correspond to a gap of ≥10 bp between the query sequence and the 16S rRNA covariance model alignment
- Phylogenetic input alignment must contain only sequences with ≥800 bp of aligned sequence and insertions ≥10 bp removed
- Metatranscriptomic reads mapped to assembled 16S rRNA genes should confirm insertions are not retained in transcribed RNAs

## Limitations

- Owing to divergent 16S rRNA gene sequences, 50–100% of organisms sampled from specific CPR phyla would evade detection in typical cultivation-independent surveys, so hmmonly cmsearch at T 1 may still miss highly divergent sequences
- Large insertions must be removed before alignment and phylogeny or they will distort the tree; 31% of CPR 16S rRNA genes encoded such insertions
- The SAR/CPR insertion-rich 16S genes differ from typical bacteria: such insertions are rare in bacteria outside the CPR, so gap-based insertion detection yields few hits on non-CPR SILVA records

## Evidence

- [methods] Identify 16S rRNA gene sequences ... using Infernal cmsearch in hmmonly mode with a score threshold of T 1: [evidence span withheld — anchor status: not_found]
