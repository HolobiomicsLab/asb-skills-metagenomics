---
name: hazard-ratio-interpretation-microbiome-survival
description: Use when a time-to-event (survival) outcome must be associated with taxon-level
  microbial abundances (e.g., phylum CLR values), optionally covariate-adjusted and
  subgroup-stratified, yielding per-taxon hazard ratios.
when_to_use_negative:
- The analysis goal is comparative survival between treatment groups without microbiome
  data — that is Module 1, not this skill.
- The goal is prediction modeling of survival from microbial taxa rather than association
  testing — that is Module 4.
- Input is raw sequencing data; this skill starts from a processed phyloseq/feature-table
  object produced by the MiSurv data processing module (QIIME 2-based OTU picking
  at 97% similarity against Greengenes, chimera removal, rarefaction).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: MiSurv
  role: Provides Module 3 association testing between microbial composition and survival
    responses with or without covariate adjustment(s), including CLR transformation
    and subgroup analysis options
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: Holds and processes the taxon abundance data (biom.Rdata) that feed the Cox
    models; also used for rarefaction via rarefy_even_depth
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:62cc2e838cb2d7e506ae09a383a7cb07aa5efa100c26f6f9f14fc288b45c111d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - phyloseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hazard-ratio-interpretation-microbiome-survival

## Summary

Fit and interpret taxon-level Cox proportional hazards models that link CLR-transformed microbial abundance to survival, extracting exp(coef) hazard ratios per taxon — the core association test of MiSurv's Module 3 for microbiome-survival studies.

## When to use

Use when a time-to-event (survival) outcome must be associated with taxon-level microbial abundances (e.g., phylum CLR values), optionally covariate-adjusted and subgroup-stratified, yielding per-taxon hazard ratios.

## When NOT to use

- The analysis goal is comparative survival between treatment groups without microbiome data — that is Module 1, not this skill.
- The goal is prediction modeling of survival from microbial taxa rather than association testing — that is Module 4.
- Input is raw sequencing data; this skill starts from a processed phyloseq/feature-table object produced by the MiSurv data processing module (QIIME 2-based OTU picking at 97% similarity against Greengenes, chimera removal, rarefaction).

## Inputs

- phyloseq object 'biom.Rdata' from the MiSurvGit Data folder
- four individual files: 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre'
- sample metadata with survival time, censored/event status, follow-up period, treatment group, and covariates (e.g., Sex)

## Outputs

- per-taxon hazard ratios (exp(coef)) from covariate-adjusted Cox models (e.g., hazard_ratios.csv)
- taxon-level association test results for significant taxa (e.g., phylum-level HRs: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796)

## How to apply

Load the processed phyloseq object 'biom.Rdata' (or the four files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurvGit Data folder into R (>4.2.0). Aggregate OTU counts to the target taxonomic rank (e.g., phylum), convert to relative abundance, and apply a centered log-ratio (CLR) transformation — one of the transformations exposed in MiSurv's Data Processing module. For each taxon, fit a Cox proportional hazards model on the CLR-transformed abundance using Module 3 (association testing between microbial composition and survival responses), adding covariates such as Sex and enabling the subgroup option when treatment-group-specific effects are expected (e.g., control vs antibiotic). Extract exp(coef) as the hazard ratio per taxon and assemble the values into a table (e.g., hazard_ratios.csv). HR > 1 indicates increasing abundance raises the hazard of the event (e.g., T1D onset), HR < 1 indicates a protective association.

## Related tools

- **MiSurv** (Provides Module 3 association testing between microbial composition and survival responses with or without covariate adjustment(s), including CLR transformation and subgroup analysis options) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (Holds and processes the taxon abundance data (biom.Rdata) that feed the Cox models; also used for rarefaction via rarefy_even_depth) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Reproduced HRs match the Fig. 6 reference values within tolerance: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796.
- Each fitted model is a Cox proportional hazards model with the stated covariate(s) (e.g., Sex) and, where specified, restricted to the control (non-antibiotic) subgroup.
- Output table (hazard_ratios.csv) contains exp(coef) values for exactly the taxa modeled, one HR per taxon-level model.
- Abundance vectors are CLR-transformed relative abundances aggregated at the phylum rank, not raw counts or proportions.
- Survival variables (time, censored/event status, follow-up period) match the sample metadata in sam.dat.txt after the MiSurv quality-control adjustment of the follow-up period.

## Limitations

- Microbiome effects on survival responses may differ between treatment groups, so whole-cohort models can mask subgroup-specific associations; use the subgroup analysis option when this is plausible.
- MiSurv assumes inputs are already processed data (phyloseq object or the four constituent files); raw reads must be processed first (QIIME 2 feature table at 97% similarity, Greengenes reference, ChimeraSlayer chimera removal, rarefaction).
- Follow-up period must be quality-controlled in the Data Processing module before association testing, or survival times may be misaligned with events.

## Evidence

- [readme] Module 3: Association testing between microbial composition and survival responses with or without covariate adjustment(s).: "Module 3 : Association testing between microbial composition and survival responses with or without covariate adjustment(s)."
- [discussion] Subgroup analysis exists because microbiome effects on survival can differ between treatment groups.: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] Data transformations available in the Data Processing module include CLR, supporting the abundance transformation step.: "data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)"
- [discussion] The final processed data used in the paper are stored as a phyloseq object named biom.Rdata.: [evidence span withheld — anchor status: not_found]
