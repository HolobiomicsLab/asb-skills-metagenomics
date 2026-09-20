---
name: kmer-minimizer-counting
description: Use when one or more metagenome-assembled genome FASTA bins (or a multi-fasta
  of MAGs) need fast, alignment-free taxonomic assignment at strain, species, or genus
  level with a measurable reliability score.
when_to_use_negative:
- When input is raw metagenomic reads rather than assembled MAG bins — kMetaShot works
  directly on the binning output
- When marker-gene/alignment-based assignment is required (e.g. GTDBtk or CAMITAX
  gold-standard workflows with marker gene identification and sequence alignments)
- When less than ~32 Gb RAM is available, since the reference loading in memory requires
  at least 32 Gb
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3274
tools:
- name: kMetaShot
  role: Alignment-free taxonomy classifier performing k-mer/minimizer counting of
    MAGs against an HDF5 reference of prokaryotic genomes
  repo: https://github.com/gdefazio/kMetaShot
- name: conda
  role: Installation channel for kMetaShot 2.0 from bioconda
- name: Python 3.9
  role: Runtime environment for the kMetaShot classifier scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:496f95776475dfe96e1ec518504da54da75563ea2fdddd2a5f684d13c533444a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - Python
  - Python 3.9
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3274
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kmer-minimizer-counting

## Summary

An alignment-free taxonomic classification strategy that represents MAGs and reference prokaryotic genomes as k-mer/minimizer sets, then assigns taxonomy up to strain level by querying a storage matrix and applying a prevalence-based rule with an ass2ref reliability filter (threshold e.g. 0.2).

## When to use

Use when one or more metagenome-assembled genome FASTA bins (or a multi-fasta of MAGs) need fast, alignment-free taxonomic assignment at strain, species, or genus level with a measurable reliability score.

## When NOT to use

- When input is raw metagenomic reads rather than assembled MAG bins — kMetaShot works directly on the binning output
- When marker-gene/alignment-based assignment is required (e.g. GTDBtk or CAMITAX gold-standard workflows with marker gene identification and sequence alignments)
- When less than ~32 Gb RAM is available, since the reference loading in memory requires at least 32 Gb

## Inputs

- Directory of MAG FASTA bin files (.fa, .fasta, .fna, .fa.gz, .fasta.gz, .fna.gz) or a multi-fasta with one header per MAG
- HDF5 kMetaShot reference file (~22 Gb, e.g. kMetaShot_bacteria_archaea_2025-05-22.h5 built from RefSeq bacterial and archaeal genomes)
- ass2ref threshold (float, 0-1; default 0; 0.2 used in benchmarks)
- Number of parallel child processes

## Outputs

- kmetashot_classifications.csv with one line per classified MAG and 11 fields: bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name
- Per-MAG taxonomic assignment up to strain level with reliability score

## How to apply

Install kMetaShot 2.0 from bioconda with Python 3.9 and download the pre-built HDF5 reference (bacteria and archaea, RefSeq 2025/05/22, ~22 Gb, Zenodo record 17375120). Verify the installation with kMetaShot_test.py against the reference .h5. The classifier shreds each MAG into q-minimizers and queries the HDF5 storage matrix to retrieve candidate strain or genus identifiers, then applies a prevalence-based assignment: most abundant genus, then most abundant species, then the strain maximizing the ass2ref ratio (observed vs expected taxa minimizers). Run kMetaShot_classifier_NV.py with -b (bins directory or multi-fasta), -r (reference.h5), -p (parallel processes, e.g. 10), -o (output directory), and an -a ass2ref threshold between 0 and 1 (0.2 was set for all CAMI II benchmark datasets) to drop unreliable strain/species classifications. Judge success by the output CSV containing one line per classified MAG with 11 fields (bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name) and ass2ref values in 0-1.

## Related tools

- **kMetaShot** (Alignment-free taxonomy classifier performing k-mer/minimizer counting of MAGs against an HDF5 reference of prokaryotic genomes) — https://github.com/gdefazio/kMetaShot
- **conda** (Installation channel for kMetaShot 2.0 from bioconda)
- **Python 3.9** (Runtime environment for the kMetaShot classifier scripts)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea_2025-05-22.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- kmetashot_classifications.csv exists and every line contains the 11 declared fields (bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name)
- Every reported ass2ref value lies in the range 0-1; MAGs below the chosen threshold (e.g. 0.2) are dropped as unreliable
- Installation verified with kMetaShot_test.py -r /path/to/kMetaShot_reference.h5 before classification
- Benchmark expectation from the paper: inferred sensitivity around 82.0%, 89.2%, and 92.9% at strain, species, and genus levels on HMP genomes (926 of 939 classified)
- Reference file loads successfully requiring ~22 Gb storage / at least 32 Gb memory; Docker runs need --shm-size=22g

## Limitations

- The reference loading in memory requires at least 32 Gb; high parallelism (-p) increases RAM usage
- ass2ref parameter setting must be chosen by the user; an inappropriate threshold drops or admits unreliable strain/species classifications
- Only 11 strains and 2 species among the 939 HMP genomes were not classified at all; strain-level sensitivity (~82.0%) is lower than genus-level (~92.9%)
- Classification is restricted to bacteria and archaea represented in the RefSeq-based reference

## Evidence

- [readme] alignment-free classifier based on k-mer/minimizer counting: "we present **kMetaShot**, an alignment-free taxonomic classifier based on k-mer/minimizer counting."
- [methods] ass2ref reliability filter: "the ass2ref parameter (i.e. ranging between 0 and 1) was implemented as an optional filter to drop out unreliable classifications"
- [results] ass2ref threshold 0.2 in benchmarks: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [methods] prevalence-based classification mechanism: "classification is achieved using a prevalence-based approach"
- [readme] CLI usage and parameters: "kMetaShot_classifier_NV.py 
                -b bins/
                -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5',
                -p 10
                -o output_dir"
- [readme] ass2ref definition: "ass2ref is a ratio between the number of MAG minimizers
                        and the reference minimizers related to the assigned strain"
- [discussion] memory requirement: "the kMetaShot peaks in memory and CPU correspond to the reference loading in memory, requiring at least 32Gb."
