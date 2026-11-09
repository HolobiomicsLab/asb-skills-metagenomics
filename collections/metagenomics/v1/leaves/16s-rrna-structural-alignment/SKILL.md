---
name: 16s-rrna-structural-alignment
description: Use when assembled 16S rRNA genes from metagenomes contain large insertion
  sequences (≥10 bp, e.g. CPR self-splicing introns up to 2,004 bp) that would corrupt
  a naive sequence alignment, and a reliable domain-level or phyla-level phylogeny
  is the goal.
when_to_use_negative:
- If the 16S rRNA genes contain no large insertion sequences (insertions are rare
  in bacteria outside the CPR per the SILVA database search) — a standard sequence
  alignment suffices.
- If the goal is amplicon-survey detectability assessment rather than phylogeny —
  use the primer-binding analysis (515F/806R) workflow instead.
- If sequences are shorter than 800 bp of aligned sequence, they will be excluded
  and cannot support this phylogeny.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0080
tools:
- name: SSUAlign
  role: Classifies insertion-free 16S rRNA sequences as bacteria/archaea/eukarya and
    produces the per-domain structural alignments used for phylogeny
- name: Infernal
  role: cmscan against Rfam during rRNA gene curation to identify insertion sequences
    to be removed before alignment
- name: RAxML
  role: Infers the maximum-likelihood 16S rRNA tree with the GTRCAT model and 100
    bootstrap resamplings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:36cc1682a2d144fca4a3ea0adaa2d5461775fe02dc120d7752214fb49831ec6f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal
  - RAxML
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# 16s-rrna-structural-alignment

## Summary

Removes large (≥10 bp) intron insertions from 16S rRNA gene sequences, then performs structural (covariance-model-based) alignment with SSUAlign to produce an insertion-free bacterial alignment suitable for maximum-likelihood phylogeny in which the candidate phyla radiation (CPR) is recovered monophyletic.

## When to use

Use when assembled 16S rRNA genes from metagenomes contain large insertion sequences (≥10 bp, e.g. CPR self-splicing introns up to 2,004 bp) that would corrupt a naive sequence alignment, and a reliable domain-level or phyla-level phylogeny is the goal.

## When NOT to use

- If the 16S rRNA genes contain no large insertion sequences (insertions are rare in bacteria outside the CPR per the SILVA database search) — a standard sequence alignment suffices.
- If the goal is amplicon-survey detectability assessment rather than phylogeny — use the primer-binding analysis (515F/806R) workflow instead.
- If sequences are shorter than 800 bp of aligned sequence, they will be excluded and cannot support this phylogeny.

## Inputs

- Curated 16S rRNA gene sequences (≥800 bp assembled and curated, intron/insertion coordinates annotated)
- List of insertion coordinates (insertions ≥10 bp identified during rRNA gene curation, e.g. via Rfam/cmscan with ≥25% rRNA-insertion overlap)
- Archaeal 16S rRNA reference sequences for outgroup rooting

## Outputs

- Insertion-free 16S rRNA sequence set (FASTA)
- Per-domain SSUAlign structural alignments (Stockholm), bacterial alignment converted to FASTA with insert columns removed (1,582 bp)
- Newick-format maximum-likelihood tree (RAxML, GTRCAT, 100 bootstrap resamplings) with archaeal root

## How to apply

First strip all intervening sequences ≥10 bp previously identified during rRNA gene curation, since insertions (present in 31% of CPR 16S genes in this study) must be removed before multiple sequence alignment. Align the insertion-free sequences with SSUAlign, which classifies each sequence as bacteria, archaea, or eukarya and generates separate per-domain structural alignments; convert the bacterial Stockholm alignment to FASTA and remove alignment insert columns to yield a 1,582 bp alignment. Retain only sequences with ≥800 bp of aligned sequence, and concatenate selected archaeal reference sequences (aligned to the SSUAlign bacterial 16S rRNA model) to serve as the phylogenetic root. Infer the maximum-likelihood tree with RAxML under the GTRCAT model with 100 bootstrap resamplings, writing the tree in Newick format. Structural alignment is used rather than plain sequence alignment because divergent CPR 16S sequences plus retained introns would otherwise misplace lineages.

## Related tools

- **SSUAlign** (Classifies insertion-free 16S rRNA sequences as bacteria/archaea/eukarya and produces the per-domain structural alignments used for phylogeny)
- **Infernal** (cmscan against Rfam during rRNA gene curation to identify insertion sequences to be removed before alignment)
- **RAxML** (Infers the maximum-likelihood 16S rRNA tree with the GTRCAT model and 100 bootstrap resamplings)

## Examples

```
raxmlHPC -m GTRCAT -p 12345 -N 100 -s bacterial_16S_insertion_free_1582bp.fasta -n cpr_16S_raxml
```

## Evaluation signals

- The bacterial SSUAlign alignment, after removing insert columns, is exactly 1,582 bp long.
- Every sequence included in the phylogeny has ≥800 bp of aligned sequence.
- The inferred RAxML tree recovers the CPR (Parcubacteria, Microgenomates, WWE3, Berkelbacteria, Saccharibacteria, WS6, Peregrinibacteria, Kazan, CPR1-3) as a monophyletic group, consistent with Supplementary Fig. 1.
- Archaeal reference sequences fall outside (root) the bacterial clade as expected.
- No retained sequence contains an insertion of ≥10 bp within the aligned 16S region.

## Limitations

- Large insertions are rare in bacteria outside the CPR (checked against SILVA version 115), so this insertion-removal step is mainly critical for CPR-dominated datasets.
- Divergent CPR 16S rRNA genes evade detection in typical cultivation-independent surveys (50–100% of organisms in some phyla); the resulting tree only covers organisms recovered by metagenome assembly, not the full community.
- Insertions are self-splicing and not retained in transcribed RNAs, so sequences mapped from metatranscriptomes will not carry the intron and should not be used to infer insertion coordinates.

## Evidence

- [abstract] Removal of introns from 16S rRNA gene sequences, followed by structural alignment 17 , was critical to establishing a reliable phylogeny.: "Removal of introns from 16S rRNA gene sequences, followed by structural alignment 17 , was critical to establishing a reliable phylogeny."
- [methods] insertions >=10 bp removed before MSA: "all insertions ≥10 bp were removed before multiple sequence alignment and phylogenetic analysis of 16S rRNA gene sequences"
- [methods] 800 bp aligned filter: "All sequences with ≥800 bp of aligned sequence were used for phylogenetic analysis"
- [abstract] 31% insertion frequency: "31% of 16S rRNA genes encoded a large (≥10 bp) insertion sequence (maximum 2,004 bp; mean 519 bp; standard deviation (s.d.) 372 bp"
