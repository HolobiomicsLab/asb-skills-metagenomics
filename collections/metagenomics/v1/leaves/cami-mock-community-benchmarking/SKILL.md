---
name: cami-mock-community-benchmarking
description: Use when a taxonomy classifier must be evaluated on MAGs from simulated
  multi-sample communities (e.g. CAMI II GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio)
  against an expected taxonomy at strain/species/genus rank.
when_to_use_negative:
- Evaluating classifiers on real (non-simulated) microbiome data where no expected
  CAMI taxonomy (CE) exists to score against
- Read-level or contig-level taxonomic profiling — this skill requires binned MAGs
  (MetaBAT2 output with minimum contig size 1,500) as input
- Classifying eukaryotic or viral genomes — the kMetaShot reference covers only prokaryotic
  RefSeq bacteria/archaea
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: kMetaShot
  role: Primary classifier under benchmark; run via kMetaShot_classifier_NV.py with
    ass2ref 0.2 on all CAMI II datasets
  repo: https://github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Comparator classifier (v1.0.2); output labels converted from GTDB to NCBI
    taxonomy
- name: CAMITAX
  role: Comparator classifier (v0.7.0, run through Nextflow v21.04.0)
- name: MetaBAT2
  role: Bins provided contigs per sample (--maxP 99 --minS 98 -m 1,500) to obtain
    the MAGs fed to all classifiers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:4fcb4c14cd265c2ed6fe9058a12ffaa9038cbcc5d63dfa3271cfee68279dcdd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gdefazio/kMetaShot
  tools:
  - CAMITAX
  - GTDBtk
  - MetaBAT2
  - kMetaShot
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cami-mock-community-benchmarking

## Summary

Benchmark taxonomic classifiers (kMetaShot, GTDBtk, CAMITAX) on CAMI II multi-sample mock community datasets by binning contigs with MetaBAT2, classifying the resulting MAGs, and scoring sensitivity, precision, FPR, balanced accuracy and F1-score per rank against the expected CAMI taxonomy.

## When to use

Use when a taxonomy classifier must be evaluated on MAGs from simulated multi-sample communities (e.g. CAMI II GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio) against an expected taxonomy at strain/species/genus rank.

## When NOT to use

- Evaluating classifiers on real (non-simulated) microbiome data where no expected CAMI taxonomy (CE) exists to score against
- Read-level or contig-level taxonomic profiling — this skill requires binned MAGs (MetaBAT2 output with minimum contig size 1,500) as input
- Classifying eukaryotic or viral genomes — the kMetaShot reference covers only prokaryotic RefSeq bacteria/archaea

## Inputs

- CAMI II multi-sample mock community contig datasets (GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio conditions, 10 samples per condition)
- Expected CAMI taxonomies (CE) per sample
- kMetaShot HDF5 prokaryotic reference (e.g. kMetaShot_bacteria_archaea_2025-05-22.h5)
- ncbi_vs_gtdb_r89 conversion tables (GTDB release89)

## Outputs

- Per-tool taxonomic classifications (CO) of all inferred MAGs at strain, species, and genus ranks
- Per-tool accuracy metrics table (cami2_tool_benchmark_accuracy.csv) with sensitivity, precision, FPR, BA, and F1-score per rank
- Median/IQR summaries of metrics across samples per condition

## How to apply

Bin the provided CAMI II contigs per sample with MetaBAT2 (v2:2.15) using --maxP 99 --minS 98 -m 1,500 to obtain MAGs. Classify all inferred MAGs with each tool: kMetaShot (kMetaShot_classifier_NV.py against the HDF5 prokaryotic reference, ass2ref = 0.2 for all CAMI II datasets), GTDBtk v1.0.2, and CAMITAX v0.7.0 via Nextflow v21.04.0. Convert GTDBtk labels from GTDB taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables from data.gtdb.ecogenomic.org release89 so all tools are comparable. Compare observed classifications (CO) against the expected CAMI taxonomies (CE) and compute sensitivity, precision, false-positive rate (FPR), balanced accuracy (BA), and F1-score per tool and rank level. Aggregate metrics across the 10 samples per GI/Air condition into a per-tool accuracy table (cami2_tool_benchmark_accuracy.csv), reported as medians and IQRs.

## Related tools

- **kMetaShot** (Primary classifier under benchmark; run via kMetaShot_classifier_NV.py with ass2ref 0.2 on all CAMI II datasets) — https://github.com/gdefazio/kMetaShot
- **GTDBtk** (Comparator classifier (v1.0.2); output labels converted from GTDB to NCBI taxonomy)
- **CAMITAX** (Comparator classifier (v0.7.0, run through Nextflow v21.04.0))
- **MetaBAT2** (Bins provided contigs per sample (--maxP 99 --minS 98 -m 1,500) to obtain the MAGs fed to all classifiers)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_bacteria_archaea_2025-05-22.h5 -p 10 -o output_dir -a 0.2
```

## Evaluation signals

- Reproduce the reported ranking: kMetaShot attains top precision at species and genus level, and highest sensitivity in GI-Illumina (species median 69.75, IQR 3.86; genus median 98.33, IQR 4.989) and GI-PacBio
- kMetaShot BA and F1-score exceed the other tools at genus level across all datasets
- GTDBtk and CAMITAX produce no strain-level classifications (their pipelines stop at species/genus)
- All GTDBtk labels are mapped to NCBI taxonomy via ncbi_vs_gtdb_r89 before metric computation, confirming cross-tool comparability
- Metrics aggregate to a per-tool table (cami2_tool_benchmark_accuracy.csv) covering 10 samples per GI/Air condition with median/IQR statistics

## Limitations

- GTDBtk and CAMITAX provide no strain-level output, so strain-rank accuracy comparisons are only possible for kMetaShot
- kMetaShot reference loading requires at least 32 Gb RAM and ~22 Gb storage; high parallelism (-p) increases RAM usage
- ass2ref filtering (0.2 here) drops unreliable classifications and shifts the sensitivity/precision trade-off; other values will change results
- GTDBtk results require GTDB-to-NCBI taxonomy conversion, introducing an additional mapping step into the comparison

## Evidence

- [methods] CAMI II binning parameters: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [results] ass2ref threshold: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [methods] GTDB to NCBI conversion: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [results] GI-Illumina result: "kMetaShot obtained the highest sensitivity at species and genus levels in GI-Illumina (median 69.75 IQR 3.86, median 98.33 IQR 4.989)"
- [readme] README usage with ass2ref flag: "Classification filtering based on ass2ref parameter ranging between 0 and 1. Default 0."
