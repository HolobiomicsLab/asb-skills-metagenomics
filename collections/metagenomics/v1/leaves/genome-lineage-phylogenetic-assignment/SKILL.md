---
name: genome-lineage-phylogenetic-assignment
description: Use when genome bins (draft or complete, e.g., CPR/TM6) must be assigned
  to lineages via 16S rRNA gene or rp16 concatenated ribosomal protein phylogeny,
  especially before building lineage-level presence/absence matrices.
when_to_use_negative:
- The genomes already have trusted lineage/taxonomic labels and the goal is only ribosomal
  protein profiling
- 16S rRNA genes cannot be recovered or are too short/divergent for alignment (<800
  bp aligned bases) and no rp16 data are available
- The analysis targets functional annotation rather than lineage-level phylogenetic
  placement
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: fix_assembly_errors
  role: Curation of assembled scaffolds/16S rRNA genes to eliminate assembly errors
    before phylogenetic analysis
  repo: https://github.com/christophertbrown/fix_assembly_errors
- name: USEARCH
  role: Clustering of assembled 16S rRNA gene sequences at 97% identity to remove
    redundancy prior to phylogeny
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature14486
    title: Unusual biology across a group comprising more than 15% of domain Bacteria
  source_content_hash: sha256:2a8f1d18218457871323e4cac360327cf956ecff357da92bab6cfd5d130ca825
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/christophertbrown/fix_assembly_errors
  tools:
  - HMMER hmmscan
  - USEARCH
  - fix_assembly_errors
  - gg Kbase
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genome-lineage-phylogenetic-assignment

## Summary

Assigns metagenome-assembled genome bins (e.g., CPR and TM6 bacteria) to lineages using 16S rRNA gene phylogeny or concatenated ribosomal protein (rp16) phylogeny, with intron removal and strict quality filtering so that presence/absence analyses (such as ribosomal protein L30 loss) can be reported per lineage.

## When to use

Use when genome bins (draft or complete, e.g., CPR/TM6) must be assigned to lineages via 16S rRNA gene or rp16 concatenated ribosomal protein phylogeny, especially before building lineage-level presence/absence matrices.

## When NOT to use

- The genomes already have trusted lineage/taxonomic labels and the goal is only ribosomal protein profiling
- 16S rRNA genes cannot be recovered or are too short/divergent for alignment (<800 bp aligned bases) and no rp16 data are available
- The analysis targets functional annotation rather than lineage-level phylogenetic placement

## Inputs

- Assembled metagenomic scaffolds from genome bins (CPR/TM6 draft and complete genomes)
- Assembled 16S rRNA gene sequences (>=800 bp) with intronic insertions
- Concatenated ribosomal protein (rp16) alignments (>=1,000 aligned residues)
- Reference 16S databases (e.g., nonredundant SILVA version 115)

## Outputs

- Lineage assignment for each genome bin (e.g., CPR phylum-level lineages, TM6)
- Phylogenetic tree with CPR rooted using Chloroflexi and TM6 included
- Lineage labels feeding binary ribosomal protein presence/absence matrices (rpL30, rpL9, rpL1, Der)

## How to apply

Assemble and curate 16S rRNA genes (1,543 bacterial 16S rRNA genes >=800 bp in the reference study, curated to eliminate assembly errors), then remove all insertions >=10 bp from the sequences since CPR organisms often carry self-splicing introns in rRNA genes. Build a sequence-only structural alignment and cluster sequences at 97% identity (e.g., with USEARCH) to remove redundancy, keeping only sequences with >=800 bp of aligned 16S sequence for phylogeny; for the rp16 concatenated ribosomal protein tree, keep sequences with >=1,000 aligned residues. Infer the tree with the CPR rooted using Chloroflexi and TM6 included, then map each genome bin to its lineage via the tree. Validate lineage assignment by checking genome quality gates (>=75% single-copy genes with <=1.125 copies for the ribosomal protein matrix) and, where 16S is divergent, cross-check against databases such as nonredundant SILVA version 115 analyzed with the same methods.

## Related tools

- **fix_assembly_errors** (Curation of assembled scaffolds/16S rRNA genes to eliminate assembly errors before phylogenetic analysis) — https://github.com/christophertbrown/fix_assembly_errors
- **USEARCH** (Clustering of assembled 16S rRNA gene sequences at 97% identity to remove redundancy prior to phylogeny)

## Evaluation signals

- Every genome bin carries a lineage label consistent with its placement in the 16S/rp16 tree with CPR rooted on Chloroflexi and TM6 included
- All 16S sequences used for phylogeny have >=800 bp of aligned sequence (>=1,000 aligned residues for rp16)
- Insertions >=10 bp are removed from 16S rRNA genes before alignment (verified by re-checking for retained introns)
- 16S sequences clustered at 97% identity so redundant copies do not inflate lineage counts
- Lineage assignments agree across independent 16S and rp16 trees

## Limitations

- Divergent 16S rRNA gene sequences mean 50-100% of organisms from some CPR phyla evade typical cultivation-independent surveys, complicating reference-based assignment
- 31% of CPR 16S rRNA genes carry large (>=10 bp) insertion sequences that must be removed or they will corrupt the alignment
- Lineages lacking complete genomes (e.g., WS6, median completeness 91%) have residual uncertainty, quantified as (1 - 0.91)^16 ~ 2e-7 for gene absence

## Evidence
