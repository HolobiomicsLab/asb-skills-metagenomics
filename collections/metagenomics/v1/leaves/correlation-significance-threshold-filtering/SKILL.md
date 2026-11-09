---
name: correlation-significance-threshold-filtering
description: Use when you have a dense pairwise correlation matrix (e.g. SparCC output
  over fungal and bacterial taxa) and must retain only significant edges — here p
  < 0.001 and |rho| > 0.05 — to build a sparse inter-kingdom network.
when_to_use_negative:
- Input is already a filtered edge list or network rather than a dense correlation
  matrix
- The correlation matrix has no associated p-values, so the p < 0.001 criterion cannot
  be applied
- The analysis targets within-kingdom (fungus–fungus or bacterium–bacterium) correlations
  rather than inter-kingdom links
tools:
- name: SparCC
  role: Estimates the inter-kingdom correlation matrix between fungal and bacterial
    taxonomic (and functional) profiles per cohort before threshold filtering
- name: FunOMIC
  role: Produces the fungal taxonomic profiles (via FunOMIC-T mapping) that supply
    the fungal side of the correlation pairs
  repo: https://github.com/ManichanhLab/FunOMIC
- name: bowtie2
  role: Maps metagenomic reads to the FunOMIC-T fungal marker-gene database to build
    the fungal abundance profiles used in the correlations
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: MetaPhlAn
  role: Generates the bacterial taxonomic profiles (v3.0.9) that supply the bacterial
    side of the correlation pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:7adf92b8b33c2e46e8b914dfdf688c4fadbfcb8afbcea269c8780669876fd84c
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# correlation-significance-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters a matrix of pairwise co-abundance correlations (e.g. SparCC inter-kingdom fungus–bacterium correlations from FunOMIC taxonomic profiles) to statistically significant links using p-value and correlation-coefficient cutoffs, yielding an edge-list network per cohort.

## When to use

Use when you have a dense pairwise correlation matrix (e.g. SparCC output over fungal and bacterial taxa) and must retain only significant edges — here p < 0.001 and |rho| > 0.05 — to build a sparse inter-kingdom network.

## When NOT to use

- Input is already a filtered edge list or network rather than a dense correlation matrix
- The correlation matrix has no associated p-values, so the p < 0.001 criterion cannot be applied
- The analysis targets within-kingdom (fungus–fungus or bacterium–bacterium) correlations rather than inter-kingdom links

## Inputs

- SparCC correlation matrix (fungus x bacterium) with paired p-values, one per cohort
- Fungal taxonomic profile table from FunOMIC-T mapping via bowtie2
- Bacterial taxonomic profile table from MetaPhlAn v3.0.9
- Sample metadata (continent, country, sample source, disease status) for cohort subsetting

## Outputs

- Per-cohort edge-list table (fungus, bacterium, rho, p-value, cohort) of significant correlations
- Summary block with counts of significant inter-kingdom correlations (expected 4,184 Danish, 3,471 Spanish)
- Filtered correlation network for visualization

## How to apply

Run SparCC separately per cohort (e.g. healthy Danish and healthy Spanish gut cohorts subsetted from the 2,679 human metagenomes profiled against FunOMIC-T with bowtie2 and bacteria profiled with MetaPhlAn v3.0.9) to estimate fungus–bacterium correlations. Then filter each correlation pair to significant links using the paper's criteria: only correlations with p-values < 0.001 and an absolute correlation coefficient over 0.05 are retained. Assemble a per-cohort edge-list (fungus, bacterium, rho, p-value, cohort) plus a summary block reporting the significant-edge counts and the strongest functional correlation coefficients (expected rho = 0.06 and rho = -0.13). The rationale is that raw SparCC matrices are dense and noisy; the dual threshold yields interpretable networks in which biologically meaningful interactions, such as Candida albicans negative correlations with short-chain-fatty-acid-producing bacteria, are preserved.

## Related tools

- **SparCC** (Estimates the inter-kingdom correlation matrix between fungal and bacterial taxonomic (and functional) profiles per cohort before threshold filtering)
- **FunOMIC** (Produces the fungal taxonomic profiles (via FunOMIC-T mapping) that supply the fungal side of the correlation pairs) — https://github.com/ManichanhLab/FunOMIC
- **bowtie2** (Maps metagenomic reads to the FunOMIC-T fungal marker-gene database to build the fungal abundance profiles used in the correlations) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **MetaPhlAn** (Generates the bacterial taxonomic profiles (v3.0.9) that supply the bacterial side of the correlation pairs)

## Evaluation signals

- Number of retained significant edges matches the reported counts: 4,184 for one healthy cohort and 3,471 for the other
- Every retained edge satisfies p < 0.001 and |rho| > 0.05
- Strongest functional correlation coefficients match expected values (rho = 0.06 and rho = -0.13)
- Candida albicans negative correlations with short-chain-fatty-acid-producing bacteria appear among the retained edges
- Edge-list schema contains the required columns (fungus, bacterium, rho, p-value, cohort) and one file per cohort

## Limitations

- The p < 0.001 and |rho| > 0.05 criteria are visualization/reporting thresholds from this paper; other studies may require different stringency
- High inter-kingdom conservation of protein-coding genes means bacterial contamination in fungal functional annotation was not totally preventable, which can affect functional correlations
- Fungal taxonomic coverage is limited by completeness of available fungal genomes, so poorly covered taxa may be absent from the correlation input profiles

## Evidence
