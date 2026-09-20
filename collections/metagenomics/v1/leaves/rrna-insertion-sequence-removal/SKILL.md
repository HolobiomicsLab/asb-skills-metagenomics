---
name: rrna-insertion-sequence-removal
description: Use when curating 16S rRNA genes that contain large insertion sequences
  (≥10 bp, up to 2,004 bp in this dataset) before multiple sequence alignment or phylogenetic
  tree building.
when_to_use_negative:
- When the goal is to characterize the insertion-encoded proteins or introns themselves
  — those ORFs are analyzed separately (e.g. Pfam searches with ≥90% ORF-overlap-with-insertion
  criterion), not removed
- When sequences are non-rRNA or already insertion-free and pre-aligned; stripping
  ≥10 bp insertions is specific to intron-laden rRNA gene curation
- When working with bacterial rRNA genes outside CPR, where such insertions are rare
  ('their rarity in bacteria outside the CPR')
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0652
tools:
- name: SSUAlign
  role: Structural alignment of insertion-free 16S rRNA sequences; classifies sequences
    into bacteria/archaea/eukarya and produces per-domain Stockholm alignments
- name: Infernal
  role: Underlying covariance-model engine used for Rfam rRNA model searching (cmscan)
    during insertion detection and SSUAlign alignment
- name: RAxML
  role: Maximum-likelihood tree inference (GTRCAT model, 100 bootstrap resamplings)
    on the insertion-removed 1,582 bp alignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:0f877410928ddaa3a948e730afdc7bb4e9cacbfcda2f369fe1bc39551062d2dd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal
  - RAxML
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0652
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rrna-insertion-sequence-removal

## Summary

Removes intervening insertion sequences (≥10 bp, putative self-splicing introns) from 16S rRNA gene sequences prior to structural alignment and phylogenetic inference. In CPR bacteria these insertions are common (31% of 16S genes) and, if left in place, disrupt rRNA alignment so a reliable phylogeny cannot be established.

## When to use

Use when curating 16S rRNA genes that contain large insertion sequences (≥10 bp, up to 2,004 bp in this dataset) before multiple sequence alignment or phylogenetic tree building.

## When NOT to use

- When the goal is to characterize the insertion-encoded proteins or introns themselves — those ORFs are analyzed separately (e.g. Pfam searches with ≥90% ORF-overlap-with-insertion criterion), not removed
- When sequences are non-rRNA or already insertion-free and pre-aligned; stripping ≥10 bp insertions is specific to intron-laden rRNA gene curation
- When working with bacterial rRNA genes outside CPR, where such insertions are rare ('their rarity in bacteria outside the CPR')

## Inputs

- Curated 16S rRNA gene sequences with annotated intervening insertion sequences of length ≥10 bp (assembled and curated 16S rRNA genes ≥800 bp)
- Rfam/Comparative RNA Web reference rRNA models for SSUAlign structural alignment
- Archaeal 16S rRNA reference sequences (for outgroup rooting)

## Outputs

- Insertion-free 16S rRNA sequences ready for multiple sequence alignment
- Bacterial structural alignment in Stockholm format, converted to FASTA with insert columns removed (1,582 bp alignment in this study)
- Filtered set of sequences with ≥800 bp aligned positions for phylogenetic analysis
- Newick-format maximum-likelihood tree (RAxML GTRCAT, 100 bootstraps) supporting CPR monophyly

## How to apply

Strip all intervening insertion sequences of length ≥10 bp identified during rRNA gene curation (max 2,004 bp, mean 519 bp, s.d. 372 bp in this study) from each 16S rRNA sequence, since insertions must be removed before alignment and tree inference. Align the insertion-free sequences with SSUAlign (built on Infernal), which classifies sequences as bacteria, archaea, or eukarya and generates per-domain structural alignments; convert the bacterial Stockholm alignment to FASTA and remove alignment insert columns to yield a 1,582 bp alignment. Retain only sequences with ≥800 bp of aligned sequence, and concatenate archaeal reference sequences (aligned to the SSUAlign bacterial 16S rRNA model) as the outgroup root. Infer the maximum-likelihood tree with RAxML under the GTRCAT model with 100 bootstrap resamplings; insertion removal followed by structural alignment was explicitly 'critical to establishing a reliable phylogeny' in which CPR is recovered as monophyletic.

## Related tools

- **SSUAlign** (Structural alignment of insertion-free 16S rRNA sequences; classifies sequences into bacteria/archaea/eukarya and produces per-domain Stockholm alignments)
- **Infernal** (Underlying covariance-model engine used for Rfam rRNA model searching (cmscan) during insertion detection and SSUAlign alignment)
- **RAxML** (Maximum-likelihood tree inference (GTRCAT model, 100 bootstrap resamplings) on the insertion-removed 1,582 bp alignment)

## Examples

```
cmscan Rfam.cm 16S_rRNA_curated.fa > rfam_hits.tblout
```

## Evaluation signals

- Resulting bacterial alignment has ~1,582 bp of aligned positions after removal of alignment insert columns
- Only sequences contributing ≥800 bp of aligned sequence are retained for phylogeny
- Resulting tree recovers CPR (Parcubacteria, Microgenomates, WWE3, Berkelbacteria, Saccharibacteria, WS6, Peregrinibacteria, Kazan, CPR1-3) as a monophyletic group, consistent with Supplementary Fig. 1
- Removed insertions correspond to putative self-splicing introns: metatranscriptomic reads mapped to assembled 16S rRNA genes should show insertions are not retained in transcribed RNAs
- No removed segment shorter than the 10 bp insertion threshold is deleted from the pre-alignment sequences

## Limitations

- Insertions are common in CPR 16S rRNA genes (31% carried a ≥10 bp insertion) but rare in other bacteria, so this step adds little outside CPR lineages
- Divergent 16S rRNA sequences in CPR mean 50–100% of organisms from specific phyla evade detection in typical cultivation-independent surveys regardless of insertion removal
- abawaca and related software were under development with limited support at publication time, affecting the upstream binning/curation pipeline feeding this step

## Evidence

- [methods] Rfam search: [evidence span withheld — anchor status: not_found]
