---
name: interkingdom-correlation-analysis
description: Use when paired fungal and bacterial taxonomic abundance tables plus
  sample metadata are available and the goal is to count or rank significant inter-kingdom
  correlations within defined healthy cohorts (e.g., Danish vs Spanish gut).
when_to_use_negative:
- Input is still raw sequencing reads rather than per-sample fungal/bacterial abundance
  profiles — profile with FunOMIC (bowtie2 to FunOMIC-T) and MetaPhlAn first
- The analysis goal is differential abundance or single-taxon association testing
  rather than compositional co-occurrence network structure between kingdoms
- Samples cannot be cleanly subset into defined cohorts (e.g., healthy Danish and
  Spanish gut) via the metadata fields, since SparCC is run separately per cohort
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SparCC
  role: Python script run per cohort to estimate inter-kingdom correlations between
    fungal and bacterial taxonomic and functional profilings
- name: FunOMIC
  role: Produces the fungal taxonomic profiles that serve as one side of the correlation
    analysis via mapping reads to the FunOMIC-T database
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Maps metagenome reads to the FunOMIC-T marker-gene database to generate the
    fungal abundance table used in the correlations
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: MetaPhlAn
  role: Generates the bacterial taxonomic profiles (v3.0.9) correlated against the
    fungal profiles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:b0b5209146469173b1c27289f9f4e05efa15cdbffe4d44a430b4fa7b4d567813
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# interkingdom-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates SparCC correlation networks between fungal taxonomic/functional profiles (from FunOMIC-T mapping of human metagenomes) and MetaPhlAn v3.0.9 bacterial profiles, retaining significant inter-kingdom links (p < 0.001, |rho| > 0.05) per cohort to quantify fungus–bacterium co-occurrence structure.

## When to use

Use when paired fungal and bacterial taxonomic abundance tables plus sample metadata are available and the goal is to count or rank significant inter-kingdom correlations within defined healthy cohorts (e.g., Danish vs Spanish gut).

## When NOT to use

- Input is still raw sequencing reads rather than per-sample fungal/bacterial abundance profiles — profile with FunOMIC (bowtie2 to FunOMIC-T) and MetaPhlAn first
- The analysis goal is differential abundance or single-taxon association testing rather than compositional co-occurrence network structure between kingdoms
- Samples cannot be cleanly subset into defined cohorts (e.g., healthy Danish and Spanish gut) via the metadata fields, since SparCC is run separately per cohort

## Inputs

- Fungal taxonomic abundance table from FunOMIC-T mapping of the 2,679 human metagenomes (bowtie2)
- Bacterial taxonomic abundance table from MetaPhlAn v3.0.9
- Sample metadata table with continent, country, sample source, and disease-status fields
- SparCC Python script for compositional correlation estimation

## Outputs

- Per-cohort edge-list table (fungus, bacterium, rho, p-value, cohort) of significant inter-kingdom correlations
- Per-cohort summary block with counts of significant correlations (4,184 Danish; 3,471 Spanish) and strongest functional correlation coefficients (rho = 0.06 and rho = -0.13)
- Correlation network visualization restricted to p < 0.001 and |rho| > 0.05 edges

## How to apply

First subset the fungal profile table (built by mapping metagenome reads to the FunOMIC-T database with bowtie2, keeping reads with >80% aligned length and q-score >30) and the MetaPhlAn v3.0.9 bacterial table to the cohorts of interest using metadata fields (continent, country, sample source, disease status). Run the SparCC Python script separately per cohort to estimate inter-kingdom correlations between fungal and bacterial taxonomic (and functional) profilings, since SparCC is designed for compositional abundance data. Filter the correlation pairs to significant links using the paper's visualization criteria: only correlations with p-values < 0.001 and an absolute correlation coefficient over 0.05 are represented. Assemble a per-cohort edge-list (fungus, bacterium, rho, p-value, cohort) plus a summary of significant-link counts and the strongest functional correlation coefficients (expected rho = 0.06 and rho = -0.13). Finally verify computed counts against the reported 4,184 (Danish) and 3,471 (Spanish) significant inter-kingdom correlations.

## Related tools

- **SparCC** (Python script run per cohort to estimate inter-kingdom correlations between fungal and bacterial taxonomic and functional profilings)
- **FunOMIC** (Produces the fungal taxonomic profiles that serve as one side of the correlation analysis via mapping reads to the FunOMIC-T database) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Maps metagenome reads to the FunOMIC-T marker-gene database to generate the fungal abundance table used in the correlations) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **MetaPhlAn** (Generates the bacterial taxonomic profiles (v3.0.9) correlated against the fungal profiles)

## Evaluation signals

- Significant (p < 0.001, |rho| > 0.05) inter-kingdom edge counts reproduce the reported values: 4,184 in one healthy cohort and 3,471 in the other (Danish vs Spanish)
- Strongest functional correlation coefficients match the expected rho = 0.06 and rho = -0.13
- Negative correlations between Candida albicans and short-chain-fatty-acid-producing bacteria appear among the retained edges
- Every retained edge passes both filter criteria (p < 0.001 and |rho| > 0.05); edge-list schema contains fungus, bacterium, rho, p-value, cohort columns
- Separate SparCC runs are confirmed for each cohort (no cross-cohort pooling before correlation)

## Limitations

- The paper reports both 'significant (p < 0.05)' correlations in the finding text and the stricter p < 0.001 visualization filter — the retained-edge count depends on which cutoff is applied
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, even with strict mapping thresholds, which can affect fungal functional profiles used in correlations
- No bacterial read filtration step was included before functional annotation in this version of FunOMIC, only proposed for a future update

## Evidence

- [methods] Fungal profiles generated by mapping reads to FunOMIC-T with bowtie2: [evidence span withheld — anchor status: not_found]
