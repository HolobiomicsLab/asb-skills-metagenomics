---
name: l1-gene-extraction-from-papillomavirus-genomes
description: Use when you have assembled papillomavirus/viral genomes and need to
  extract the L1 major capsid gene for HPV type assignment or phylogenetic tree reconstruction.
when_to_use_negative:
- Input genomes are not from the Papillomaviridae family — the L1 major capsid locus
  is papillomavirus-specific and ORF prediction alone will not substitute for L1 identification
- You need whole-virus genome quality assessment or vOTU clustering rather than a
  specific gene-level typing — use CheckV/virusClstr.sh-based workflows instead
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Prodigal
  role: Predicts open reading frames on each papillomavirus genome in meta mode (nucleotide
    + protein output) so the L1 major capsid protein gene can be located and extracted
- name: MAFFT
  role: Aligns the extracted L1 sequences and the combined VMGC + RefSeq L1 set prior
    to identity computation and tree inference
- name: IQ-TREE
  role: Infers the maximum-likelihood phylogeny of the L1 alignment, saved as l1_phylogeny.nwk
- name: NCBI RefSeq
  role: Source of reference papillomavirus L1 sequences for combined alignment and
    Fig. 5b topology comparison
- name: BLAST
  role: Used alongside GNU parallel in the L1-based typing/assignment workflow
- name: seqkit
  role: Sequence manipulation/extraction helper in the L1 typing workflow
- name: GNU parallel
  role: Parallelizes per-genome ORF prediction and sequence processing across the
    627 Papillomaviridae genomes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:9857697eb020770f812a2d25988c73ee009f2419a03da6b3e95ad19cd7f2855d
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# l1-gene-extraction-from-papillomavirus-genomes

## Summary

Extracts the L1 major capsid protein gene from Papillomaviridae genomes in the VMGC viral collection (627 genomes across 61 vOTUs) to support HPV typing and L1-based phylogenetics under ICTV-style identity thresholds.

## When to use

Use when you have assembled papillomavirus/viral genomes and need to extract the L1 major capsid gene for HPV type assignment or phylogenetic tree reconstruction.

## When NOT to use

- Input genomes are not from the Papillomaviridae family — the L1 major capsid locus is papillomavirus-specific and ORF prediction alone will not substitute for L1 identification
- You need whole-virus genome quality assessment or vOTU clustering rather than a specific gene-level typing — use CheckV/virusClstr.sh-based workflows instead

## Inputs

- VMGC viral genome FASTA collection (VMGC_virus.tar.gz; 14,224 viral genomes)
- VMGC_virus.info annotation table (taxonomy/quality/clustering/host fields)
- NCBI RefSeq papillomavirus L1 sequences (for combined alignment)

## Outputs

- Extracted L1 nucleotide/protein sequences per papillomavirus genome
- hpv_type_assignments.tsv (HPV type per vOTU under 90/70/60% L1 identity thresholds)
- l1_phylogeny.nwk (IQ-TREE maximum-likelihood tree of the MAFFT-aligned VMGC + RefSeq L1 set)

## How to apply

First screen the 14,224 VMGC viral genomes (VMGC_virus.tar.gz) using VMGC_virus.info annotations to identify the 627 Papillomaviridae genomes belonging to 61 vOTUs. Predict open reading frames on each papillomavirus genome with Prodigal in meta mode, producing both nucleotide and protein outputs, and extract the L1 major capsid protein gene from each genome. Align the extracted L1 sequences (e.g., with MAFFT) and compute pairwise nucleotide identities to assign taxonomy by ICTV-style thresholds: >90% identity for the same type, >70% for the same species, >60% for the same genus, which yielded 58 HPV types (including 5 unidentified types). For phylogenetic validation, retrieve matching L1 sequences from NCBI RefSeq papillomavirus genomes, align the combined VMGC + RefSeq L1 set with MAFFT, and infer a maximum-likelihood phylogeny with IQ-TREE.

## Related tools

- **Prodigal** (Predicts open reading frames on each papillomavirus genome in meta mode (nucleotide + protein output) so the L1 major capsid protein gene can be located and extracted)
- **MAFFT** (Aligns the extracted L1 sequences and the combined VMGC + RefSeq L1 set prior to identity computation and tree inference)
- **IQ-TREE** (Infers the maximum-likelihood phylogeny of the L1 alignment, saved as l1_phylogeny.nwk)
- **NCBI RefSeq** (Source of reference papillomavirus L1 sequences for combined alignment and Fig. 5b topology comparison)
- **BLAST** (Used alongside GNU parallel in the L1-based typing/assignment workflow)
- **seqkit** (Sequence manipulation/extraction helper in the L1 typing workflow)
- **GNU parallel** (Parallelizes per-genome ORF prediction and sequence processing across the 627 Papillomaviridae genomes)

## Examples

```
prodigal -i papillomavirus_genome.fa -p meta -d orfs.fna -a orfs.faa
```

## Evaluation signals

- Screening must recover 627 Papillomaviridae genomes distributed across 61 vOTUs from the 14,224 VMGC viral genomes
- The 90/70/60% L1 pairwise identity thresholds must yield 58 HPV types (including 5 unidentified types) written to hpv_type_assignments.tsv
- The IQ-TREE L1 topology (l1_phylogeny.nwk) should match the published Fig. 5b and represent the major phylogenetic clades of human vaginal papillomaviruses
- Each of the 61 vOTUs should yield at least one identifiable L1 gene after Prodigal meta-mode prediction

## Limitations

- 5 of the 58 HPV types could not be identified, reflecting the incompleteness of current viral databases — 'many phages in the VMGC are unknown, and their functional exploration based on existing databases remains limited'
- L1-based typing depends on genome quality; only 10.3% of VMGC viral sequences were estimated as complete viral genomes, and the rest are ≥50% completeness, so L1 regions may be truncated or missing in partial genomes

## Evidence

- [other] Screen the 14,224 VMGC viral genomes (VMGC_virus.tar.gz) using the VMGC_virus.info annotations to identify the 627 Papillomaviridae genomes belonging to 61 vOTUs.: [evidence span withheld — anchor status: not_found]
- [methods] Predict open reading frames on each papillomavirus genome with Prodigal (meta mode, nucleotide + protein output) and extract the L1 major capsid protein gene from each genome.: [evidence span withheld — anchor status: not_found]
- [methods] compute pairwise nucleotide identities to assign taxonomy by ICTV-style thresholds — >90% identity for the same type, >70% for the same species, >60% for the same genus: [evidence span withheld — anchor status: not_found]
- [readme] | Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz |: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
