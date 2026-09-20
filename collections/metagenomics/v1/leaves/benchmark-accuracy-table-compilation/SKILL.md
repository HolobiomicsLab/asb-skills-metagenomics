---
name: benchmark-accuracy-table-compilation
description: Use when MAGs from CAMI II mock community datasets have been classified
  by multiple taxonomic tools and you need a ranked accuracy comparison (sensitivity,
  precision, BA, F1, FPR) at strain/species/genus levels against the CAMI gold standard.
when_to_use_negative:
- No expected taxonomy (CE/gold standard) is available for the sample set — sensitivity,
  precision, FPR, BA, and F1 all require ground truth and cannot be computed
- Classifications are of individual reads rather than binned MAGs — this benchmark
  operates on MetaBAT2-binned MAG FASTAs, not raw read classifiers
- Only one tool's classification output exists — the purpose of this skill is a cross-tool
  accuracy ranking (kMetaShot vs GTDBtk vs CAMITAX), which needs comparable outputs
  from ≥2 tools on identical MAGs
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: kMetaShot
  role: Benchmarked classifier under test; classifies MAGs against the HDF5 prokaryotic
    reference with the ass2ref filter set to 0.2 for all CAMI II datasets
  repo: github.com/gdefazio/kMetaShot
- name: GTDBtk
  role: Benchmarked comparator (v1.0.2); its GTDB-taxonomy labels are converted to
    NCBI taxonomy before metric computation
- name: CAMITAX
  role: Benchmarked comparator (v0.7.0, run through Nextflow v21.04.0) providing NCBI-taxonomy
    MAG classifications
- name: MetaBAT2
  role: Bins the CAMI II contigs per sample (--maxP 99 --minS 98 -m 1,500) to produce
    the shared MAG inputs for all benchmarked classifiers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:3236f89ff00bb285e37b5a43e6149ab0cb7ff0de37f69709bdff399b29ba90e8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - GTDBtk
  - MetaBAT2
  - kMetaShot
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# benchmark-accuracy-table-compilation

## Summary

Compile a per-tool taxonomic classification accuracy benchmark table by comparing observed classifications (CO) of MAG taxonomers (kMetaShot, GTDBtk, CAMITAX) against expected CAMI taxonomies (CE), computing sensitivity, precision, FPR, balanced accuracy, and F1-score per rank and aggregating them into a single CSV.

## When to use

Use when MAGs from CAMI II mock community datasets have been classified by multiple taxonomic tools and you need a ranked accuracy comparison (sensitivity, precision, BA, F1, FPR) at strain/species/genus levels against the CAMI gold standard.

## When NOT to use

- No expected taxonomy (CE/gold standard) is available for the sample set — sensitivity, precision, FPR, BA, and F1 all require ground truth and cannot be computed
- Classifications are of individual reads rather than binned MAGs — this benchmark operates on MetaBAT2-binned MAG FASTAs, not raw read classifiers
- Only one tool's classification output exists — the purpose of this skill is a cross-tool accuracy ranking (kMetaShot vs GTDBtk vs CAMITAX), which needs comparable outputs from ≥2 tools on identical MAGs

## Inputs

- Per-tool MAG classification outputs on the same binned CAMI II MAG set (kMetaShot, GTDBtk v1.0.2, CAMITAX v0.7.0)
- Expected CAMI taxonomies (CE / gold-standard profiles) for the GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio mock communities
- ncbi_vs_gtdb_r89 conversion tables from data.gtdb.ecogenomic.org release89
- MetaBAT2 MAG FASTAs binned with --maxP 99 --minS 98 -m 1,500

## Outputs

- cami2_tool_benchmark_accuracy.csv — per-tool accuracy table with sensitivity, precision, FPR, balanced accuracy, and F1-score per taxonomic rank
- Per-sample metric distributions (median and IQR) aggregated over 10 samples per GI/Air condition and 20 PA-PacBio samples

## How to apply

First ensure all tools classified the same MAG set — in the source study, contigs from each CAMI II sample were binned with MetaBAT2 (v2:2.15) using --maxP 99 --minS 98 -m 1,500, then classified with kMetaShot (ass2ref reliability filter = 0.2 for all CAMI II datasets), GTDBtk (v1.0.2), and CAMITAX (v0.7.0 via Nextflow v21.04.0). Convert GTDBtk labels from GTDB taxonomy to NCBI taxonomy using the ncbi_vs_gtdb_r89 conversion tables (data.gtdb.ecogenomic.org release89) so all outputs are on a common taxonomy. Compare observed classifications (CO) against expected CAMI taxonomies (CE) and compute sensitivity, precision, false-positive rate (FPR), balanced accuracy (BA), and F1-score per tool and per rank level (strain, species, genus). Aggregate the metrics across the 10 samples per GI/Air condition and the 20 PA-PacBio samples into a per-tool accuracy table (cami2_tool_benchmark_accuracy.csv), reporting medians and IQRs. Validation compares the compiled table against the published ranking, e.g. GI-Illumina species sensitivity median 69.75 (IQR 3.86).

## Related tools

- **kMetaShot** (Benchmarked classifier under test; classifies MAGs against the HDF5 prokaryotic reference with the ass2ref filter set to 0.2 for all CAMI II datasets) — github.com/gdefazio/kMetaShot
- **GTDBtk** (Benchmarked comparator (v1.0.2); its GTDB-taxonomy labels are converted to NCBI taxonomy before metric computation)
- **CAMITAX** (Benchmarked comparator (v0.7.0, run through Nextflow v21.04.0) providing NCBI-taxonomy MAG classifications)
- **MetaBAT2** (Bins the CAMI II contigs per sample (--maxP 99 --minS 98 -m 1,500) to produce the shared MAG inputs for all benchmarked classifiers)

## Examples

```
kMetaShot_classifier_NV.py -b bins/ -r kMetaShot_bacteria_archaea.h5 -p 10 -o out/ -a 0.2
```

## Evaluation signals

- The compiled cami2_tool_benchmark_accuracy.csv reproduces the reported ranking: kMetaShot attains top sensitivity and precision at species and genus levels (e.g. GI-Illumina species sensitivity median 69.75, IQR 3.86)
- GTDBtk and CAMITAX rows show no strain-level classification, while kMetaShot provides strain-level results
- All five metrics (sensitivity, precision, FPR, BA, F1) are populated for every tool and rank level (strain, species, genus) across GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio
- Aggregation covers all samples: 10 per GI/Air condition and 20 PA-PacBio samples, reported as medians with IQRs
- The kMetaShot run consistently applied ass2ref = 0.2 across all CAMI II datasets before metrics were computed

## Limitations

- GTDBtk outputs require GTDB-to-NCBI taxonomy conversion using ncbi_vs_gtdb_r89 tables before comparison; omitting this biases cross-tool metrics
- GTDBtk and CAMITAX show no strain-level classification in this benchmark, so strain-rank comparisons are only meaningful for kMetaShot
- The kMetaShot HDF5 reference requires ~22 Gb of storage and at least 32 Gb RAM to load; Docker runs need --shm-size=22g
- Metrics depend on CAMI II gold-standard expected taxonomies (CE), so the workflow cannot be transferred to communities without such reference profiles

## Evidence

- [methods] CAMI II benchmark settings: "benchmarked against GTDBtk (v1.0.2) and CAMITAX (v0.7.0; Nextflow v21.04.0) [37, 39] by using in silico CAMI II multi-sample mock community datasets"
- [results] ass2ref threshold applied uniformly: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [results] reference validation against published result: "kMetaShot obtained the highest sensitivity at species and genus levels in GI-Illumina (median 69.75 IQR 3.86, median 98.33 IQR 4.989)"
- [results] FPR ranges to expect in the table: "we observed values ranging from 0.2% (strains and species) to 1.9% (genus)."
- [methods] GTDB-to-NCBI conversion prerequisite: "using the ncbi_vs_gtdb_r89 conversion tables (available at https://data.gtdb.ecogenomic.org/releases/release89/89.0/)"
- [readme] reference resource constraints: "kMetaShot requires a reference file available at these Zenodo links:"
