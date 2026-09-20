---
name: reference-database-integration
description: Use when viral or microbial genomes from a custom catalogue need taxonomic
  typing against an external reference database, and/or a combined catalogue+reference
  phylogeny of a marker gene (e.g., L1) is required.
when_to_use_negative:
- When the goal is whole-genome similarity or species-level binning of the catalogue
  (95% nucleotide SGB clustering or Mash distances), not marker-gene typing against
  external references
- When input contigs are too fragmented to yield an intact marker locus — the 90/70/60%
  identity typing depends on recovering a complete L1 ortholog from each genome
- When classifying reads/samples by composition against a reference catalogue (use
  the Kraken2/Bracken SGB database workflow instead)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0194
tools:
- name: Prodigal
  role: Predicts ORFs in meta mode (nucleotide + protein output) on each papillomavirus
    genome so the L1 major capsid gene can be extracted for typing and phylogeny
- name: BLAST
  role: Used with seqkit and GNU parallel to compute pairwise marker-gene identities
    for ICTV-threshold-based type assignment against references
- name: MAFFT
  role: Aligns the combined VMGC + NCBI RefSeq L1 sequence set before tree inference
- name: IQ-TREE
  role: Infers the maximum-likelihood phylogeny of the combined L1 alignment, saved
    as l1_phylogeny.nwk
- name: seqkit
  role: Sequence manipulation/extraction of L1 genes during marker-gene subsetting
- name: GNU parallel
  role: Parallelizes per-genome ORF prediction and pairwise identity computation across
    the 627 Papillomaviridae genomes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:eca17c44f08678fe973d607b4dcfce34dea22de4c83834dfe868e5f32425983b
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# reference-database-integration

## Summary

Integrates an in-house genome catalogue (e.g., the 14,224-genome VMGC viral set) with external references such as NCBI RefSeq to assign taxonomy at marker-gene level and build combined phylogenies. In this paper it types Papillomaviridae vOTUs as HPV types via L1 identity thresholds and places them among RefSeq references in a maximum-likelihood tree.

## When to use

Use when viral or microbial genomes from a custom catalogue need taxonomic typing against an external reference database, and/or a combined catalogue+reference phylogeny of a marker gene (e.g., L1) is required.

## When NOT to use

- When the goal is whole-genome similarity or species-level binning of the catalogue (95% nucleotide SGB clustering or Mash distances), not marker-gene typing against external references
- When input contigs are too fragmented to yield an intact marker locus — the 90/70/60% identity typing depends on recovering a complete L1 ortholog from each genome
- When classifying reads/samples by composition against a reference catalogue (use the Kraken2/Bracken SGB database workflow instead)

## Inputs

- VMGC_virus.tar.gz (14,224 viral genomes) with VMGC_virus.info annotations
- NCBI RefSeq papillomavirus genomes (reference L1 sequences)
- Reference taxonomy thresholds (ICTV-style: 90/70/60% L1 nucleotide identity)

## Outputs

- hpv_type_assignments.tsv (per-vOTU type assignments under 90/70/60% thresholds)
- Multi-FASTA of marker genes (L1) from catalogue + RefSeq references
- l1_phylogeny.nwk (IQ-TREE maximum-likelihood tree of the combined L1 alignment)

## How to apply

First, subset the catalogue of interest using its annotation file (e.g., VMGC_virus.info) to pull the target group — here 627 Papillomaviridae genomes across 61 vOTUs from VMGC_virus.tar.gz. Predict ORFs with Prodigal in meta mode (nucleotide + protein output) and extract the marker gene (L1 major capsid protein) from each genome. Assign taxonomy by computing pairwise nucleotide identities of the marker alignment against ICTV-style thresholds — >90% for same type, >70% for same species, >60% for same genus — writing assignments to a table such as hpv_type_assignments.tsv. Then retrieve the homologous marker sequences from NCBI RefSeq, align the combined catalogue+RefSeq set with MAFFT, and infer a maximum-likelihood tree with IQ-TREE saved as a Newick file (l1_phylogeny.nwk). Correctness is judged by recovering the expected counts (627 genomes / 61 vOTUs / 58 HPV types, including 5 unidentified) and a topology matching the published figure (Fig. 5b).

## Related tools

- **Prodigal** (Predicts ORFs in meta mode (nucleotide + protein output) on each papillomavirus genome so the L1 major capsid gene can be extracted for typing and phylogeny)
- **BLAST** (Used with seqkit and GNU parallel to compute pairwise marker-gene identities for ICTV-threshold-based type assignment against references)
- **MAFFT** (Aligns the combined VMGC + NCBI RefSeq L1 sequence set before tree inference)
- **IQ-TREE** (Infers the maximum-likelihood phylogeny of the combined L1 alignment, saved as l1_phylogeny.nwk)
- **seqkit** (Sequence manipulation/extraction of L1 genes during marker-gene subsetting)
- **GNU parallel** (Parallelizes per-genome ORF prediction and pairwise identity computation across the 627 Papillomaviridae genomes)

## Examples

```
prodigal -i papillomavirus_vOTU_01.fasta -p meta -d l1_genes.fna -a l1_proteins.faa
```

## Evaluation signals

- Annotation screening recovers exactly 627 Papillomaviridae genomes belonging to 61 vOTUs from the 14,224-genome viral set
- Identity-based typing yields 58 HPV types (including 5 unidentified types) under the >90% (type) / >70% (species) / >60% (genus) L1 thresholds
- The IQ-TREE L1 topology matches the published figure (Fig. 5b), recovering the major phylogenetic clades of human vaginal papillomaviruses
- Every vOTU in the subset has both an L1 sequence and a RefSeq homolog represented in the MAFFT alignment (no dropped tips)
- hpv_type_assignments.tsv covers all 61 vOTUs with non-null type labels or an explicit 'unidentified' designation

## Limitations

- Identity thresholds on a single marker locus (L1) cannot resolve types below the 90% cutoff — 5 of the 58 assigned types remained unidentified
- Viral sequences were only kept at ≥50% completeness, so truncated genomes may lack an intact L1 and be dropped from typing/phylogeny
- Reference coverage is a hard constraint: 85.8% of VMGC vOTUs were absent from existing large-scale virome databases, so many catalogue members have no close external reference for comparison

## Evidence

- [other] Screen the 14,224 VMGC viral genomes (VMGC_virus.tar.gz) using the VMGC_virus.info annotations to identify the 627 Papillomaviridae genomes: [evidence span withheld — anchor status: not_found]
- [other] ICTV identity thresholds: [evidence span withheld — anchor status: not_found]
- [other] MAFFT + IQ-TREE combined tree: [evidence span withheld — anchor status: not_found]
- [readme] VMGC viral data files: "| Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz |"
- [abstract] Limited overlap with existing reference databases: "The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases"
