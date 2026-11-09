---
name: kmer-minimizer-taxonomic-classification
description: Use when classifying metagenome-assembled genomes (MAGs) or genome FASTAs
  up to strain/species/genus level without alignment, especially when marker-gene
  tools like GTDBtk or CAMITAX are too slow or cannot provide strain-level labels.
when_to_use_negative:
- Input is reads or raw metagenomic data rather than assembled bins/MAGs — kMetaShot
  classifies bins, not shotgun reads
- A strain-level resolution is not needed and marker-gene alignment tools are already
  part of the pipeline and sufficient
- Memory or storage budget is below the reference requirement (reference needs ~22Gb
  storage and at least 32Gb RAM to load)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: Implements the k-mer/minimizer counting classifier; installed via conda bioconda
    and run as kMetaShot_classifier_NV.py
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Marker-gene-based benchmarking comparator (v1.0.2); classifies only at genus/species
    level, outputs converted to NCBI taxonomy via ncbi_vs_gtdb_r89 tables
- name: CAMITAX
  role: Marker-gene/alignment-based benchmarking comparator (v0.7.0; Nextflow v21.04.0)
- name: conda
  role: Installation channel for kMetaShot from bioconda
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:e1700341c641a237ebe523af2c5d1b11da23399b8190c4cba4f94ee11c2d0a4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - GTDBtk
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kmer-minimizer-taxonomic-classification

## Summary

An alignment-free taxonomic classification approach that builds a concise k-mer/minimizer representation of reference prokaryotic genomes and classifies MAGs (up to strain level) by shredding query sequences into minimizer sets and querying the reference storage matrix, as implemented in kMetaShot.

## When to use

Use when classifying metagenome-assembled genomes (MAGs) or genome FASTAs up to strain/species/genus level without alignment, especially when marker-gene tools like GTDBtk or CAMITAX are too slow or cannot provide strain-level labels.

## When NOT to use

- Input is reads or raw metagenomic data rather than assembled bins/MAGs — kMetaShot classifies bins, not shotgun reads
- A strain-level resolution is not needed and marker-gene alignment tools are already part of the pipeline and sufficient
- Memory or storage budget is below the reference requirement (reference needs ~22Gb storage and at least 32Gb RAM to load)

## Inputs

- Directory of MAG/bin FASTA files (.fa, .fasta, .fna, .fa.gz, .fasta.gz, .fna.gz) or a multi-fasta with one header per bin/MAG
- HDF5 kMetaShot reference file (e.g. kMetaShot_bacteria_archaea.h5, ~22Gb, RefSeq bacteria+archaea)
- NCBI Taxonomy labels (baked into the reference; no conversion table needed)

## Outputs

- Per-MAG taxonomic assignments at strain, species, and genus levels (NCBI-based)
- ass2ref reliability value per classification (ratio of observed to expected taxa minimizers)
- Classified/unclassified counts usable for sensitivity, precision, FPR, balanced accuracy, and F1 evaluation

## How to apply

Install kMetaShot 2.0 from bioconda (kmetashot=2.0 with python=3.9) and download the HDF5 reference built from RefSeq Bacterial and Archaeal genomes (Release 213), where nonredundant minimizers per genome are defined by k-mer/minimizer counting, redundant minimizers shared across genera are discarded, and strain refinement uses pairwise ANI (Mash) with DBSCAN clustering at 97%. Run kMetaShot_classifier_NV.py on a directory of bin FASTAs (or a multi-fasta with one header per MAG) against the reference with parallel processes (-p) and an ass2ref reliability filter ranging 0–1 (0.2 was used for CAMI II benchmarks); ass2ref is the ratio between observed and expected minimizers for the assigned strain. The classifier shreds the query into minimizer sets, queries the storage matrix with a prevalence-based approach, and returns strain/species/genus taxonomy per bin. Rationale: since HMP taxonomy is NCBI-based, no taxonomy conversion is needed for kMetaShot results, whereas GTDBtk outputs require ncbi_vs_gtdb_r89 conversion tables.

## Related tools

- **kMetaShot** (Implements the k-mer/minimizer counting classifier; installed via conda bioconda and run as kMetaShot_classifier_NV.py) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Marker-gene-based benchmarking comparator (v1.0.2); classifies only at genus/species level, outputs converted to NCBI taxonomy via ncbi_vs_gtdb_r89 tables)
- **CAMITAX** (Marker-gene/alignment-based benchmarking comparator (v0.7.0; Nextflow v21.04.0))
- **conda** (Installation channel for kMetaShot from bioconda)

## Examples

```
conda create --name kmetashot kmetashot=2.0=pyh7e72e81_1 python=3.9 -c bioconda; kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_bacteria_archaea.h5 -p 10 -o out_dir -a 0.2
```

## Evaluation signals

- Run kMetaShot_test.py against the downloaded reference HDF5 to verify installation before classification
- On the 939 HMP genomes, expect ~926 classified and 770/830/872 correct at strain/species/genus (sensitivity ~82.0%/89.2%/92.9%)
- Balanced accuracy should fall in the 90.8–95.4% range and FPR in the 0.2–1.9% range on HMP-like genome sets
- Verify ass2ref values lie in [0,1] and that classifications below the chosen threshold (e.g. 0.2) are dropped as unreliable
- Unclassified bins should be rare (only 11 strains and 2 species unclassified in the HMP benchmark)

## Limitations

- Reference loading in memory is the CPU/memory peak and requires at least 32Gb RAM; the HDF5 reference requires about 22Gb of storage
- Strain-level resolution depends on the strain refinement procedure; NCBI-based reference taxonomy may differ from GTDB taxonomy used by comparators
- Docker execution requires --shm-size=22g for the container to run properly

## Evidence

- [abstract] we present here kMetaShot, a taxonomy classifier based on k-mer/minimizer counting: "we present here kMetaShot, a taxonomy classifier based on k-mer/minimizer counting"
- [methods] shredding query data in minimizer sets and then queries the storage matrix: "shredding query data in minimizer sets and then queries the storage matrix to retrieve related taxonomic information"
- [methods] the ass2ref parameter (i.e. ranging between 0 and 1) was implemented as an optional filter: "the ass2ref parameter (i.e. ranging between 0 and 1) was implemented as an optional filter to drop out unreliable classifications"
- [methods] Minimizers in common to different genera were flagged as redundant and discarded: "Minimizers in common to different genera were flagged as redundant and discarded"
- [readme] kMetaShot requires a reference file available at these Zenodo links: "kMetaShot requires a reference file available at these Zenodo links: 
1. [2nd kMetaShot reference release](https://zenodo.org/records/17375120/files/kMetaShot_bacteria_archaea_2025-05-22.h5)"
- [results] It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively: "It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively"
