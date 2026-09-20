---
name: fecal-source-cell-fraction-estimation
description: Use when metagenomic read-mapping counts against source-associated genome
  databases must be converted into cell-fraction-like abundance units (genome equivalents)
  for fecal source attribution/apportionment.
when_to_use_negative:
- When the goal is a mass-balance of mass inputs — SourceApp portions estimate proportions
  of cells based on genome equivalents and do not represent a mass balance
- When reads have not been mapped to source-associated genome databases (e.g., input
  is already a taxonomic profile or 16S table) — GEQ normalization requires per-genome
  sequence depth
- When contamination sources lack representative genomes in the database, e.g., cow
  feces where false negatives across all samples reflect a poor understanding of cow
  fecal prokaryotic genomic content
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: MicrobeCensus
  role: Estimates average genome size of each metagenome from reads to compute total
    genome equivalents used to normalize sequence depths into cell fractions
- name: CoverM
  role: Maps metagenomic reads to source-associated genome databases (v0.7.0) and
    reports per-genome read counts and sequence depth used as the X_i terms in the
    cell fraction formula
  repo: https://github.com/wwood/CoverM
- name: SourceApp
  role: Integrates the GEQ normalization (--use-geq) and cell fraction formula into
    its fractioning output; provides the Source/Fraction table format and default
    database
  repo: https://github.com/bglindner/SourceApp
- name: Python
  role: Implementation language of SourceApp and the downstream normalization/aggregation
    script producing cell_fractions.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:3ab1b177920344ca477b4d809116759fc6559cabf3713ee8247b92b1ae26b610
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wwood/CoverM
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fecal-source-cell-fraction-estimation

## Summary

Normalizes per-genome mapped read counts (from CoverM) into genome-copy-based relative abundances by estimating average genome size and total genome equivalents (GEQ) with MicrobeCensus, yielding prokaryotic cell fractions per fecal source category as used by SourceApp.

## When to use

Use when metagenomic read-mapping counts against source-associated genome databases must be converted into cell-fraction-like abundance units (genome equivalents) for fecal source attribution/apportionment.

## When NOT to use

- When the goal is a mass-balance of mass inputs — SourceApp portions estimate proportions of cells based on genome equivalents and do not represent a mass balance
- When reads have not been mapped to source-associated genome databases (e.g., input is already a taxonomic profile or 16S table) — GEQ normalization requires per-genome sequence depth
- When contamination sources lack representative genomes in the database, e.g., cow feces where false negatives across all samples reflect a poor understanding of cow fecal prokaryotic genomic content

## Inputs

- Per-genome mapped read counts / sequence depth from CoverM genome mode against source-associated genome databases
- Shotgun metagenomic FASTQ reads (Illumina NovaSeq 6000 2 x 150-bp, gzipped) for GEQ estimation
- SourceApp source-associated genome database with per-genome source category assignments (including cross-reactive flags)

## Outputs

- Average genome size and total genome equivalents (GEQ) per metagenome
- Per-source cell fraction table (columns: Source, Fraction; cell_fractions.csv) in SourceApp fractioning output format

## How to apply

Load per-genome mapped read counts and sequence depths produced by CoverM (v0.7.0) mapping each mesocosm metagenome against the source-associated genome databases. Run MicrobeCensus (MicrobeCensus-SourceApp 1.1.2) on each metagenome to estimate average genome size and compute total genome equivalents (GEQ). For each source category j, sum the sequence depth X of all genomes assigned to that source — including both source-specific and cross-reactive genomes flagged at ANI ≥ 95% — and divide by GEQ following the cell fraction formula cellfraction_j = sum(X_i)/GEQ. Assemble per-source cell fractions into a table with Source and Fraction columns mirroring SourceApp's fractioning output, applying SourceApp parameters (93% read-to-genome identity, 70% alignment-to-read overlap, 0.01% LOD) when using the tool directly with --use-geq. The rationale is that genome equivalents approximate cell proportions, so reported portions estimate the proportion of cells belonging to each source rather than a mass balance of mass inputs.

## Related tools

- **MicrobeCensus** (Estimates average genome size of each metagenome from reads to compute total genome equivalents used to normalize sequence depths into cell fractions)
- **CoverM** (Maps metagenomic reads to source-associated genome databases (v0.7.0) and reports per-genome read counts and sequence depth used as the X_i terms in the cell fraction formula) — https://github.com/wwood/CoverM
- **SourceApp** (Integrates the GEQ normalization (--use-geq) and cell fraction formula into its fractioning output; provides the Source/Fraction table format and default database) — https://github.com/bglindner/SourceApp
- **Python** (Implementation language of SourceApp and the downstream normalization/aggregation script producing cell_fractions.csv)

## Examples

```
sourceapp.py --use-geq -i mesocosm_R1.fastq.gz -2 mesocosm_R2.fastq.gz -d sourceapp_db --percent-identity 0.93 --query-coverage 0.7 -o out/
```

## Evaluation signals

- cell_fractions.csv exists with the expected Source/Fraction fields
- All fraction values fall within [0, 1]
- Each source category represented in the genome database appears as a row in the output table
- Fractions for known spike-in sources in ground-truth mesocosms (Table S1) are detectable above the 0.01% limit of detection
- With optimal parameters (93% identity, 70% overlap, 0.01% LOD) attribution performance reaches ~0.898 specificity and 0.902 sensitivity against the spike-in mesocosm ground truth

## Limitations

- SourceApp portions do not represent a mass balance but estimate proportions of cells belonging to source categories based on genome equivalents, and may deviate from mass-based expectations
- Substantial residuals remain after linear correction of cell fraction estimates (r2 = 0.64) even with the best-performing parameters
- Performance varies substantially by source; cows were underestimated due to under-sampling of cow fecal microbiomes
- SourceApp cannot infer the age of contamination or identify nonconservative (growth-capable) populations; persistence of species-level marker genomes in the environment is largely unknown
- Estimated cell fraction must exceed the 0.01% limit of detection to be reported

## Evidence

- [intro] Normalization of mapped counts into genome equivalents: "we theorized and tested a new framework for metagenomic FST based on read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units i.e.,"
- [results] GEQ-based cell fraction semantics: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories based on genome equivalents"
- [results] Detection threshold and tuning parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [methods] CoverM as quantification step: "CoverM (v0.7.0; https://github.com/wwood/CoverM)"
- [readme] SourceApp reports cell-fraction-like metrics via --use-geq: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called) and has been benchmarked in laboratory trials"
- [results] Linear correction residuals: "substantial residuals remained after linear correction even with the best performing parameters (r 2 = 0.64"
