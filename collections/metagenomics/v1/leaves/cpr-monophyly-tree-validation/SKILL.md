---
name: cpr-monophyly-tree-validation
description: Use when bacterial 16S rRNA gene sequences contain large insertions (≥10
  bp) that would corrupt multiple sequence alignment, and the goal is a defensible
  phylogeny to test whether CPR lineages form a monophyletic group.
when_to_use_negative:
- The input alignment is a ribosomal-protein (rp16) phylogeny — that workflow uses
  a ≥1,000 aligned-residue filter on protein SCGs, not intron-stripped 16S sequences
- 16S rRNA genes have no large insertions (rare outside CPR, per SILVA v115 screening)
  and a standard nucleotide alignment without intron removal is appropriate
- Sequences are short-read amplicons that cannot provide ≥800 bp of aligned 16S sequence
  for phylogeny
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0092
tools:
- name: SSUAlign
  role: Performs structural multiple alignment of intron-free 16S rRNA sequences,
    classifying them by domain and producing the bacterial Stockholm alignment
- name: Infernal
  role: Supports Rfam/cmscan-based identification of rRNA insertions to be removed
    before alignment
- name: RAxML
  role: Infers the maximum-likelihood tree (GTRCAT model, 100 bootstrap resamplings)
    and writes the Newick output used for the monophyly check
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:2f4dd03c62c5858206a8b1e7279978bd46fbbc2d4bafb5168a87823377ac8122
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Infernal
  - RAxML
  - SSUAlign
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cpr-monophyly-tree-validation

## Summary

Builds an intron-corrected 16S rRNA maximum-likelihood phylogeny in which the candidate phyla radiation (CPR) can be tested for monophyly: strip rRNA insertions ≥10 bp, structurally realign with SSUAlign, and infer a rooted RAxML tree.

## When to use

Use when bacterial 16S rRNA gene sequences contain large insertions (≥10 bp) that would corrupt multiple sequence alignment, and the goal is a defensible phylogeny to test whether CPR lineages form a monophyletic group.

## When NOT to use

- The input alignment is a ribosomal-protein (rp16) phylogeny — that workflow uses a ≥1,000 aligned-residue filter on protein SCGs, not intron-stripped 16S sequences
- 16S rRNA genes have no large insertions (rare outside CPR, per SILVA v115 screening) and a standard nucleotide alignment without intron removal is appropriate
- Sequences are short-read amplicons that cannot provide ≥800 bp of aligned 16S sequence for phylogeny

## Inputs

- Curated bacterial 16S rRNA gene sequences with annotated intron/insertion coordinates (insertions ≥10 bp flagged during rRNA gene curation)
- Curated archaeal reference 16S rRNA sequences for outgroup rooting
- SSUAlign bacterial 16S rRNA model (built from the Comparative RNA Web seed alignment) with accompanying Infernal/cmsearch machinery

## Outputs

- Insertion-free 16S rRNA FASTA sequences
- SSUAlign per-domain Stockholm structural alignments, converted to a 1,582 bp FASTA bacterial alignment with insert columns removed
- Rooted Newick maximum-likelihood tree with 100-bootstrap support values, from which CPR monophyly is inspected

## How to apply

First remove all intervening insertion sequences of length ≥10 bp identified during rRNA gene curation, since insertions (max 2,004 bp, mean 519 bp) must be removed before multiple sequence alignment. Align the insertion-free sequences with SSUAlign (which uses Infernal covariance models to classify sequences as bacteria, archaea, or eukarya and produce per-domain structural alignments); convert the bacterial Stockholm alignment to FASTA and drop alignment insert columns to yield a 1,582 bp alignment. Keep only sequences with ≥800 bp of aligned sequence, and concatenate selected archaeal reference sequences (aligned against the SSUAlign bacterial 16S rRNA model) to serve as the phylogenetic root. Infer the maximum-likelihood tree with RAxML under the GTRCAT model with 100 bootstrap resamplings and export in Newick format. Correct application is judged by whether the CPR clade (Parcubacteria, Microgenomates, WWE3, Berkelbacteria, Saccharibacteria, WS6, Peregrinibacteria, Kazan, CPR1-3) is recovered as monophyletic, consistent with Supplementary Fig. 1.

## Related tools

- **SSUAlign** (Performs structural multiple alignment of intron-free 16S rRNA sequences, classifying them by domain and producing the bacterial Stockholm alignment)
- **Infernal** (Supports Rfam/cmscan-based identification of rRNA insertions to be removed before alignment)
- **RAxML** (Infers the maximum-likelihood tree (GTRCAT model, 100 bootstrap resamplings) and writes the Newick output used for the monophyly check)

## Examples

```
raxmlHPC -f a -m GTRCAT -p 12345 -x 12345 -N 100 -s bacterial_16S_1582bp.fasta -n cpr_monophyly
```

## Evaluation signals

- Final bacterial alignment is exactly 1,582 bp after FASTA conversion and removal of alignment insert columns
- Every retained sequence contributes ≥800 bp of aligned 16S sequence to the phylogeny
- All insertions ≥10 bp are absent from the sequences fed to SSUAlign
- Archaeal reference sequences group as the root, outside the bacterial radiation
- CPR clade containing Parcubacteria, Microgenomates, WWE3, Berkelbacteria, Saccharibacteria, WS6, Peregrinibacteria, Kazan, and CPR1-3 is monophyletic, with 100-bootstrap support values on the Newick tree

## Limitations

- Large rRNA insertions (31% of CPR 16S genes had one; ≥10 bp, max 2,004 bp) must be curated first; unremoved insertions would misalign the structural alignment
- Insertions are self-splicing introns and are not retained in transcribed RNAs (metatranscriptomic mapping showed insertions are rapidly degraded), so RNA-derived 16S reads alone will not reveal them
- Divergent CPR 16S rRNA sequences mean 50-100% of organisms from specific CPR phyla would evade detection in typical amplicon surveys, so sequence input matters

## Evidence
