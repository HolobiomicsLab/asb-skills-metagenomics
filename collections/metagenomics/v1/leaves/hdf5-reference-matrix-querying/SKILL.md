---
name: hdf5-reference-matrix-querying
description: Use when finished MAG FASTA bins (or a multi-fasta with one header per
  MAG) need NCBI-taxonomic classification up to strain level without alignment, by
  matching their k-mer/minimizer profiles against an HDF5 reference storage matrix.
when_to_use_negative:
- The query is raw sequencing reads rather than binned MAGs — the classifier is designed
  for bins/MAGs and multi-fasta MAG collections
- The reference HDF5 has not been downloaded or cannot fit in memory (kMetaShot requires
  at least 32 Gb RAM to load the reference)
- Viruses/eukaryotes are the classification target — the shipped references represent
  prokaryotic (bacterial and archaeal) RefSeq genomes only
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: kMetaShot
  role: Provides the reference generator (HDF5 storage matrix) and the classifier
    module (kMetaShot_classifier_NV.py) that shreds MAGs into minimizers, queries
    the matrix, and applies prevalence-based assignment with the ass2ref filter
  repo: github.com/gdefazio/kMetaShot
- name: conda
  role: Installation channel (bioconda) for kMetaShot 2.0 with Python 3.9
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:54c730fc4640d55f30d1223079ed6548ee57a2719ff1914f73a32ee1cd0296d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# hdf5-reference-matrix-querying

## Summary

Query a pre-built HDF5 kMetaShot reference storage matrix with q-minimizer sets shredded from MAG FASTA bins, retrieving strain/genus taxonomic identifiers and assigning taxonomy via a prevalence-based approach with an ass2ref reliability filter.

## When to use

Use when finished MAG FASTA bins (or a multi-fasta with one header per MAG) need NCBI-taxonomic classification up to strain level without alignment, by matching their k-mer/minimizer profiles against an HDF5 reference storage matrix.

## When NOT to use

- The query is raw sequencing reads rather than binned MAGs — the classifier is designed for bins/MAGs and multi-fasta MAG collections
- The reference HDF5 has not been downloaded or cannot fit in memory (kMetaShot requires at least 32 Gb RAM to load the reference)
- Viruses/eukaryotes are the classification target — the shipped references represent prokaryotic (bacterial and archaeal) RefSeq genomes only

## Inputs

- Directory of MAG FASTA bin files (.fa, .fasta, .fna, .fa.gz, .fasta.gz, .fna.gz) or a multi-fasta file with one header per bin/MAG
- HDF5 kMetaShot reference file (.h5), e.g. kMetaShot_bacteria_archaea_2025-05-22.h5 from Zenodo record 17375120 (~22 Gb)

## Outputs

- kmetashot_classifications.csv with one line per classified MAG and 11 fields: bin name, ass2ref, taxid, taxonomy from species to kingdom, assigned microorganism name

## How to apply

Load the ~22 Gb HDF5 kMetaShot reference (prokaryotic RefSeq bacteria and archaea) with the classifier module, then shred each MAG into q-minimizers and query the storage matrix slots to retrieve related strain or genus taxonomic information. Assignment follows a prevalence-based approach: the most abundant genus is fixed first, then the most abundant species within that genus, and finally the strain maximizing the ass2ref ratio (the ratio of observed to expected taxa minimizers). Apply the optional ass2ref filter (range 0–1; a threshold of 0.2 was used for CAMI II benchmarks, the README example uses 0.1) to drop unreliable strain/species classifications. Run kMetaShot_classifier_NV.py with -b (bins dir or multi-fasta), -r (HDF5 reference), -p parallel processes, -o output dir, and -a ass2ref threshold; collect one line per classified MAG in kmetashot_classifications.csv.

## Related tools

- **kMetaShot** (Provides the reference generator (HDF5 storage matrix) and the classifier module (kMetaShot_classifier_NV.py) that shreds MAGs into minimizers, queries the matrix, and applies prevalence-based assignment with the ass2ref filter) — github.com/gdefazio/kMetaShot
- **conda** (Installation channel (bioconda) for kMetaShot 2.0 with Python 3.9)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_reference/kMetaShot_bacteria_archaea.h5 -p 10 -o output_dir -a 0.1
```

## Evaluation signals

- kMetaShot_test.py -r /path/to/kMetaShot_reference.h5 completes without error before classification
- kmetashot_classifications.csv exists and every line contains the 11 declared fields with an ass2ref value in the range 0–1
- ass2ref filter threshold is within [0, 1]; unreliable strain/species classifications below the threshold (e.g. 0.2) are dropped from output
- When running as a Docker container, --shm-size=22g is supplied so the reference loads correctly
- HMP benchmark sanity: classification sensitivity expected around 82.0%, 89.2%, and 92.9% at strain, species, and genus levels with balanced accuracy between 90.8% and 95.4%

## Limitations

- kMetaShot requires at least 32 Gb of memory to load the reference, and high parallelism (-p) further increases RAM usage
- The kMetaShot reference requires about 22 Gb of storage and represents prokaryotic RefSeq genomes (bacteria and archaea) only
- ass2ref filtering drops unreliable classifications entirely, so low-ass2ref MAGs may be left unclassified at strain/species level; in the HMP benchmark only 11 strains and 2 species were not classified at all

## Evidence

- [methods] storage matrix querying for taxonomic classification: "shredding query data in minimizer sets and then queries the storage matrix to retrieve related taxonomic information"
- [methods] prevalence-based classification: "classification is achieved using a prevalence-based approach"
- [results] ass2ref reliability filter: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [readme] HDF5 reference size: "kMetaShot reference represents prokaryotic RefSeq genomes and requires about 22Gb of storage."
- [readme] ass2ref parameter definition: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
