---
name: multiple-sequence-alignment-of-viral-genes
description: Use when you need to type or classify viral genomes/OTUs from a conserved
  marker gene (e.g., L1) and place them on a phylogeny against reference sequences
  from RefSeq.
when_to_use_negative:
- The viral genomes lack the target marker gene (e.g., non-papillomaviruses without
  an L1 gene) — use whole-genome or different marker approaches instead
- The goal is abundance/taxonomic profiling of metagenomic reads rather than phylogenetic
  placement of assembled genomes — use Kraken2/Bracken with a customized database
  instead
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Predicts open reading frames in meta mode on each papillomavirus genome to
    extract the L1 major capsid protein gene
- name: MAFFT
  role: Aligns the combined VMGC + RefSeq L1 nucleotide sequences
- name: IQ-TREE
  role: Infers the maximum-likelihood phylogeny of the L1 alignment saved as l1_phylogeny.nwk
- name: BLAST
  role: Used for sequence identity comparison supporting taxonomy assignment of L1
    sequences
- name: seqkit
  role: Sequence manipulation/processing of viral FASTA files during gene extraction
- name: GNU parallel
  role: Parallelizes per-genome processing steps across the 627 Papillomaviridae genomes
  repo: https://github.com/RChGO/VMGC/tree/main/Pipelines
- name: NCBI RefSeq
  role: Source of reference papillomavirus L1 sequences for combined alignment and
    phylogeny
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:da4db0433c5e0c470db0d84b6cd4fa7587e9d979ad81dfb8de6732a8641ef7af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - GNU parallel
  - IQ-TREE
  - MAFFT
  - NCBI RefSeq
  - Prodigal
  - seqkit
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multiple-sequence-alignment-of-viral-genes

## Summary

Align a homologous viral gene set (e.g., the L1 major capsid protein gene extracted from Papillomaviridae genomes) with MAFFT and infer a maximum-likelihood phylogeny with IQ-TREE to resolve taxonomy and phylogenetic clade structure across vOTUs.

## When to use

Use when you need to type or classify viral genomes/OTUs from a conserved marker gene (e.g., L1) and place them on a phylogeny against reference sequences from RefSeq.

## When NOT to use

- The viral genomes lack the target marker gene (e.g., non-papillomaviruses without an L1 gene) — use whole-genome or different marker approaches instead
- The goal is abundance/taxonomic profiling of metagenomic reads rather than phylogenetic placement of assembled genomes — use Kraken2/Bracken with a customized database instead

## Inputs

- Viral genome FASTA files (e.g., VMGC_virus.tar.gz, 14,224 viral genomes filtered to ≥50% completeness)
- Genome annotation table (VMGC_virus.info) to select family members (e.g., 627 Papillomaviridae genomes across 61 vOTUs)
- Reference gene sequences from NCBI RefSeq papillomavirus genomes

## Outputs

- Extracted marker-gene sequences (e.g., L1 nucleotide sequences per vOTU)
- Multiple sequence alignment of VMGC + RefSeq L1 sequences (MAFFT output)
- Type assignments table (hpv_type_assignments.tsv) based on 90/70/60% identity thresholds
- Maximum-likelihood phylogenetic tree in Newick format (l1_phylogeny.nwk)

## How to apply

First extract the homologous gene from each viral genome — for papillomaviruses, predict ORFs with Prodigal in meta mode (nucleotide + protein output) and pull out the L1 major capsid protein gene from each of the VMGC vOTU representative genomes. Align the extracted sequences together with corresponding reference L1 sequences retrieved from NCBI RefSeq papillomavirus genomes using MAFFT. From the alignment, compute pairwise nucleotide identities and apply ICTV-style thresholds — >90% identity for the same type, >70% for the same species, >60% for the same genus — to assign taxonomy (yielding 58 HPV types for the 61 Papillomaviridae vOTUs). Finally, infer a maximum-likelihood tree of the L1 alignment with IQ-TREE and export it as a Newick file (l1_phylogeny.nwk) to confirm the vOTUs capture the major phylogenetic clades of human vaginal papillomaviruses.

## Related tools

- **Prodigal** (Predicts open reading frames in meta mode on each papillomavirus genome to extract the L1 major capsid protein gene)
- **MAFFT** (Aligns the combined VMGC + RefSeq L1 nucleotide sequences)
- **IQ-TREE** (Infers the maximum-likelihood phylogeny of the L1 alignment saved as l1_phylogeny.nwk)
- **BLAST** (Used for sequence identity comparison supporting taxonomy assignment of L1 sequences)
- **seqkit** (Sequence manipulation/processing of viral FASTA files during gene extraction)
- **GNU parallel** (Parallelizes per-genome processing steps across the 627 Papillomaviridae genomes) — https://github.com/RChGO/VMGC/tree/main/Pipelines
- **NCBI RefSeq** (Source of reference papillomavirus L1 sequences for combined alignment and phylogeny)

## Examples

```
mafft --auto vmgc_l1_plus_refseq_l1.fa > l1_alignment.fa && iqtree -s l1_alignment.fa -nt AUTO -o l1_phylogeny.nwk
```

## Evaluation signals

- The genome screen recovers 627 Papillomaviridae genomes distributed across 61 vOTUs from the 14,224 VMGC viral genomes
- Pairwise-identity typing with 90/70/60% thresholds yields 58 HPV types (including 5 unidentified types)
- The IQ-TREE L1 topology matches the published figure (Fig. 5b), recovering the major phylogenetic clades of human vaginal papillomaviruses
- Every vOTU contributes exactly one L1 sequence to the MAFFT alignment (no missing or duplicated taxa)
- hpv_type_assignments.tsv covers all 61 vOTUs with a type label or an 'unidentified' designation

## Limitations

- 5 of the 58 HPV types could not be identified, reflecting incomplete reference coverage for some vOTUs
- ICTV-style identity thresholds on L1 are convention-based; genomes near threshold boundaries (e.g., ~90% identity) may be ambiguous in type assignment
- Many VMGC viral genomes are incomplete (only 10.3% estimated complete by CheckV), which can affect ORF prediction and L1 extraction quality

## Evidence

- [other] The 61 Papillomaviridae vOTUs were assigned into 58 HPV types (including 5 unidentified types) based on their L1 genes, and the L1-based phylogenetic analysis showed that these vOTUs represented the major phylogenetic clades of human vaginal papillomaviruses.: [evidence span withheld — anchor status: not_found]
- [methods] Predict open reading frames on each papillomavirus genome with Prodigal (meta mode, nucleotide + protein output) and extract the L1 major capsid protein gene from each genome.: [evidence span withheld — anchor status: not_found]
- [methods] >90% identity for the same type, >70% for the same species, >60% for the same genus — yielding 58 HPV types; write assignments to hpv_type_assignments.tsv.: [evidence span withheld — anchor status: not_found]
- [methods] Infer a maximum-likelihood phylogeny of the L1 alignment with IQ-TREE and save the tree as l1_phylogeny.nwk.: [evidence span withheld — anchor status: not_found]
- [abstract] 10.3% of viral sequences were estimated as complete viral genomes, 29.3% were of high quality (completeness ≥90%) and 60.4% were of medium quality: "10.3% of viral sequences were estimated as complete viral genomes, 29.3% were of high quality"
