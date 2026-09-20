---
name: gtdb-to-ncbi-taxonomy-conversion
description: Use when GTDBtk (v1.0.2) classifications of MAGs or genomes must be compared
  against NCBI-taxonomy-based ground truth (e.g., HMP genome labels or CAMI taxonomies),
  because GTDBtk outputs use GTDB taxonomy.
when_to_use_negative:
- kMetaShot classification results — the reference uses NCBI taxonomy, so no conversion
  is needed for kMetaShot outputs
- HMP genomes labelled with NCBI-based flat-file taxonomy where GTDBtk was not run
  — labels are already NCBI-native
- Analyses comparing tools purely within the GTDB framework where NCBI nomenclature
  is not the reference standard
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: GTDBtk
  role: Source of the GTDB-taxonomy classifications (v1.0.2, genus/species level)
    that require conversion to NCBI taxonomy before benchmarking against kMetaShot
    and CAMITAX
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:9b304d3c9c4873cd03ad1ff53a12e927cace6bec3de5b245826cbf2f1a54fdfd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - Sickle
  - conda
  - kMetaShot
  - metaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# gtdb-to-ncbi-taxonomy-conversion

## Summary

Converts taxonomic labels produced by GTDBtk from the Genome Taxonomy Database (GTDB) taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables, so that classifier outputs (GTDBtk vs. kMetaShot) can be compared against NCBI-based expected taxonomies such as those of the HMP genomes and CAMI II reference taxonomies.

## When to use

Use when GTDBtk (v1.0.2) classifications of MAGs or genomes must be compared against NCBI-taxonomy-based ground truth (e.g., HMP genome labels or CAMI taxonomies), because GTDBtk outputs use GTDB taxonomy.

## When NOT to use

- kMetaShot classification results — the reference uses NCBI taxonomy, so no conversion is needed for kMetaShot outputs
- HMP genomes labelled with NCBI-based flat-file taxonomy where GTDBtk was not run — labels are already NCBI-native
- Analyses comparing tools purely within the GTDB framework where NCBI nomenclature is not the reference standard

## Inputs

- GTDBtk (v1.0.2) classification output files with GTDB taxonomy strings per genome/MAG
- ncbi_vs_gtdb_r89 conversion tables from GTDB release89 (https://data.gtdb.ecogenomic.org/releases/release89/89.0/)

## Outputs

- Per-genome/MAG taxonomic labels re-expressed in NCBI taxonomy at genus and species ranks
- Comparison-ready CO (observed) labels for computing sensitivity, precision, FPR, balanced accuracy, and F1-score against NCBI-based expected taxonomies

## How to apply

After running GTDBtk v1.0.2 on the same set of genomes/MAGs classified with kMetaShot, take the GTDBtk output taxonomic strings and map each rank (genus, species; GTDBtk provides no strain-level classification) to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables downloaded from data.gtdb.ecogenomic.org release89 (https://data.gtdb.ecogenomic.org/releases/release89/89.0/). Do this before computing sensitivity, precision, FPR, balanced accuracy, and F1-score, since the set-based metric definitions (Tp = CO∩CE, Fp = CO−(CO∩CE), Fn = CE−(CO∩CE), Tn = CT−(CO∩CE)) compare observed classifications (CO) against expected NCBI taxonomies (CE). Apply the same conversion uniformly across benchmark datasets (HMP genomes, CAMI II GI/Air/PA conditions, and the real mock community SRR606249) so that kMetaShot, GTDBtk, and CAMITAX are scored on a common taxonomy. Note that kMetaShot's own results are already NCBI-based and require no conversion.

## Related tools

- **GTDBtk** (Source of the GTDB-taxonomy classifications (v1.0.2, genus/species level) that require conversion to NCBI taxonomy before benchmarking against kMetaShot and CAMITAX)

## Evaluation signals

- Converted labels should resolve to valid NCBI taxa for the same ranks reported by GTDBtk (genus, species only — GTDBtk provides no strain level)
- After conversion, computed sensitivity for the 939 HMP genomes should reproduce the paper's reported values (~82.0% strain, ~89.2% species, ~92.9% genus for kMetaShot) under the set-based Tp/Fp/Fn/Tn definitions
- FPR values should fall in the reported 0.2%–1.9% range for the HMP benchmark
- Cross-dataset consistency: the same release89 conversion tables must be applied identically across HMP, CAMI II (GI/Air/PA), and mock community (SRR606249) benchmarks
- Spot-check that converted GTDB species/genus names match the expected NCBI taxonomies extracted from the HMP flat files or CAMI taxonomies

## Limitations

- GTDBtk classifies only at genus and species levels, so no strain-level comparison is possible even after conversion
- Conversion depends on release-specific tables (ncbi_vs_gtdb_r89); using other GTDB releases may misalign labels
- GTDB and NCBI taxonomies can differ structurally, so some GTDB labels may have no clean NCBI counterpart in the r89 tables

## Evidence

- [methods] Convert GTDBtk outputs from GTDB taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables.: "The GTDBtk results were converted from the Genome Taxonomy Database (GTDB) taxonomy to NCBI"
- [methods] Conversion tables are hosted at the GTDB release89 archive.: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [results] GTDBtk does not classify at strain level, so post-conversion benchmarks cover only genus and species ranks.: [evidence span withheld — anchor status: not_found]
- [other] kMetaShot results are already NCBI-based and need no conversion, unlike GTDBtk outputs.: [evidence span withheld — anchor status: not_found]
