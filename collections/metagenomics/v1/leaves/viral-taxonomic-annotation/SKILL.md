---
name: viral-taxonomic-annotation
description: Use when assembled viral contigs or vOTUs (e.g. from MAG-derived metagenomic
  assemblies) need family-, genus-, species- or type-level taxonomic labels, or when
  viral OTU clusters must be typed against a reference set such as NCBI RefSeq.
when_to_use_negative:
- When input is raw reads rather than assembled viral contigs — taxonomic profiling
  of reads is handled separately (e.g. Kraken2/Bracken against customized databases)
- When the goal is host prediction rather than taxonomic annotation (use the virusHost.sh
  host-assignment step)
- When viral genomes lack the type-defining locus (e.g. no L1 gene), identity-threshold
  typing cannot be applied and genomes must remain 'unidentified' types
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Predict open reading frames on each papillomavirus genome (meta mode, nucleotide
    + protein output) to extract the L1 major capsid protein gene
- name: MAFFT
  role: Align extracted VMGC L1 sequences with NCBI RefSeq L1 sequences for phylogenetic
    placement
- name: IQ-TREE
  role: Infer the maximum-likelihood phylogeny of the L1 alignment saved as l1_phylogeny.nwk
- name: BLAST
  role: Homology-based comparison of viral genes against reference databases for taxonomic
    assignment
- name: seqkit
  role: Sequence manipulation and extraction of genomes/genes from the VMGC_virus.tar.gz
    set
- name: GNU parallel
  role: Parallelize per-genome ORF prediction and annotation across the 627 Papillomaviridae
    genomes
- name: NCBI RefSeq
  role: Source of reference papillomavirus genomes/L1 sequences for type assignment
    and phylogeny
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:020e788b6278cd191868312d75a9f1612f50935a99d2dbeff8b0ea7f546b906e
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# viral-taxonomic-annotation

## Summary

Assigns taxonomy to assembled viral genomes/vOTUs using combined feature- and homology-based identification plus family- and type-level annotation against reference databases, as applied to the 14,224 viral genomes in the VMGC (e.g. classifying 627 Papillomaviridae genomes into 61 vOTUs and 58 HPV types via L1 gene identities).

## When to use

Use when assembled viral contigs or vOTUs (e.g. from MAG-derived metagenomic assemblies) need family-, genus-, species- or type-level taxonomic labels, or when viral OTU clusters must be typed against a reference set such as NCBI RefSeq.

## When NOT to use

- When input is raw reads rather than assembled viral contigs — taxonomic profiling of reads is handled separately (e.g. Kraken2/Bracken against customized databases)
- When the goal is host prediction rather than taxonomic annotation (use the virusHost.sh host-assignment step)
- When viral genomes lack the type-defining locus (e.g. no L1 gene), identity-threshold typing cannot be applied and genomes must remain 'unidentified' types

## Inputs

- Assembled viral genomes/contigs with annotation table (e.g. VMGC_virus.tar.gz, n=14,224 genomes, with VMGC_virus.info carrying Taxonomy/Quality/Clustering/Host fields)
- vOTU clustering assignments (e.g. 4,263 vOTUs)
- Reference viral genomes and their taxonomic annotations (NCBI RefSeq papillomavirus genomes)
- Nucleotide assemblies of the target viral family (627 Papillomaviridae genomes across 61 vOTUs)

## Outputs

- Per-genome taxonomic assignments at family/genus/species/type level (e.g. hpv_type_assignments.tsv for 58 HPV types)
- Pairwise nucleotide identity matrix of the type-defining gene (L1)
- Maximum-likelihood phylogeny of the L1 alignment (l1_phylogeny.nwk) including reference sequences
- Updated VMGC_virus.info-style annotations mapping vOTUs to taxa and hosts

## How to apply

Identify viral contigs from metagenomic assemblies with a combined feature- and homology-based approach (the VMGC pipeline's virusFind_flow.sh, clustering with virusClstr.sh, and annotation with virusTaxAnno.sh from github.com/RChGO/VMGC). For a targeted typing task, screen the annotated viral genomes (e.g. VMGC_virus.tar.gz with VMGC_virus.info) to select the family of interest — here 627 Papillomaviridae genomes in 61 vOTUs. Predict ORFs on each genome with Prodigal in meta mode (nucleotide + protein output), extract the type-defining gene (the L1 major capsid protein for papillomaviruses), and align L1 sequences to compute pairwise nucleotide identities. Apply ICTV-style thresholds — >90% identity for same type, >70% for same species, >60% for same genus — to assign types (yielding 58 HPV types, including 5 unidentified), and confirm phylogenetic placement by aligning VMGC + NCBI RefSeq L1 sequences with MAFFT and inferring a maximum-likelihood tree with IQ-TREE.

## Related tools

- **Prodigal** (Predict open reading frames on each papillomavirus genome (meta mode, nucleotide + protein output) to extract the L1 major capsid protein gene)
- **MAFFT** (Align extracted VMGC L1 sequences with NCBI RefSeq L1 sequences for phylogenetic placement)
- **IQ-TREE** (Infer the maximum-likelihood phylogeny of the L1 alignment saved as l1_phylogeny.nwk)
- **BLAST** (Homology-based comparison of viral genes against reference databases for taxonomic assignment)
- **seqkit** (Sequence manipulation and extraction of genomes/genes from the VMGC_virus.tar.gz set)
- **GNU parallel** (Parallelize per-genome ORF prediction and annotation across the 627 Papillomaviridae genomes)
- **NCBI RefSeq** (Source of reference papillomavirus genomes/L1 sequences for type assignment and phylogeny)

## Examples

```
prodigal -meta -i papillomavirus_genome.fa -a proteins.faa -d genes.ffn && mafft --auto vmgc_l1.fa refseq_l1.fa > l1_aln.fa && iqtree -s l1_aln.fa -nt AUTO
```

## Evaluation signals

- Annotation step recovers 627 Papillomaviridae genomes assigned to 61 vOTUs from the 14,224-genome VMGC viral set
- Identity-threshold typing yields 58 HPV types (including 5 unidentified) under the 90/70/60% L1 nucleotide identity cutoffs
- IQ-TREE maximum-likelihood topology of L1 matches the published Fig. 5b, covering the major phylogenetic clades of human vaginal papillomaviruses
- Every genome in the target family receives a type label or is explicitly flagged as an unidentified type; no silent drops
- Per-vOTU type assignments are consistent (one type per vOTU unless identity evidence supports otherwise) and reproducible when re-run against the same RefSeq reference set

## Limitations

- Many phages in the VMGC are unknown, so functional exploration and taxonomic assignment based on existing databases remains limited
- 85.8% of VMGC vOTUs were not found in other large-scale virome databases, meaning reference-based typing may fail for the majority of vOTUs
- Further isolation, cultivation and physiological studies are necessary to fully resolve phage genomic information, limiting confidence in annotations of uncultured viruses

## Evidence

- [results] The 61 Papillomaviridae vOTUs were assigned into 58 HPV types (including 5 unidentified types) based on their L1 genes: [evidence span withheld — anchor status: not_found]
- [methods] virusTaxAnno.sh script: "we performed virus identification on the metagenome-assembled contigs using a combined feature- and homology-based approach"
- [readme] VirusTaxAnno script in VMGC repo: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
- [readme] Annotations of viral genomes: "Annotations of viral genomes (Taxonomy/Quality/Clusting/Host...)  | 969 KB | VMGC_virus.info"
- [discussion] many phages unknown limitation: "many phages in the VMGC are unknown, and their functional exploration based on existing databases remains limited"
