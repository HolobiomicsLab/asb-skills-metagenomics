---
name: protein-sequence-clustering
description: Use when you have a set of assembled genomes (e.g. MAGs, fungal genomes,
  vOTUs) and need a non-redundant gene catalogue at a defined amino-acid identity
  threshold, or need to quantify catalogue-to-catalogue protein-space overlap.
when_to_use_negative:
- Input is already a non-redundant gene catalogue or a gene profile table (clustering
  would be redundant)
- You need strain-level gene resolution below the species/SGB level and cannot tolerate
  collapsing sequences at 90% identity
- Your goal is nucleotide-level genome dereplication (the VMGC uses 95% nucleotide
  similarity for species-level genome bins, a different clustering layer)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Predicts protein-coding genes from each VMGC genome in anonymous metagenomic
    mode (-p meta), producing .faa protein, .ffn nucleotide and .gff coordinate files
- name: MMseqs2
  role: Clusters the concatenated ~26 million predicted proteins via easy-linclust
    at 50/90/95% amino acid identity to build the VMGC catalogues, and performs the
    all-vs-all search (90% identity, 80% coverage) against VIRGO-90
- name: GNU parallel
  role: Parallelizes Prodigal gene prediction across all genomes in the collection
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:e08d2ef5092b7bf39c348595e4da40479a3255da800de17338170ff00c02aa4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU parallel
  - MMseqs2
  - Prodigal
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# protein-sequence-clustering

## Summary

Builds a non-redundant gene catalogue from a multi-kingdom genome collection by first predicting protein-coding genes with Prodigal and then clustering the translated proteins at fixed amino-acid identity thresholds; in the VMGC paper this produced the VMGC-50/90/95 catalogues from ~26 million genes spanning prokaryotic, fungal and viral genomes.

## When to use

Use when you have a set of assembled genomes (e.g. MAGs, fungal genomes, vOTUs) and need a non-redundant gene catalogue at a defined amino-acid identity threshold, or need to quantify catalogue-to-catalogue protein-space overlap.

## When NOT to use

- Input is already a non-redundant gene catalogue or a gene profile table (clustering would be redundant)
- You need strain-level gene resolution below the species/SGB level and cannot tolerate collapsing sequences at 90% identity
- Your goal is nucleotide-level genome dereplication (the VMGC uses 95% nucleotide similarity for species-level genome bins, a different clustering layer)

## Inputs

- Genome FASTA files for all prokaryotic MAGs, fungal genomes and viral genomes/vOTUs (from the VMGC Zenodo deposit, record 10457006)
- Concatenated predicted protein sequences (.faa) from all kingdoms (~26 million protein-coding genes)
- External reference gene catalogue (e.g. VIRGO-90, 0.62M genes) for overlap analysis

## Outputs

- VMGC-90 non-redundant gene catalogue (vmgc90_gene_catalogue.faa; 1,415,799 representative genes)
- VMGC-50 and VMGC-95 catalogues (595,219 and 1,786,695 non-redundant genes, respectively)
- Cluster-to-member mapping files linking each representative sequence to its cluster members
- Overlap statistics TSV (e.g. vmgc_virgo90_overlap_stats.tsv: 70.6% VIRGO-90 covered, 30.8% VMGC-90 shared, 162% protein-space increase)

## How to apply

Predict protein-coding genes by running Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) on every genome in parallel with GNU parallel, collecting translated proteins (.faa), nucleotide gene sequences (.ffn) and coordinates (.gff). Concatenate all predicted proteins from each microbial kingdom (prokaryotic MAGs, fungal genomes, viral genomes/vOTUs) into a single input set — ~26 million genes for VMGC. Cluster the concatenated proteins with MMseqs2 easy-linclust at 50%, 90% and 95% amino acid identity to generate the VMGC-50 (595,219 genes), VMGC-90 (1,415,799 genes) and VMGC-95 (1,786,695 genes) non-redundant catalogues plus cluster-to-member mappings. To compare against an existing catalogue (e.g. VIRGO-90, 0.62M genes), run an all-vs-all MMseqs2 search at 90% amino acid identity and 80% sequence coverage and count mutual coverage fractions. Choose 90% identity when you need a catalogue with balanced granularity for read mapping and cross-catalogue comparisons.

## Related tools

- **Prodigal** (Predicts protein-coding genes from each VMGC genome in anonymous metagenomic mode (-p meta), producing .faa protein, .ffn nucleotide and .gff coordinate files)
- **MMseqs2** (Clusters the concatenated ~26 million predicted proteins via easy-linclust at 50/90/95% amino acid identity to build the VMGC catalogues, and performs the all-vs-all search (90% identity, 80% coverage) against VIRGO-90)
- **GNU parallel** (Parallelizes Prodigal gene prediction across all genomes in the collection)

## Examples

```
mmseqs easy-linclust vmgc_all_proteins.faa vmgc90 tmp --min-seq-id 0.9
```

## Evaluation signals

- vmgc90_gene_catalogue.faa exists and its sequence count is consistent with the reported 1,415,799 non-redundant genes
- Each representative sequence traces back to its cluster members via the cluster-to-member mapping file
- The combined catalogue sizes (595,219 / 1,415,799 / 1,786,695 for VMGC-50/90/95) are monotonic in clustering stringency: VMGC-95 > VMGC-90 > VMGC-50
- Overlap fractions against VIRGO-90 fall within a small tolerance of the published values: 70.6% VIRGO-90 coverage, 30.8% VMGC-90 shared, 162% protein-space increase (Fig. 6b)
- Prodigal output files (.faa/.ffn/.gff) are present for every input genome before concatenation

## Limitations

- The VMGC gene space is incomplete, especially for vaginal samples from non-western populations, so the catalogue inherits those sampling gaps
- Protein-coding prediction relies on Prodigal in metagenomic mode, which may miss or mispredict genes in atypical genomes (e.g. fungal or viral elements)
- Cross-catalogue overlap statistics depend on the chosen identity (90%) and coverage (80%) thresholds; different cutoffs change the reported expansion figures

## Evidence

- [abstract] we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [discussion] surpassing the VIRGO database10, VMGC offers an extensive expansion of microbial gene sequences: "surpassing the VIRGO database10, VMGC offers an extensive expansion of microbial gene sequences"
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
