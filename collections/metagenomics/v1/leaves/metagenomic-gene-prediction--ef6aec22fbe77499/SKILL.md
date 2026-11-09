---
name: metagenomic-gene-prediction
description: Use when you have assembled genome FASTA files (MAGs, fungal genomes,
  viral genomes/vOTUs) and need translated protein-coding gene sequences as input
  for downstream gene clustering, annotation or gene catalogue construction.
when_to_use_negative:
- Input is raw metagenomic reads rather than assembled contigs/genomes — gene prediction
  should follow assembly and binning (e.g. the MetaBAT2 single-/multi-coverage binning
  workflow).
- Sequences are already predicted genes or a prebuilt gene catalogue (e.g. the existing
  VIRGO catalogue of 0.95 million genes) rather than genome FASTA files.
- Genomes come from a single well-annotated isolate set where RefSeq/NCBI annotations
  are already available and consistent.
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Prodigal 2.6.3
  role: Predicts protein-coding genes on each genome in anonymous metagenomic mode
    (-p meta), producing .faa, .ffn and .gff outputs
- name: GNU parallel 20201122
  role: Parallelizes Prodigal gene prediction across all genomes in the collection
  repo: https://github.com/RChGO/VMGC
- name: MMseqs2 easy-linclust
  role: Consumes the concatenated predicted proteins to cluster them at 50%, 90% and
    95% amino acid identity into the VMGC gene catalogues
  repo: https://github.com/RChGO/VMGC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:196da7688e32ba3e9fbad73fd88783b59ae0008fc23f3d90650272abb3e75d60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RChGO/VMGC
  tools:
  - GNU parallel 20201122
  - MMseqs2 easy-linclust
  - Prodigal
  - Prodigal 2.6.3
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metagenomic-gene-prediction

## Summary

Predicts protein-coding genes on assembled microbial genomes using Prodigal 2.6.3 in anonymous metagenomic mode (-p meta), producing translated protein FASTA, nucleotide gene FASTA and GFF coordinates. In the VMGC paper this step converts 33,804 prokaryotic, fungal and viral genomes into the ~26 million protein-coding genes that feed the VMGC-50/90/95 non-redundant gene catalogues.

## When to use

Use when you have assembled genome FASTA files (MAGs, fungal genomes, viral genomes/vOTUs) and need translated protein-coding gene sequences as input for downstream gene clustering, annotation or gene catalogue construction.

## When NOT to use

- Input is raw metagenomic reads rather than assembled contigs/genomes — gene prediction should follow assembly and binning (e.g. the MetaBAT2 single-/multi-coverage binning workflow).
- Sequences are already predicted genes or a prebuilt gene catalogue (e.g. the existing VIRGO catalogue of 0.95 million genes) rather than genome FASTA files.
- Genomes come from a single well-annotated isolate set where RefSeq/NCBI annotations are already available and consistent.

## Inputs

- Genome FASTA files of prokaryotic MAGs (e.g. VMGC_prokaryote_MAG.tar.gz, n=19,542)
- Fungal/eukaryotic genome FASTA files (VMGC_eukaryocyte.tar.gz, n=42)
- Viral genome and vOTU FASTA files (VMGC_virus.tar.gz, n=14,224 genomes / 4,263 vOTUs)

## Outputs

- Translated protein-coding gene sequences (*.faa)
- Nucleotide gene sequences (*.ffn)
- Gene coordinates in GFF format (*.gff)
- Concatenated protein set of ~26 million genes for downstream clustering

## How to apply

Collect the genome FASTA files for all prokaryotic MAGs, fungal genomes and viral genomes/vOTUs (for VMGC these are deposited at https://zenodo.org/records/10457006). Run Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) on every genome in parallel using GNU parallel 20201122, since -p meta avoids needing a training genome and works across the divergent prokaryotic, fungal and viral sequences. Write translated proteins to *.faa, nucleotide gene sequences to *.ffn and gene coordinates to *.gff. Concatenate the predicted protein sequences from all three kingdoms into a single non-redundant input set (~26 million protein-coding genes) ready for MMseqs2 easy-linclust clustering at 50%, 90% or 95% amino acid identity to build the VMGC-50/90/95 catalogues.

## Related tools

- **Prodigal 2.6.3** (Predicts protein-coding genes on each genome in anonymous metagenomic mode (-p meta), producing .faa, .ffn and .gff outputs)
- **GNU parallel 20201122** (Parallelizes Prodigal gene prediction across all genomes in the collection) — https://github.com/RChGO/VMGC
- **MMseqs2 easy-linclust** (Consumes the concatenated predicted proteins to cluster them at 50%, 90% and 95% amino acid identity into the VMGC gene catalogues) — https://github.com/RChGO/VMGC

## Examples

```
find genomes/*.fasta | parallel -j 20 'prodigal -i {} -p meta -a {.}.faa -d {.}.ffn -o {.}.gff'
```

## Evaluation signals

- vmgc90_gene_catalogue.faa (and the VMGC-50/95 counterparts) exist after clustering the predicted proteins with MMseqs2 easy-linclust
- VMGC-90 catalogue size is consistent with the reported ~1,415,799 non-redundant genes (with 595,219 for VMGC-50 and 1,786,695 for VMGC-95) from ~26 million input genes
- Each representative sequence in the catalogue is traceable to its cluster members in the MMseqs2 cluster-to-member mapping file
- Every input genome produced a *.faa, *.ffn and *.gff triplet with gene counts plausible for the genome size (e.g. median viral genome size of 33.5 kbp)
- Predicted proteins from all three kingdoms (786 prokaryotic species, 11 fungal species, 4,263 vOTUs) are represented in the concatenated input set

## Limitations

- Gene predictions depend on genome quality: only MAGs meeting ≥50% completeness and <5% contamination (revised MIMAG criteria) were carried forward, so genes from low-quality assemblies are excluded.
- VMGC gene content is incomplete for understudied populations — microbial sequences are especially lacking for vaginal samples from non-western populations, and fungi from Cladosporium and Malassezia are missing.
- Functional exploitation of predicted genes is limited: only ~67.4% of protein-coding genes could be accommodated in KEGG and 5.2% in CAZy, and many phage genes remain functionally uncharacterized.

## Evidence

- [abstract] we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity: "we clustered approximately 26 million protein-coding genes from all prokaryotic, fungal and viral genomes at 50% (VMGC-50), 90% (VMGC-90) and 95% (VMGC-95) amino acid identity"
- [methods] Run Prodigal 2.6.3 in anonymous metagenomic mode (-p meta) on every genome in parallel with GNU parallel to predict protein-coding genes: [evidence span withheld — anchor status: not_found]
- [readme] The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006: "The microbial sequence data for VMGC is stored at https://zenodo.org/records/10457006, including the data presented in the table below."
- [abstract] yielded 18,570 prokaryotic genomes with ≥50% completeness and <5% contamination: "Applying quality control, metagenomic assembly and both single- and multi-coverage metagenomic binning processes in each sample"
