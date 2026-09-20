---
name: genome-equivalent-abundance-normalization
description: Use when per-genome mapped read counts/sequence depths (e.g. CoverM genome-mode
  output against source-associated genome databases) must be converted into genome-equivalent
  cell fractions for metagenomic fecal source tracking or apportionment.
when_to_use_negative:
- You need true mass-balance apportionment of inputs — SourceApp portions estimate
  proportions of cells based on genome equivalents, not a mass balance
- Your input is already a normalized feature/abundance table rather than raw per-genome
  mapped read counts and sequence depths
- You need to infer contamination age or identify nonconservative (growth-capable)
  populations, which the framework cannot do
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: SourceApp
  role: Tool whose --use-geq mode reports metrics akin to prokaryotic cell fraction
    via genome-equivalent normalization; provides the fractioning output format
  repo: https://github.com/bglindner/SourceApp
- name: MicrobeCensus
  role: Estimates average genome size of each metagenome to compute total genome equivalents
    used as the normalization denominator
- name: CoverM
  role: Quantifies per-genome mapped read counts and sequence depths against source-associated
    genome databases, feeding the GEQ normalization
  repo: https://github.com/wwood/CoverM
- name: Python
  role: Implementation language of SourceApp; used to assemble per-source cell fractions
    into the Source/Fraction output table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:241cc2b0811e3964ecc1a0e8a52d20027dd5ed7b11008a9fc7d9dad5bf59e9b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - MicrobeCensus
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# genome-equivalent-abundance-normalization

## Summary

Normalizes mapped-read counts from metagenomes against source-associated genome databases into genome equivalents (GEQ), yielding per-source cell fractions comparable to prokaryotic cell proportions for fecal source attribution and apportionment.

## When to use

Use when per-genome mapped read counts/sequence depths (e.g. CoverM genome-mode output against source-associated genome databases) must be converted into genome-equivalent cell fractions for metagenomic fecal source tracking or apportionment.

## When NOT to use

- You need true mass-balance apportionment of inputs — SourceApp portions estimate proportions of cells based on genome equivalents, not a mass balance
- Your input is already a normalized feature/abundance table rather than raw per-genome mapped read counts and sequence depths
- You need to infer contamination age or identify nonconservative (growth-capable) populations, which the framework cannot do

## Inputs

- Per-genome mapped read counts and sequence depths from CoverM (v0.7.0) genome mode against a source-associated genome database
- Metagenomic shotgun FASTQ reads (Illumina NovaSeq 6000 2 x 150-bp) for MicrobeCensus average-genome-size estimation
- Source-associated genome database with source-category assignments and cross-reactivity (ANI >= 95%) flags

## Outputs

- Per-metagenome total genome equivalents (GEQ) and estimated average genome size
- Per-source cell fraction table (cell_fractions.csv with Source and Fraction columns), values in [0, 1]

## How to apply

Load the per-genome mapped read counts and sequence depths produced by CoverM (v0.7.0) against the source-associated genome database for each metagenome. Run MicrobeCensus (MicrobeCensus-SourceApp 1.1.2) on each metagenome to estimate average genome size and compute total genome equivalents (GEQ) of the sample. For each source category j, sum the sequence depth X of all genomes in that source (including cross-reactive genomes flagged at ANI >= 95%) and divide by GEQ, applying the cell fraction formula cellfraction_j = sum(X_i)/GEQ. Apply the 0.01% relative-abundance limit of detection before reporting fractions, mirroring SourceApp's default LOD. Assemble per-source fractions into a table with Source and Fraction columns, matching SourceApp's fractioning output. Conceptually, this approximates the proportions of cells (genome copies) per source, not a mass balance of input mass.

## Related tools

- **SourceApp** (Tool whose --use-geq mode reports metrics akin to prokaryotic cell fraction via genome-equivalent normalization; provides the fractioning output format) — https://github.com/bglindner/SourceApp
- **MicrobeCensus** (Estimates average genome size of each metagenome to compute total genome equivalents used as the normalization denominator)
- **CoverM** (Quantifies per-genome mapped read counts and sequence depths against source-associated genome databases, feeding the GEQ normalization) — https://github.com/wwood/CoverM
- **Python** (Implementation language of SourceApp; used to assemble per-source cell fractions into the Source/Fraction output table)

## Examples

```
python sourceapp.py --use-geq -1 mesocosm_R1.fastq.gz -2 mesocosm_R2.fastq.gz
```

## Evaluation signals

- cell_fractions.csv exists with Source and Fraction fields
- All fraction values fall within [0, 1]
- Each source represented in the database appears as a row in the output table
- Known spike-in sources in ground-truth mesocosms (Table S1) are detectable above the 0.01% limit of detection
- Fractions are interpreted as genome-copy-based cell proportions, consistent with cellfraction_j = sum(X_i)/GEQ

## Limitations

- SourceApp's reported portions are estimates of proportions of cells based on genome equivalents, not the results of a mass balance of mass inputs, and may vary from mass-based expectations
- Substantial residuals remained after linear correction of cell fraction estimates even with the best-performing parameters (r2 = 0.64)
- Cow sources were underestimated due to under-sampling of cow fecal microbiomes; prokaryotic content of cow feces at the genomic level is poorly understood
- Species-level marker genomes have largely unknown persistence in the environment; the tool cannot infer contamination age or detect growth-capable populations

## Evidence

- [results] portions estimate proportions of cells based on genome equivalents, not mass balance: "the portions reported by SourceApp do not represent the results of a mass balance but rather efforts to estimate the proportions of cells belonging to source categories based on genome equivalents"
- [readme] SourceApp --use-geq reports prokaryotic cell fraction: "SourceApp reports metrics akin to prokaryotic cell fraction (when `--use-geq` is called)"
- [intro] Framework rationale: read mapping normalized by meaningful abundance units: "read mapping of metagenomic data to source-associated genome databases and normalization by meaningful abundance units"
- [results] Default LOD for reported cell fractions: "a limit of detection (LOD) of 0.01% relative abundance"
