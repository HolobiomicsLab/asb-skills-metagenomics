---
name: classification-metric-computation
description: Use when comparing taxonomic classification tools on MAGs or genomes
  with a known ground-truth taxonomy (e.g., CAMI II expected taxonomies, real mock
  community composition), and you need per-rank accuracy metrics.
when_to_use_negative:
- There is no ground-truth/expected taxonomy for the input genomes, so CO-vs-CE sensitivity/precision
  cannot be defined
- The task is to produce classifications rather than evaluate them (e.g., run kMetaShot_classifier_NV.py
  on novel MAGs); use the classification skill instead
- Tool outputs use different taxonomy backbones (e.g., GTDB vs NCBI) and no conversion
  table (like ncbi_vs_gtdb_r89) is available to harmonize labels before comparison
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: kMetaShot
  role: Classifier under evaluation; its ass2ref-filtered (0.2) classifications are
    compared against expected taxonomies to compute the metrics
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparator tool (v1.0.2); its GTDB-taxon labels are converted to NCBI taxonomy
    before metric computation
- name: CAMITAX
  role: Comparator tool (v0.7.0 via Nextflow v21.04.0); its classifications are evaluated
    against the same expected taxonomies
- name: Mash
  role: Computes pairwise ANI values used to refine strain assignments (highest ANI
    >= 97%) for species lacking strain annotation in the real mock community
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:2165a2029e66c785a4cdcaa91e866f506887e1a4db7678e2bff223156fb83eb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - Mash
  - MegaHIT
  - Sickle
  - kMetaShot
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# classification-metric-computation

## Summary

Compute benchmark accuracy metrics (sensitivity, precision, FPR, balanced accuracy, F1-score) for genome/MAG taxonomic classifiers by comparing observed classifications (CO) against expected taxonomies (CE) per rank level (strain, species, genus), as done in the kMetaShot paper against GTDBtk and CAMITAX.

## When to use

Use when comparing taxonomic classification tools on MAGs or genomes with a known ground-truth taxonomy (e.g., CAMI II expected taxonomies, real mock community composition), and you need per-rank accuracy metrics.

## When NOT to use

- There is no ground-truth/expected taxonomy for the input genomes, so CO-vs-CE sensitivity/precision cannot be defined
- The task is to produce classifications rather than evaluate them (e.g., run kMetaShot_classifier_NV.py on novel MAGs); use the classification skill instead
- Tool outputs use different taxonomy backbones (e.g., GTDB vs NCBI) and no conversion table (like ncbi_vs_gtdb_r89) is available to harmonize labels before comparison

## Inputs

- Observed taxonomic classifications (CO) per MAG/tool, e.g., kMetaShot, GTDBtk, and CAMITAX outputs at strain/species/genus ranks
- Expected (ground-truth) taxonomies (CE), e.g., CAMI II mock community taxonomies or the known real mock community composition (48 bacteria and 16 archaea)
- GTDB-to-NCBI conversion tables (ncbi_vs_gtdb_r89, release89)
- Pairwise ANI values from Mash against RefSeq genomes of observed strains (for strain refinement)

## Outputs

- Per-tool, per-rank accuracy table with sensitivity, precision, FPR, balanced accuracy (BA), and F1-score (e.g., cami2_tool_benchmark_accuracy.csv)
- Median and IQR summaries per dataset/condition (e.g., GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio)
- Squared ANI heat maps of MAGs vs. reference strain genomes for strain-level validation

## How to apply

First harmonize taxonomic nomenclature across tools: convert GTDBtk labels from GTDB taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables from data.gtdb.ecogenomic.org release89, since kMetaShot uses NCBI taxonomy. Then compare observed classifications (CO) against expected/ground-truth taxonomies (CE) and compute sensitivity, precision, false-positive rate (FPR), balanced accuracy (BA), and F1-score per tool and per taxonomic rank level (strain, species, genus). Apply the kMetaShot classifications with the ass2ref reliability filter set to 0.2, the value used for all CAMI II datasets and the real mock benchmark. For the real mock community, handle missing strain annotation by refining strain assignments for species lacking strain annotation (47 of 64 mock genomes) using Mash ANI against RefSeq genomes of the observed strains, assigning the strain with highest ANI (>=97%). Aggregate metrics across replicates (10 samples per GI/Air condition, 20 PA-PacBio samples) into a per-tool accuracy table (e.g., cami2_tool_benchmark_accuracy.csv) and validate against reported rankings such as kMetaShot's GI-Illumina species sensitivity median of 69.75 (IQR 3.86) and no strain-level classification by GTDBtk/CAMITAX.

## Related tools

- **kMetaShot** (Classifier under evaluation; its ass2ref-filtered (0.2) classifications are compared against expected taxonomies to compute the metrics) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparator tool (v1.0.2); its GTDB-taxon labels are converted to NCBI taxonomy before metric computation)
- **CAMITAX** (Comparator tool (v0.7.0 via Nextflow v21.04.0); its classifications are evaluated against the same expected taxonomies)
- **Mash** (Computes pairwise ANI values used to refine strain assignments (highest ANI >= 97%) for species lacking strain annotation in the real mock community)

## Evaluation signals

- The computed metric table reproduces the paper's ranking: kMetaShot attains top sensitivity and precision at species and genus levels, and GTDBtk/CAMITAX show no strain-level classification
- For CAMI II GI-Illumina, kMetaShot species sensitivity should match the reported median 69.75 with IQR 3.86 (genus median 98.33, IQR 4.989)
- kMetaShot classifications must have been produced with ass2ref = 0.2, the threshold applied to all CAMI II datasets and the mock benchmark
- Metric aggregation covers the correct sample counts: 10 samples per GI/Air condition and 20 PA-PacBio samples
- For the real mock community, strain-level kMetaShot calls are evaluated both before and after the Mash ANI refinement (highest ANI >= 97%), matching the paper's reported tool ranking

## Limitations

- GTDBtk and CAMITAX cannot be assessed at the strain level because they perform no strain-level classification, limiting rank-level comparisons
- Metric values are sensitive to the ass2ref reliability filter setting; the kMetaShot paper used 0.2 for all CAMI II datasets and the mock dataset benchmark
- Cross-taxonomy comparisons require GTDB-to-NCBI label conversion (ncbi_vs_gtdb_r89); mismatches in conversion propagate into the computed metrics
- Strain-level ground truth is incomplete for real communities: 47 of 64 mock genomes lacked strain annotation and required Mash ANI-based strain refinement

## Evidence

- [methods] GTDBtk labels from GTDB taxonomy to NCBI: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [results] ass2ref filter setting for benchmarks: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [results] kMetaShot highest sensitivity in GI-Illumina: "kMetaShot obtained the highest sensitivity at species and genus levels in GI-Illumina (median 69.75 IQR 3.86, median 98.33 IQR 4.989)"
- [readme] ass2ref definition in README usage: "ass2ref is a ratio between the number of MAG minimizers and the reference minimizers related to the assigned strain"
- [results] HMP evaluation metric ranges: "BA ranged between 90.8% and 95.4% (Fig. 4B and Supplementary Table 2)"
