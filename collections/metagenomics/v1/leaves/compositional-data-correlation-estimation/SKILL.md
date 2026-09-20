---
name: compositional-data-correlation-estimation
description: Use when you have two compositional taxonomic abundance tables (e.g.
  fungal profiles from FunOMIC-T and bacterial profiles from MetaPhlAn) over matched
  samples and need statistically filtered cross-kingdom correlation edges.
when_to_use_negative:
- Input is raw sequencing reads rather than taxonomic profile tables — profile first
  with FunOMIC (bowtie2 vs FunOMIC-T) and MetaPhlAn before correlating
- The analysis targets within-kingdom (fungus–fungus or bacterium–bacterium) correlations
  rather than the inter-kingdom fungus–bacterium networks this skill builds
- Samples include diseased or mixed-disease cohorts — the reported 4,184 and 3,471
  edge counts are specific to the healthy Danish and healthy Spanish gut subsets;
  do not expect these counts outside them
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SparCC
  role: Python script that estimates inter-kingdom correlations between fungal and
    bacterial taxonomic/functional profiles per cohort
- name: FunOMIC
  role: Pipeline producing the fungal taxonomic profiles (via FunOMIC-T) and functional
    profiles (via FunOMIC-P) that are the correlation inputs
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Maps next-generation sequencing reads to the FunOMIC-T fungal marker-gene
    database to generate the fungal abundance table
- name: MetaPhlAn
  role: Generates the bacterial taxonomic profiles (v3.0.9) used as the bacterial
    side of the inter-kingdom correlations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:af066763b722fb60d5689ff9eb3b5e7f730c048817fceb46901a4a306faab4fa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - FunOMIC
  - MetaPhlAn
  - SparCC
  - bowtie2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# compositional-data-correlation-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate inter-kingdom (fungal–bacterial) correlations from compositional taxonomic profiles using SparCC, as applied in FunOMIC to healthy Danish and Spanish gut cohorts to build correlation networks of mycobiome–bacteriome interactions.

## When to use

Use when you have two compositional taxonomic abundance tables (e.g. fungal profiles from FunOMIC-T and bacterial profiles from MetaPhlAn) over matched samples and need statistically filtered cross-kingdom correlation edges.

## When NOT to use

- Input is raw sequencing reads rather than taxonomic profile tables — profile first with FunOMIC (bowtie2 vs FunOMIC-T) and MetaPhlAn before correlating
- The analysis targets within-kingdom (fungus–fungus or bacterium–bacterium) correlations rather than the inter-kingdom fungus–bacterium networks this skill builds
- Samples include diseased or mixed-disease cohorts — the reported 4,184 and 3,471 edge counts are specific to the healthy Danish and healthy Spanish gut subsets; do not expect these counts outside them

## Inputs

- Fungal taxonomic abundance table obtained by mapping 2,679 human metagenomes to the FunOMIC-T database (1.69 million fungal marker genes) with bowtie2
- Bacterial taxonomic abundance table from MetaPhlAn v3.0.9 profiling
- Fungal functional profiles mapped to FunOMIC-P with Diamond blastx (optional, for functional correlations)
- Sample metadata with fields: continent, country, sample source, disease status

## Outputs

- Per-cohort edge-list table of significant inter-kingdom correlations (fungus, bacterium, rho, p-value, cohort)
- Summary counts of significant correlations per cohort (e.g. 4,184 Danish, 3,471 Spanish)
- Strongest functional correlation coefficients (e.g. rho = 0.06 and rho = -0.13)
- Filtered correlation network for visualization (p < 0.001, |rho| > 0.05)

## How to apply

First map the metagenomic reads to the FunOMIC-T database with bowtie2 to obtain fungal taxonomic profiles, and to a bacterial database (MetaPhlAn v3.0.9) for bacterial profiles. Subset both tables to the cohorts of interest using metadata fields (continent, country, sample source, disease status) — in the paper, the healthy Danish and healthy Spanish gut cohorts from the 2,679 human metagenomes. Run the SparCC Python script separately per cohort to estimate inter-kingdom correlations between the compositional fungal and bacterial (and functional) profiles, since SparCC handles the compositional nature of relative abundance data. Filter the resulting correlation pairs to significant links using the paper's network criteria: p-values < 0.001 and absolute correlation coefficient over 0.05. Assemble a per-cohort edge list (fungus, bacterium, rho, p-value, cohort) and check the significant-edge counts against the reported values (4,184 for the Danish healthy cohort and 3,471 for the Spanish cohort), verifying that Candida albicans negative correlations with short-chain-fatty-acid-producing bacteria are retained.

## Related tools

- **SparCC** (Python script that estimates inter-kingdom correlations between fungal and bacterial taxonomic/functional profiles per cohort)
- **FunOMIC** (Pipeline producing the fungal taxonomic profiles (via FunOMIC-T) and functional profiles (via FunOMIC-P) that are the correlation inputs) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Maps next-generation sequencing reads to the FunOMIC-T fungal marker-gene database to generate the fungal abundance table)
- **MetaPhlAn** (Generates the bacterial taxonomic profiles (v3.0.9) used as the bacterial side of the inter-kingdom correlations)

## Evaluation signals

- Significant-edge counts match the reported values: 4,184 for one healthy cohort and 3,471 for the other (Danish vs Spanish)
- Retained edges all satisfy the network filter: p < 0.001 and |correlation coefficient| > 0.05
- Strongest functional correlation coefficients fall at the expected values rho = 0.06 and rho = -0.13
- Candida albicans negative correlations with short-chain-fatty-acid-producing bacteria appear among the retained edges
- Edge lists are cohort-specific and only contain fungus–bacterium pairs after metadata-based subsetting to healthy samples

## Limitations

- FunOMIC's marker-gene extraction and quality rely on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomies compared with fungal amplicon databases, affecting which taxa enter the network
- Bacterial contamination of fungal functional profiles was not totally preventable, even after applying an exceedingly strict mapping threshold (read coverage >95%, identity >99%, e-value <10e-10) to the fungal functional annotation
- No bacterial read filtration step is included before functional annotation yet; the authors only proposed it for a future update of the tool

## Evidence
