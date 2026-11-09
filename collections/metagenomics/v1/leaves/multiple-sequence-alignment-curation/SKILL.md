---
name: multiple-sequence-alignment-curation
description: Use when rRNA gene sequences contain large insertion sequences (≥10 bp)
  that must be stripped prior to multiple sequence alignment, or when building a phylogeny
  from divergent 16S sequences (e.g. CPR) where naive alignment would be unreliable.
when_to_use_negative:
- Sequences lack rRNA introns/insertions — insertion stripping is unnecessary and
  the standard alignment suffices
- The alignment is for protein-coding genes rather than structurally modeled rRNA
  genes — SSUAlign per-domain structural models do not apply
- Sequences are shorter than the ≥800 bp aligned-residue threshold and are intended
  for phylogenetic placement rather than being excluded
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: SSUAlign
  role: classifies sequences as bacteria/archaea/eukarya and generates per-domain
    structural alignments of insertion-free 16S rRNA sequences
- name: Infernal
  role: searches full rRNA sequences against Rfam via cmscan to support rRNA gene
    curation and insertion identification
- name: USEARCH
  role: clusters assembled 16S rRNA genes at 97% sequence identity (cluster_smallmem)
    to remove redundancy before alignment
- name: RAxML
  role: infers the maximum-likelihood tree from the curated alignment using GTRCAT
    and 100 bootstrap resamplings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:3da7f8be94951edb7181704b41c4dba53db1bcee5febc72bfea89999a42609bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal
  - RAxML
  - SSUAlign
  - USEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# multiple-sequence-alignment-curation

## Summary

Curates 16S rRNA gene sequences before phylogenetic inference by removing self-splicing intron insertions (≥10 bp), structurally aligning with SSUAlign, and applying length/redundancy filters so CPR monophyly is not an artifact of insertion sequences.

## When to use

Use when rRNA gene sequences contain large insertion sequences (≥10 bp) that must be stripped prior to multiple sequence alignment, or when building a phylogeny from divergent 16S sequences (e.g. CPR) where naive alignment would be unreliable.

## When NOT to use

- Sequences lack rRNA introns/insertions — insertion stripping is unnecessary and the standard alignment suffices
- The alignment is for protein-coding genes rather than structurally modeled rRNA genes — SSUAlign per-domain structural models do not apply
- Sequences are shorter than the ≥800 bp aligned-residue threshold and are intended for phylogenetic placement rather than being excluded

## Inputs

- curated 16S rRNA gene sequences with identified intron/insertion coordinates (≥10 bp insertions flagged)
- assembled bacterial 16S rRNA genes ≥800 bp (e.g. 1,543 curated genes)
- archaeal reference 16S rRNA sequences for outgroup rooting
- SSUAlign bacterial 16S rRNA structural model (Rfam/Comparative RNA Web seed alignment based)

## Outputs

- insertion-free 16S rRNA sequence set
- 97%-identity-dereplicated 16S sequence clusters (USEARCH cluster_smallmem)
- 1,582 bp bacterial structural alignment in FASTA (converted from SSUAlign Stockholm, insert columns removed)
- maximum-likelihood phylogenetic tree in Newick format with 100 bootstrap replicates (RAxML GTRCAT)

## How to apply

First remove all insertion sequences of length ≥10 bp identified during rRNA gene curation, since insertions must be gone before multiple sequence alignment. Cluster curated 16S sequences at 97% identity with USEARCH (cluster_smallmem) to remove redundancy. Align insertion-free sequences with SSUAlign, which classifies sequences as bacteria, archaea, or eukarya and produces per-domain structural alignments; convert the bacterial Stockholm alignment to FASTA and remove alignment insert columns to yield a 1,582 bp alignment. Retain only sequences with ≥800 bp of aligned sequence, concatenate archaeal reference sequences (aligned to the SSUAlign bacterial 16S rRNA model) as the outgroup root, then infer the maximum-likelihood tree with RAxML under the GTRCAT model with 100 bootstrap resamplings.

## Related tools

- **SSUAlign** (classifies sequences as bacteria/archaea/eukarya and generates per-domain structural alignments of insertion-free 16S rRNA sequences)
- **Infernal** (searches full rRNA sequences against Rfam via cmscan to support rRNA gene curation and insertion identification)
- **USEARCH** (clusters assembled 16S rRNA genes at 97% sequence identity (cluster_smallmem) to remove redundancy before alignment)
- **RAxML** (infers the maximum-likelihood tree from the curated alignment using GTRCAT and 100 bootstrap resamplings)

## Examples

```
raxmlHPC -m GTRCAT -p 12345 -N 100 -s bacterial_16S.intron_free.1582bp.fasta -n cpr_16s
```

## Evaluation signals

- Resulting alignment length equals 1,582 bp after insert-column removal
- All retained sequences have ≥800 bp of aligned sequence
- All insertions ≥10 bp are absent from the sequences fed to SSUAlign
- Inspected tree recovers CPR (Parcubacteria, Microgenomates, WWE3, Berkelbacteria, Saccharibacteria, WS6, Peregrinibacteria, Kazan, CPR1-3) as monophyletic, consistent with Supplementary Fig. 1
- Archaeal references placed as the outgroup root of the bacterial tree

## Limitations

- Insertions are rare in bacteria outside the CPR, so this skill is mainly needed for CPR and related divergent rRNA genes
- Divergent 16S rRNA sequences mean 50–100% of organisms from specific CPR phyla would evade typical cultivation-independent surveys; primer-based checks (515F/806R) should be run separately
- Insertions must be removed before alignment; leaving them in would compromise the multiple sequence alignment and downstream phylogeny

## Evidence
