---
name: non-redundant-gene-catalogue-construction
description: Use when you have a large collection of (meta)genomes — e.g. 33,804 VMGC
  prokaryotic/fungal/viral genomes — and need a non-redundant reference gene catalogue
  for read mapping or functional profiling.
when_to_use_negative:
- Your input is already a non-redundant gene catalogue or a dereplicated set of cluster
  representatives — re-clustering adds no information
- You only have unassembled reads rather than genome FASTA files — gene prediction
  with Prodigal (-p meta) requires contigs/genomes, not raw FASTQ
- You need taxonomically annotated genomes rather than a gene-level catalogue — use
  the VMGC SGB/Kraken resources instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0081
tools:
- name: Prodigal 2.6.3
  role: Predicts protein-coding genes from every prokaryotic, fungal and viral genome
    in anonymous metagenomic mode (-p meta), producing *.faa, *.ffn and *.gff files
- name: GNU parallel
  role: Runs Prodigal gene prediction across all genomes in parallel
- name: MMseqs2 (easy-linclust)
  role: Clusters the concatenated ~26 million proteins at 50%, 90% and 95% amino acid
    identity to build the VMGC-50/90/95 non-redundant gene catalogues
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:da42ad29d5c7db6b3d5d9bdde9ec365ab8deb8fa01847962ea28e69a1e9bd5cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNU parallel
  - MMseqs2 (easy-linclust)
  - Prodigal
  - Prodigal 2.6.3
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0081
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# non-redundant-gene-catalogue-construction

## Summary

Builds a non-redundant gene catalogue by predicting protein-coding genes across a multi-kingdom genome collection (prokaryotic MAGs, fungal genomes, viral genomes/vOTUs) and clustering the concatenated proteins at fixed amino-acid identity thresholds (50%, 90%, 95%) to yield VMGC-50/90/95.

## When to use

Use when you have a large collection of (meta)genomes — e.g. 33,804 VMGC prokaryotic/fungal/viral genomes — and need a non-redundant reference gene catalogue for read mapping or functional profiling.

## When NOT to use

- Your input is already a non-redundant gene catalogue or a dereplicated set of cluster representatives — re-clustering adds no information
- You only have unassembled reads rather than genome FASTA files — gene prediction with Prodigal (-p meta) requires contigs/genomes, not raw FASTQ
- You need taxonomically annotated genomes rather than a gene-level catalogue — use the VMGC SGB/Kraken resources instead

## Inputs

- Genome FASTA files for all prokaryotic MAGs (n=19,542), fungal/eukaryotic genomes and viral genomes/vOTUs (n=14,224 / 4,263 vOTUs) from the VMGC Zenodo deposit (https://zenodo.org/records/10457006)
- Concatenated predicted protein set of ~26 million protein-coding genes (*.faa)

## Outputs

- vmgc90_gene_catalogue.faa — representative protein sequences of VMGC-90 clusters (1,415,799 non-redundant genes)
- VMGC-50 catalogue (595,219 genes) and VMGC-95 catalogue (1,786,695 genes) at the alternative identity thresholds
- Cluster-to-member mapping file linking each representative to its cluster members

## How to apply

First predict protein-coding genes on every genome by running Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) in parallel with GNU parallel, capturing translated proteins (*.faa), nucleotide CDS (*.ffn) and coordinates (*.gff). Concatenate the predicted proteins from all kingdoms into a single non-redundant input set of approximately 26 million protein-coding genes. Cluster the concatenated proteins with MMseqs2 easy-linclust at 90% amino acid identity to generate the VMGC-90 catalogue plus its cluster-to-member mapping, and repeat at 50% and 95% identity for VMGC-50 and VMGC-95. Write the cluster representative sequences to vmgc90_gene_catalogue.faa. The rationale for multiple thresholds is that looser clustering (50%) gives a more compressed catalogue (595,219 genes) while stricter clustering (95%) preserves finer sequence diversity (1,786,695 genes); VMGC-90 sits between them with 1,415,799 genes.

## Related tools

- **Prodigal 2.6.3** (Predicts protein-coding genes from every prokaryotic, fungal and viral genome in anonymous metagenomic mode (-p meta), producing *.faa, *.ffn and *.gff files)
- **GNU parallel** (Runs Prodigal gene prediction across all genomes in parallel)
- **MMseqs2 (easy-linclust)** (Clusters the concatenated ~26 million proteins at 50%, 90% and 95% amino acid identity to build the VMGC-50/90/95 non-redundant gene catalogues)

## Examples

```
mmseqs easy-linclust all_vmgc_proteins.faa vmgc90 tmp --min-seq-id 0.9 -c 0.8
```

## Evaluation signals

- vmgc90_gene_catalogue.faa exists and contains approximately 1,415,799 non-redundant gene sequences, consistent with the published VMGC-90 size
- Each representative sequence in the catalogue is traceable to its cluster members via the cluster-to-member mapping file
- Catalogue sizes follow the expected ordering across thresholds: VMGC-50 (595,219) < VMGC-90 (1,415,799) < VMGC-95 (1,786,695) ≤ total input genes (~26 million)
- Prodigal was run in metagenomic mode (-p meta) with *.faa, *.ffn and *.gff outputs produced for all three kingdoms
- Input gene pool matches the genome inventory (19,542 prokaryotic genomes, 42 eukaryotic genomes, 14,224 viral genomes) from the VMGC deposit

## Limitations

- The catalogue is bounded by the underlying genome collection: microbial sequences captured by the VMGC are not complete, especially for vaginal samples from the non-western population
- Quality-filtered genomes underlie the catalogue (prokaryotic genomes required ≥50% completeness and <5% contamination; viral sequences required ≥50% completeness), so genes from excluded low-quality assemblies are absent
- Potential contamination from experimental operations or kits in public vaginal metagenomic datasets used to construct the VMGC may propagate into the catalogue

## Evidence

- [abstract] we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [methods] Run Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) on every genome in parallel with GNU parallel to predict protein-coding genes: [evidence span withheld — anchor status: not_found]
- [results] Clustering approximately 26 million protein-coding genes from all prokaryotic, fungal and viral VMGC genomes at 90% amino acid identity generates the VMGC-90 catalogue containing 1,415,799 non-redundant genes: [evidence span withheld — anchor status: not_found]
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
