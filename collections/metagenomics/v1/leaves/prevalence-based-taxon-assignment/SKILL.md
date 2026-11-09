---
name: prevalence-based-taxon-assignment
description: Use when assigning taxonomy to metagenome-assembled genome (MAG) FASTA
  bins at strain-level resolution with k-mer/minimizer counting against the kMetaShot
  HDF5 reference, and when a reliability ratio (ass2ref) is needed to filter questionable
  strain/species calls.
when_to_use_negative:
- The task requires marker-gene identification or sequence-alignment-based classification
  (e.g. GTDBtk or CAMITAX workflows) rather than alignment-free k-mer/minimizer counting.
- The target taxa are not represented in the prokaryotic (bacteria and archaea) RefSeq
  kMetaShot reference, e.g. eukaryotic or viral MAGs.
- Only raw reads need classification — kMetaShot works directly on MAG bins/binned
  contigs, not FASTQ reads.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: kMetaShot
  role: Classifier that shreds MAGs into q-minimizers, queries the storage matrix,
    and applies prevalence-based strain/species/genus assignment with the ass2ref
    reliability filter
  repo: https://github.com/gdefazio/kMetaShot
- name: kMetaShot 2.0
  role: Installed version (kmetashot=2.0=pyh7e72e81_1) providing kMetaShot_classifier_NV.py
    for the prevalence-based classification
  repo: https://github.com/gdefazio/kMetaShot
- name: conda
  role: Installs kMetaShot 2.0 with Python 3.9 from the bioconda channel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:afc870b56798b7754d45b2523d7165078fcd3f93494158900c7eaa6f61bfb516
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - Python
  - conda
  - kMetaShot
  - kMetaShot 2.0
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# prevalence-based-taxon-assignment

## Summary

kMetaShot's mechanism for assigning a MAG taxonomy up to the strain level: after shredding the MAG into q-minimizers and querying the HDF5 storage matrix, taxonomy is resolved with a prevalence-based approach (most abundant genus, then species, then the strain maximizing the ass2ref ratio), with an optional ass2ref filter (range 0–1) to drop unreliable classifications.

## When to use

Use when assigning taxonomy to metagenome-assembled genome (MAG) FASTA bins at strain-level resolution with k-mer/minimizer counting against the kMetaShot HDF5 reference, and when a reliability ratio (ass2ref) is needed to filter questionable strain/species calls.

## When NOT to use

- The task requires marker-gene identification or sequence-alignment-based classification (e.g. GTDBtk or CAMITAX workflows) rather than alignment-free k-mer/minimizer counting.
- The target taxa are not represented in the prokaryotic (bacteria and archaea) RefSeq kMetaShot reference, e.g. eukaryotic or viral MAGs.
- Only raw reads need classification — kMetaShot works directly on MAG bins/binned contigs, not FASTQ reads.

## Inputs

- MAG FASTA bins in a directory (with .fa/.fasta/.fna/.fa.gz/.fasta.gz/.fna.gz extensions) or a multi-FASTA with one header per MAG
- kMetaShot HDF5 reference (e.g. kMetaShot_bacteria_archaea_2025-05-22.h5, ~22 Gb)
- ass2ref threshold (float, 0–1)

## Outputs

- kmetashot_classifications.csv with one line per classified MAG and 11 fields (bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name)

## How to apply

Shred each MAG bin into q-minimizers and query the kMetaShot storage matrix (HDF5 reference built from RefSeq bacterial/archaeal CDSs and ncRNAs) to retrieve strain or genus identifiers for each minimizer. Apply the prevalence-based classification: assign the most abundant genus, then the most abundant species within it, then the strain that maximizes the ass2ref ratio (observed vs expected taxa minimizers). Apply the ass2ref filter as a threshold between 0 and 1 (e.g. 0.2, as used for all CAMI II benchmark datasets) to drop unreliable strain/species classifications; the default in kMetaShot_classifier_NV.py is 0. Collect results in the output CSV with 11 fields (bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name). Rationale: the prevalence-based approach plus the ass2ref reliability ratio underpin kMetaShot's reported accuracy (sensitivity ~82.0% strain, ~89.2% species, ~92.9% genus on HMP genomes) while remaining alignment-free and fast.

## Related tools

- **kMetaShot** (Classifier that shreds MAGs into q-minimizers, queries the storage matrix, and applies prevalence-based strain/species/genus assignment with the ass2ref reliability filter) — https://github.com/gdefazio/kMetaShot
- **kMetaShot 2.0** (Installed version (kmetashot=2.0=pyh7e72e81_1) providing kMetaShot_classifier_NV.py for the prevalence-based classification) — https://github.com/gdefazio/kMetaShot
- **conda** (Installs kMetaShot 2.0 with Python 3.9 from the bioconda channel)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- kmetashot_classifications.csv exists and every line contains the 11 declared fields (bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name)
- Every ass2ref value in the output is within the declared 0–1 range, and classifications below the chosen threshold (e.g. 0.2) are dropped
- Taxonomic labels (genus, species, strain) are consistent with NCBI taxonomy taxids returned by the storage matrix query
- Installation verified beforehand with kMetaShot_test.py -r /path/to/kMetaShot_reference.h5
- Benchmark reproduction: with ass2ref = 0.2, sensitivity on HMP-like genomes should approach ~82.0% at strain, ~89.2% at species, ~92.9% at genus level

## Limitations

- The kMetaShot reference requires about 22 Gb of storage and peaks in memory at reference loading, requiring at least 32 Gb of RAM
- The ass2ref threshold trades sensitivity against reliability: too high a value drops valid strain/species calls, while the default of 0 retains unreliable classifications
- Only 11 strains and 2 species among the 939 HMP genomes were left unclassified, but taxa absent from the prokaryotic RefSeq reference cannot be classified at all

## Evidence

- [results] kMetaShot implements ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications.: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [methods] Prevalence-based classification approach in kMetaShot methods.: "classification is achieved using a prevalence-based approach"
- [methods] Storage matrix querying is the core classification mechanism.: "shredding query data in minimizer sets and then queries the storage matrix to retrieve related taxonomic information"
- [readme] ass2ref parameter description and default from the classifier usage.: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
- [results] ass2ref of 0.2 was set for all CAMI II datasets.: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
