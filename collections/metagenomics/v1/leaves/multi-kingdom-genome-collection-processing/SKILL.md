---
name: multi-kingdom-genome-collection-processing
description: Use when you have a mixed FASTA collection of genomes (e.g. prokaryotic
  MAGs, fungal genomes, vOTUs) and need a non-redundant pan-genome gene catalogue
  at a defined amino acid identity threshold.
when_to_use_negative:
- Input is already a gene catalogue or non-redundant protein set (no re-clustering
  needed).
- You only need taxonomic profiling of reads against the genomes — use the customised
  Kraken2/Bracken VMGC database instead of rebuilding gene clusters.
- Genomes are eukaryotic with standard models suited to dedicated predictors; Prodigal
  -p meta is designed for anonymous/prokaryotic-style gene calling.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Prodigal
  role: Predicts protein-coding genes on every genome in anonymous metagenomic mode
    (-p meta), producing .faa/.ffn/.gff files
- name: MMseqs2
  role: easy-linclust clustering of the concatenated ~26 million proteins at 90% (also
    50%/95%) amino acid identity to build VMGC-50/90/95 catalogues
- name: GNU parallel
  role: Parallelises Prodigal gene prediction across all genomes
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:6e1ddc552c887f62df6fd8da309b983536dc183a51ed17db68586dc9cb45fc2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RChGO/VMGC
  tools:
  - GNU parallel
  - MMseqs2
  - Prodigal
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multi-kingdom-genome-collection-processing

## Summary

Constructs a non-redundant multi-kingdom gene catalogue (e.g. VMGC-90) by predicting protein-coding genes across prokaryotic MAGs, fungal and viral genomes with Prodigal, then clustering the ~26 million concatenated proteins with MMseqs2 at 90% amino acid identity.

## When to use

Use when you have a mixed FASTA collection of genomes (e.g. prokaryotic MAGs, fungal genomes, vOTUs) and need a non-redundant pan-genome gene catalogue at a defined amino acid identity threshold.

## When NOT to use

- Input is already a gene catalogue or non-redundant protein set (no re-clustering needed).
- You only need taxonomic profiling of reads against the genomes — use the customised Kraken2/Bracken VMGC database instead of rebuilding gene clusters.
- Genomes are eukaryotic with standard models suited to dedicated predictors; Prodigal -p meta is designed for anonymous/prokaryotic-style gene calling.

## Inputs

- Genome FASTA files for prokaryotic MAGs (n=19,542), fungal genomes (n=11 species) and viral genomes/vOTUs (n=4,263) from the VMGC Zenodo deposit
- Concatenated set of ~26 million predicted protein-coding gene sequences (.faa)

## Outputs

- vmgc90_gene_catalogue.faa — representative sequences of VMGC-90 clusters (1,415,799 non-redundant genes)
- Cluster-to-member mapping file tracing each representative to its member genes
- Per-genome gene prediction files: *.faa (proteins), *.ffn (nucleotide genes), *.gff (coordinates)

## How to apply

Collect the genome FASTA files for all prokaryotic MAGs, fungal genomes and viral genomes/vOTUs (for VMGC, from the Zenodo deposit 10457006). Run Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) on every genome in parallel with GNU parallel, writing translated proteins (*.faa), nucleotide gene sequences (*.ffn) and coordinates (*.gff). Concatenate the predicted proteins from all three kingdoms into a single non-redundant input set of ~26 million protein-coding genes. Cluster the concatenated proteins with MMseqs2 easy-linclust at 90% amino acid identity (VMGC also uses 50% and 95% for VMGC-50/95) to generate the catalogue and its cluster-to-member mapping, writing cluster representatives to vmgc90_gene_catalogue.faa.

## Related tools

- **Prodigal** (Predicts protein-coding genes on every genome in anonymous metagenomic mode (-p meta), producing .faa/.ffn/.gff files)
- **MMseqs2** (easy-linclust clustering of the concatenated ~26 million proteins at 90% (also 50%/95%) amino acid identity to build VMGC-50/90/95 catalogues)
- **GNU parallel** (Parallelises Prodigal gene prediction across all genomes) — https://github.com/RChGO/VMGC

## Examples

```
seqkit seq -m 0 genomes/*.fna > all_genomes.fna; prodigal -p meta -i all_genomes.fna -a all.faa; mmseqs easy-linclust all.faa vmgc90 tmp --min-seq-id 0.9
```

## Evaluation signals

- vmgc90_gene_catalogue.faa exists and contains ~1,415,799 representative sequences (VMGC-90); VMGC-50 and VMGC-95 should contain 595,219 and 1,786,695 respectively.
- The number of genes summed over all .ffn/.faa prediction files matches the expected ~26 million input genes from the three kingdoms.
- Every representative in the catalogue maps to at least one member gene in the cluster-to-member mapping file, and no member is assigned to two representatives.
- Cluster identity threshold check: representative–member sequence pairs are consistent with the 90% (or chosen) amino acid identity cut-off.
- Total member count in the mapping equals the concatenated input gene count (no genes silently dropped).

## Limitations

- Prodigal in meta mode may under- or over-predict genes on divergent fungal/viral genomes, propagating errors into the catalogue.
- The catalogue is incomplete: VMGC lacks representation for non-western populations and misses fungal genera such as Cladosporium and Malassezia observed by ITS sequencing.
- Clustering at a single identity threshold (e.g. 90%) can merge paralogous or functionally distinct genes; multiple thresholds (50/90/95%) should be built to assess sensitivity.

## Evidence

- [abstract] Protein-coding gene clustering at 50%, 90% and 95% amino acid identity to build VMGC-50/90/95 catalogues: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [readme] Genome inputs from Zenodo: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006"
- [abstract] Genome counts feeding the catalogue: "comprising 33,804 microbial genomes spanning 786 prokaryotic species, 11 fungal species and 4,263 viral operational taxonomic units"
- [readme] Parallelised per-genome processing pattern: "find contigs/*.fasta | parallel -k -j 10 mash dist mash.sketch.msh {}"
