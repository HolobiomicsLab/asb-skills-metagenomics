---
name: mixed-effects-microbiome-statistics
description: Use when you have a feature-abundance matrix (e.g. fungal species from
  shotgun metagenomes) plus sample metadata and need significant taxon–metadata associations
  while controlling for repeated or nuisance covariates like age.
when_to_use_negative:
- Input is raw sequencing reads rather than a per-sample abundance/feature table —
  profile reads with FunOMIC first
- The question is co-occurrence/inter-kingdom correlation structure, which this paper
  addressed with correlation network analysis (p < 0.001, |r| > 0.05) instead of fixed-effect
  association testing
- Sample size or metadata design cannot support a multivariable mixed-effects model
  with age as a random effect
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_2269
tools:
- name: MaAsLin2
  role: Multivariable linear association testing with metadata fixed effects, age
    as random effect, and default FDR multiple-testing correction
- name: FunOMIC
  role: Produces the fungal taxonomic (FunOMIC-T) and functional (FunOMIC-P) abundance
    feature matrices that are the model inputs
  repo: https://github.com/ManichanhLab/FunOMIC
- name: R
  role: Environment (version 4.1.2) in which MaAsLin2 association analysis and result
    filtering are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:5cc4ea65f4af70b73b6d619601732c44c98f4e130d7256a633654ff7e855763d
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
  - MaAsLin2
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mixed-effects-microbiome-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run multivariable linear association testing (MaAsLin2 in R 4.1.2) between fungal taxonomic/functional abundance features (from FunOMIC-T/FunOMIC-P profiling) and clinical/demographic metadata, with age as a random effect and FDR < 0.05 significance filtering.

## When to use

Use when you have a feature-abundance matrix (e.g. fungal species from shotgun metagenomes) plus sample metadata and need significant taxon–metadata associations while controlling for repeated or nuisance covariates like age.

## When NOT to use

- Input is raw sequencing reads rather than a per-sample abundance/feature table — profile reads with FunOMIC first
- The question is co-occurrence/inter-kingdom correlation structure, which this paper addressed with correlation network analysis (p < 0.001, |r| > 0.05) instead of fixed-effect association testing
- Sample size or metadata design cannot support a multivariable mixed-effects model with age as a random effect

## Inputs

- Fungal taxonomic abundance table (species-level, from FunOMIC-T mapping of 2,679 human metagenome samples)
- Fungal functional abundance table (from FunOMIC-P mapping)
- Sample metadata table (Supplementary Table S2: continent, country, city, sample source, gender, age, extraction procedure, use of mechanical lysis, health status)

## Outputs

- Table of significant fungal species–metadata associations (FDR < 0.05) with coefficients, p-values, and q-values
- Ranked results file maaslin2_significant_associations.tsv

## How to apply

Load the FunOMIC-derived fungal taxonomic abundance table (FunOMIC-T mapping of the 2,679 metagenomic samples) and/or the FunOMIC-P functional abundance table as the feature matrix, joined with sample metadata (continent, country, city, sample source, gender, age, extraction procedure, use of mechanical lysis) from Supplementary Table S2. Run MaAsLin2 in R (version 4.1.2) with each metadata variable as a fixed effect and age as a random effect, keeping MaAsLin2's default multiple-testing correction (BH FDR). Filter results to FDR < 0.05 and rank species–metadata associations (geography, body site, disease cohorts such as CD/UC/T1D/T2D/ESRD, extraction method such as bead-beating) by coefficient and significance. The rationale is that geography and health status dominate mycobiome variation, and technical covariates like bead-beating can induce artefactual associations (e.g. Yarrowia lipolytica positively associated with bead-beating, likely due to its higher chitin fraction of 10.3–18.9% versus S. cerevisiae, C. albicans, and M. restricta).

## Related tools

- **MaAsLin2** (Multivariable linear association testing with metadata fixed effects, age as random effect, and default FDR multiple-testing correction)
- **FunOMIC** (Produces the fungal taxonomic (FunOMIC-T) and functional (FunOMIC-P) abundance feature matrices that are the model inputs) — https://github.com/ManichanhLab/FunOMIC
- **R** (Environment (version 4.1.2) in which MaAsLin2 association analysis and result filtering are executed)

## Examples

```
fit <- MaAsLin2::runMaaslin2(taxon_abundance, metadata, output='maaslin2_out', fixed_effects=c('country','health_status','bead_beating'), random_effects=c('age'))
```

## Evaluation signals

- All reported associations pass MaAsLin2's default multiple-testing correction with q-value (FDR) < 0.05
- Output table maaslin2_significant_associations.tsv contains per-association feature, metadata variable, coefficient, p-value, and q-value columns
- Age is specified as the random effect and each metadata variable (e.g. country, health status, bead-beating) as fixed effects in the model call
- Feature matrix dimensions match the 2,679 human metagenome samples profiled by FunOMIC and metadata rows join 1:1 on sample IDs
- Known positive controls reproduce: Yarrowia lipolytica significantly positively associated with bead-beating

## Limitations

- Bead-beating/extraction-method associations may reflect technical bias rather than biology (e.g. Yarrowia lipolytica's higher chitin cell-wall fraction of 10.3–18.9%)
- Fungal taxonomic coverage is limited by completeness of available fungal genomes — single-copy marker gene extraction yields lower taxonomy coverage than amplicon databases
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, potentially confounding the functional (FunOMIC-P) feature matrix

## Evidence
