---
name: metagenomic-genome-size-estimation
description: Use when metagenomic read-mapping results (per-genome sequence depth
  X from CoverM against source-associated genome databases) must be normalized to
  genome equivalents to express source signal as approximate cell fractions.
when_to_use_negative:
- The goal is a mass balance of mass inputs — SourceApp portions are cell-fraction
  estimates based on genome equivalents, not mass-balance results
- Input is raw unassembled reads and no genome mapping has been done — normalization
  requires per-genome mapped sequence depths first
- Sources lack adequate genome representation in the database (e.g., cow fecal microbiomes
  were under-sampled, causing underestimation/false negatives)
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MicrobeCensus
  role: Estimates average genome size of each metagenome from reads to compute total
    genome equivalents (GEQ) used in the cell fraction formula
- name: CoverM
  role: Quantifies per-genome mapped read counts and sequence depth against the source-associated
    genome databases; those depths are the numerator of cellfraction_j = sum(X_i)/GEQ
  repo: https://github.com/wwood/CoverM
- name: SourceApp
  role: Consumes the normalized genome-equivalent-based fractions when --use-geq is
    called, producing source attribution and apportionment output in Source/Fraction
    format
  repo: https://github.com/bglindner/SourceApp
- name: Python
  role: Implementation language for SourceApp and the fraction-table assembly/validation
    scripts (Unix or Unix-like OS required)
  repo: https://github.com/bglindner/SourceApp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:b6f62de9975e712b5aa106a7116a656bf4272ef60eac2635f62fa641f5c21ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wwood/CoverM
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# metagenomic-genome-size-estimation

## Summary

Estimates average genome size and total genome equivalents (GEQ) for each metagenome with MicrobeCensus (MicrobeCensus-SourceApp 1.1.2), so that per-genome mapped read counts from CoverM can be converted into SourceApp's genomic-basis abundance unit approximating prokaryotic cell fraction for fecal source apportionment.

## When to use

Use when metagenomic read-mapping results (per-genome sequence depth X from CoverM against source-associated genome databases) must be normalized to genome equivalents to express source signal as approximate cell fractions.

## When NOT to use

- The goal is a mass balance of mass inputs — SourceApp portions are cell-fraction estimates based on genome equivalents, not mass-balance results
- Input is raw unassembled reads and no genome mapping has been done — normalization requires per-genome mapped sequence depths first
- Sources lack adequate genome representation in the database (e.g., cow fecal microbiomes were under-sampled, causing underestimation/false negatives)

## Inputs

- Per-genome mapped read counts / sequence depth (X) for each metagenome from CoverM genome mode against a source-associated genome database
- Metagenomic shotgun read sets (e.g., Illumina NovaSeq 6000 2 x 150-bp FASTQ) for which average genome size must be estimated
- Source-associated genome database with known source categories (e.g., SourceApp default database on Zenodo, 10.5281/zenodo.12795076)

## Outputs

- Estimated average genome size per metagenome from MicrobeCensus
- Total genome equivalents (GEQ) per sample
- cell_fractions.csv with Source and Fraction columns; each fraction within [0, 1]

## How to apply

Load the per-genome mapped read counts and sequence depths quantified by CoverM (v0.7.0) for each metagenome against the source-associated genome database. Run MicrobeCensus on each metagenome to estimate average genome size and compute the sample's total genome equivalents (GEQ). For each source category j, sum sequence depth X of all genomes belonging to that source (source-specific and cross-reactive) and divide by GEQ, i.e., cellfraction_j = sum(X_i)/GEQ. Assemble per-source fractions into a table with Source/Fraction columns, mirroring the SourceApp fractioning output. Note the rationale: the resulting portions estimate proportions of cells belonging to source categories based on genome equivalents, not a mass balance of mass inputs.

## Related tools

- **MicrobeCensus** (Estimates average genome size of each metagenome from reads to compute total genome equivalents (GEQ) used in the cell fraction formula)
- **CoverM** (Quantifies per-genome mapped read counts and sequence depth against the source-associated genome databases; those depths are the numerator of cellfraction_j = sum(X_i)/GEQ) — https://github.com/wwood/CoverM
- **SourceApp** (Consumes the normalized genome-equivalent-based fractions when --use-geq is called, producing source attribution and apportionment output in Source/Fraction format) — https://github.com/bglindner/SourceApp
- **Python** (Implementation language for SourceApp and the fraction-table assembly/validation scripts (Unix or Unix-like OS required)) — https://github.com/bglindner/SourceApp

## Examples

```
python sourceapp.py --use-geq
```

## Evaluation signals

- cell_fractions.csv exists with Source/Fraction fields and each source represented in the database appears as a row
- All fraction values fall within [0, 1]
- Fractions for known spike-in sources in ground-truth mesocosms (Table S1) are detectable above the 0.01% limit of detection
- GEQ estimates are finite and positive for every metagenome, and dividing summed CoverM sequence depths by GEQ reproduces the stored fractions
- Reported portions align with the genome-equivalent interpretation (approximately prokaryotic cell fraction), reproducible when SourceApp is run with --use-geq

## Limitations

- SourceApp portions do not represent a mass balance of mass inputs; they estimate proportions of cells based on genome equivalents and may vary from mass-based expectations
- Performance varies substantially by source — cows were underestimated due to under-sampling of cow fecal microbiomes, and false negatives for cows point to poor genomic understanding of cow feces
- Substantial residuals remained after linear correction of cell fraction estimates (r2 = 0.64), even with the best performing parameters
- Species-level marker genomes have largely unknown persistence in the environment; the tool cannot infer contamination age or distinguish growth-capable (nonconservative) populations

## Evidence

- [results] the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories based on genome equivalents: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories based on genome equivalents"
- [intro] we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units i.e.,: "we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units i.e.,"
- [readme] SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called): "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called)"
- [results] a limit of detection (LOD) of 0.01% relative abundance, which yielded 0.898: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
