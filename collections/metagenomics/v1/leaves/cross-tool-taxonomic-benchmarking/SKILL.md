---
name: cross-tool-taxonomic-benchmarking
description: Use when a newly proposed MAG/prokaryotic genome taxonomy classifier
  must be validated against established gold standards (GTDBtk, CAMITAX) on a common
  benchmark set with known NCBI taxonomic labels.
when_to_use_negative:
- The goal is functional or compositional profiling of shotgun reads rather than classification
  of assembled genomes/MAGs — kMetaShot and this benchmark target bins/MAG FASTA inputs
- Only one classifier is to be run with no comparison baseline — cross-tool benchmarking
  requires at least two tools evaluated on the identical genome set
- Expected taxonomy is not in NCBI space and no conversion table exists (GTDBtk outputs
  must be converted to NCBI via ncbi_vs_gtdb_r89 for fair comparison)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: kMetaShot
  role: Alignment-free k-mer/minimizer classifier under test; classifies MAGs up to
    strain level against the HDF5 reference
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Marker-gene/alignment-based gold-standard classifier (v1.0.2) benchmarked
    against kMetaShot at species and genus levels
- name: CAMITAX
  role: Gold-standard classifier (v0.7.0; Nextflow v21.04.0) benchmarked against kMetaShot
    at species and genus levels
- name: conda
  role: Installs kMetaShot from the bioconda channel
  repo: https://github.com/gdefazio/kMetaShot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:db6019c2db892c79975bf11b1305d8ad9093d9189c789178bd3165258120c62d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - GTDBtk
  - conda
  - kMetaShot
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-tool-taxonomic-benchmarking

## Summary

Head-to-head benchmarking of taxonomic classifiers (kMetaShot vs GTDBtk and CAMITAX) on a curated genome set such as the 939 HMP genomes, comparing strain-, species-, and genus-level classification using set-based sensitivity, precision, FPR, balanced accuracy, and F1-score.

## When to use

Use when a newly proposed MAG/prokaryotic genome taxonomy classifier must be validated against established gold standards (GTDBtk, CAMITAX) on a common benchmark set with known NCBI taxonomic labels.

## When NOT to use

- The goal is functional or compositional profiling of shotgun reads rather than classification of assembled genomes/MAGs — kMetaShot and this benchmark target bins/MAG FASTA inputs
- Only one classifier is to be run with no comparison baseline — cross-tool benchmarking requires at least two tools evaluated on the identical genome set
- Expected taxonomy is not in NCBI space and no conversion table exists (GTDBtk outputs must be converted to NCBI via ncbi_vs_gtdb_r89 for fair comparison)

## Inputs

- 939 HMP bacterial and archaeal genome FASTA files with expected NCBI taxonomic labels (strain/species/genus)
- kMetaShot HDF5 reference file (kMetaShot_reference.h5)
- GTDBtk v1.0.2 classification outputs
- ncbi_vs_gtdb_r89 conversion tables (from https://data.gtdb.ecogenomic.org/releases/release89/89.0/)

## Outputs

- Per-genome taxonomic classifications at strain, species, and genus levels for each tool
- Metrics table (sensitivity, precision, FPR, balanced accuracy, F1-score) per rank per tool
- Comparison table of kMetaShot vs GTDBtk vs CAMITAX assignments (e.g. 770/830/872 correct kMetaShot assignments at strain/species/genus)

## How to apply

Download the benchmark genomes (e.g. the 939 HMP genomes spanning 190 genera, 556 species, and 934 strains) from the NCBI Human Microbiome FTP site and extract expected NCBI taxonomic labels from the associated flat files. Run each classifier on the same FASTA inputs: kMetaShot (HDF5 reference, e.g. kMetaShot_reference.h5) with an ass2ref filter (0.2 was used for CAMI II and mock benchmarks) and GTDBtk v1.0.2, noting GTDBtk and CAMITAX classify only at genus/species level (no strain level). Convert GTDBtk output from GTDB taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables so all tools share a common label space. Compute set-based metrics per rank (Tp = CO∩CE, Fp = CO−(CO∩CE), Fn = CE−(CO∩CE), Tn = CT−(CO∩CE)) for sensitivity, precision, FPR, balanced accuracy, and F1-score, then tabulate per-tool values. The HMP reference run of kMetaShot correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels (~82.0%, 89.2%, 92.9% sensitivity), with BA 90.8–95.4% and FPR 0.2–1.9%, and both GTDBtk and CAMITAX reached lower sensitivity and balanced accuracy at species and genus levels.

## Related tools

- **kMetaShot** (Alignment-free k-mer/minimizer classifier under test; classifies MAGs up to strain level against the HDF5 reference) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Marker-gene/alignment-based gold-standard classifier (v1.0.2) benchmarked against kMetaShot at species and genus levels)
- **CAMITAX** (Gold-standard classifier (v0.7.0; Nextflow v21.04.0) benchmarked against kMetaShot at species and genus levels)
- **conda** (Installs kMetaShot from the bioconda channel) — https://github.com/gdefazio/kMetaShot

## Examples

```
conda create --name kmetashot kmetashot=2.0=pyh7e72e81_1 python=3.9 -c bioconda; conda activate kmetashot; kMetaShot_classifier_NV.py -b hmp_genomes/ -r kMetaShot_reference.h5 -p 10 -o hmp_out -a 0.2
```

## Evaluation signals

- kMetaShot on HMP genomes reproduces 926/939 classified genomes with 770, 830, and 872 correct assignments at strain, species, and genus levels
- Sensitivity matches reported ~82.0%, ~89.2%, and ~92.9% at strain, species, and genus levels
- Balanced accuracy falls within 90.8%–95.4% and FPR within 0.2%–1.9% (0.2% for strains and species, 1.9% for genus)
- GTDBtk/CAMITAX outputs are converted to NCBI taxonomy (ncbi_vs_gtdb_r89) before computing metrics, and neither reports strain-level calls
- Metrics use the paper's set-based definitions (Tp = CO∩CE, Fp = CO−(CO∩CE), Fn = CE−(CO∩CE), Tn = CT−(CO∩CE))

## Limitations

- GTDBtk and CAMITAX provide no strain-level classification, so strain-level comparison is one-sided
- GTDBtk results require conversion from GTDB to NCBI taxonomy via the ncbi_vs_gtdb_r89 tables, which may introduce label-mapping ambiguity
- kMetaShot requires a ~22Gb HDF5 reference and at least 32Gb RAM for reference loading; high process parallelism increases RAM usage
- HMP taxonomy is NCBI-based, so no conversion was needed for kMetaShot results — if the benchmark uses GTDB-space labels the setup differs

## Evidence

- [results] HMP sensitivity: "with an inferred sensitivity of around 82.0%, 89.2%, and 92.9% at strain, species, and genus levels, respectively"
- [results] HMP correct assignments: "It correctly assigned 770, 830, and 872 genomes at strain, species, and genus levels, respectively"
- [methods] GTDB conversion tables: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [results] ass2ref filter: "we implemented ass2ref (the ratio among the observed and expected taxa minimizer), as a measure of the reliability of the kMetaShot classifications."
- [readme] kMetaShot install: "conda create --name kmetashot kmetashot=2.0=pyh7e72e81_1 python=3.9 -c bioconda"
- [results] BA range: "BA ranged between 90.8% and 95.4% (Fig. 4B and Supplementary Table 2)"
