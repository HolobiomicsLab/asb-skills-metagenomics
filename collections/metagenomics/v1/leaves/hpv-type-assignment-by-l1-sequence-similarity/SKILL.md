---
name: hpv-type-assignment-by-l1-sequence-similarity
description: Use when you have Papillomavirus (Papillomaviridae) genomes — e.g. from
  the VMGC's 627 genomes in 61 vOTUs — and need L1-based type/species/genus classification
  plus a phylogeny of human vaginal papillomaviruses.
when_to_use_negative:
- Viral genomes are not Papillomaviridae — the 90/70/60% L1 thresholds are papillomavirus-specific
  and invalid for phage or other DNA virus families
- Genomes are too fragmented to carry an intact L1 ORF, since type calling depends
  entirely on the L1 major capsid protein gene
- The goal is species-level prokaryotic typing (e.g. Kraken2/Bracken SGB profiling
  against VMGC prokaryotic SGBs), which uses a completely different database and read-mapping
  approach
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0092
tools:
- name: Prodigal
  role: Predicts open reading frames on each papillomavirus genome (meta mode, nucleotide
    + protein output) so the L1 major capsid protein gene can be extracted
- name: seqkit
  role: Manipulates and extracts the L1 sequences from predicted ORF outputs
- name: BLAST
  role: Supports identification/verification of L1 major capsid protein genes among
    predicted ORFs
- name: MAFFT
  role: Aligns the combined VMGC + RefSeq L1 sequence set before phylogenetic inference
- name: IQ-TREE
  role: Infers the maximum-likelihood phylogeny of the L1 alignment, saved as l1_phylogeny.nwk
- name: GNU parallel
  role: Parallelizes ORF prediction and sequence processing across the 627 Papillomaviridae
    genomes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:aa8db43c538aff4e0f71d22aba0144088e657c683561aa246594f8f9d5b0eee9
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# HPV type assignment by L1 sequence similarity

## Summary

Assigns Papillomaviridae vOTUs to HPV types by predicting the L1 major capsid protein from each vaginal viral genome and applying ICTV-style pairwise nucleotide identity thresholds (>90% type, >70% species, >60% genus), then placing the VMGC L1 sequences into a RefSeq-anchored maximum-likelihood phylogeny.

## When to use

Use when you have Papillomavirus (Papillomaviridae) genomes — e.g. from the VMGC's 627 genomes in 61 vOTUs — and need L1-based type/species/genus classification plus a phylogeny of human vaginal papillomaviruses.

## When NOT to use

- Viral genomes are not Papillomaviridae — the 90/70/60% L1 thresholds are papillomavirus-specific and invalid for phage or other DNA virus families
- Genomes are too fragmented to carry an intact L1 ORF, since type calling depends entirely on the L1 major capsid protein gene
- The goal is species-level prokaryotic typing (e.g. Kraken2/Bracken SGB profiling against VMGC prokaryotic SGBs), which uses a completely different database and read-mapping approach

## Inputs

- Papillomavirus genome FASTA files (Papillomaviridae subset of VMGC_virus.tar.gz)
- VMGC_virus.info annotation table (taxonomy/vOTU/quality/host annotations)
- NCBI RefSeq papillomavirus genomes for reference L1 sequences

## Outputs

- hpv_type_assignments.tsv (per-vOTU HPV type assignments)
- Extracted L1 nucleotide and protein sequences per genome
- MAFFT multiple-sequence alignment of combined VMGC + RefSeq L1 sequences
- l1_phylogeny.nwk (IQ-TREE maximum-likelihood L1 tree)

## How to apply

First screen the annotated viral genome set (VMGC_virus.tar.gz with VMGC_virus.info, 14,224 viral genomes / 4,263 vOTUs) to isolate the 627 Papillomaviridae genomes belonging to 61 vOTUs. Predict open reading frames on each genome with Prodigal in meta mode, emitting both nucleotide and protein output, and extract the L1 major capsid protein gene per genome (seqkit and BLAST assist extraction/verification). Align the L1 sequences, compute pairwise nucleotide identities, and assign taxonomy by ICTV-style thresholds — >90% identity = same type, >70% = same species, >60% = same genus — writing assignments to hpv_type_assignments.tsv (expected: 58 HPV types including 5 unidentified). Retrieve corresponding L1 sequences from NCBI RefSeq papillomavirus genomes, align the combined VMGC + RefSeq set with MAFFT, and infer a maximum-likelihood tree with IQ-TREE saved as l1_phylogeny.nwk.

## Related tools

- **Prodigal** (Predicts open reading frames on each papillomavirus genome (meta mode, nucleotide + protein output) so the L1 major capsid protein gene can be extracted)
- **seqkit** (Manipulates and extracts the L1 sequences from predicted ORF outputs)
- **BLAST** (Supports identification/verification of L1 major capsid protein genes among predicted ORFs)
- **MAFFT** (Aligns the combined VMGC + RefSeq L1 sequence set before phylogenetic inference)
- **IQ-TREE** (Infers the maximum-likelihood phylogeny of the L1 alignment, saved as l1_phylogeny.nwk)
- **GNU parallel** (Parallelizes ORF prediction and sequence processing across the 627 Papillomaviridae genomes)

## Examples

```
prodigal -i papillomavirus_genome.fa -p meta -d l1_nucl.fa -a l1_prot.fa
```

## Evaluation signals

- The Papillomaviridae screen recovers exactly 627 genomes distributed across 61 vOTUs from the 14,224-genome viral set
- The 90/70/60% L1 identity thresholds yield 58 HPV types (including 5 unidentified types) across the 61 vOTUs
- hpv_type_assignments.tsv covers every one of the 61 vOTUs with a single type call
- The IQ-TREE L1 tree (l1_phylogeny.nwk) places VMGC L1 sequences such that the major phylogenetic clades of human vaginal papillomaviruses are represented, matching the published Fig. 5b topology

## Limitations

- 5 of the 58 HPV types could not be identified, reflecting incompleteness of reference type coverage
- Many VMGC viral genomes are partial — only 10.3% were estimated as complete (≥50% completeness cutoff applied) — so L1 may be absent or truncated in some genomes
- Many phages/papillomaviruses in the VMGC are unknown and functional exploration against existing databases remains limited

## Evidence

- [results] The 61 Papillomaviridae vOTUs were assigned into 58 HPV types (including 5 unidentified types) based on their L1 genes: [evidence span withheld — anchor status: not_found]
- [methods] ICTV-style identity thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] Viral genome completeness cutoff: "resulting in 14,224 viral sequences achieving at least 50% completeness"
- [readme] Viral genome data files in the VMGC Zenodo deposit: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
