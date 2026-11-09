---
name: fungal-abundance-association-analysis
description: Use when you have a fungal taxonomic or functional abundance feature
  table (e.g., FunOMIC-T/FunOMIC-P profiles) joined to sample metadata and need to
  find which species associate with variables like country, body site, or disease
  cohort.
when_to_use_negative:
- Input is raw sequencing reads rather than a processed abundance feature table —
  run the FunOMIC pipeline (bowtie2 mapping to FunOMIC-T, Diamond blastx to FunOMIC-P)
  first
- Question is inter-kingdom correlation structure rather than metadata association
  — use the correlation network approach (p < 0.001, |correlation| > 0.05) instead
  of MaAsLin2
- Only a single metadata variable without covariates or repeated/age structure — a
  simple univariate test may suffice; MaAsLin2's multivariable model with age random
  effect is designed for multiple covariates
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: MaAsLin2
  role: Multivariable linear association testing with fixed metadata effects and age
    as random effect, default multiple-testing correction
- name: FunOMIC
  role: Source of the fungal taxonomic (FunOMIC-T) and functional (FunOMIC-P) abundance
    feature tables analyzed
  repo: https://github.com/ManichanhLab/FunOMIC
- name: R
  role: Execution environment for the MaAsLin2 association analysis (version 4.1.2)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:fb019ee54748e1bb67ff31b86e64440c08e3aae16deec6d068286b9f6b35ce07
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# fungal-abundance-association-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Multivariable association testing between FunOMIC-derived fungal taxonomic/functional abundance tables and clinical-demographic metadata using MaAsLin2 in R, identifying species–metadata links (geography, body site, disease, extraction method) significant at FDR < 0.05.

## When to use

Use when you have a fungal taxonomic or functional abundance feature table (e.g., FunOMIC-T/FunOMIC-P profiles) joined to sample metadata and need to find which species associate with variables like country, body site, or disease cohort.

## When NOT to use

- Input is raw sequencing reads rather than a processed abundance feature table — run the FunOMIC pipeline (bowtie2 mapping to FunOMIC-T, Diamond blastx to FunOMIC-P) first
- Question is inter-kingdom correlation structure rather than metadata association — use the correlation network approach (p < 0.001, |correlation| > 0.05) instead of MaAsLin2
- Only a single metadata variable without covariates or repeated/age structure — a simple univariate test may suffice; MaAsLin2's multivariable model with age random effect is designed for multiple covariates

## Inputs

- FunOMIC-T fungal taxonomic abundance table from 2,679 human metagenomic samples
- FunOMIC-P fungal functional abundance table (3.4 million non-redundant fungal proteins)
- Sample metadata table (Supplementary Table S2: continent, country, city, sample source, gender, age, extraction procedure, mechanical lysis usage)

## Outputs

- maaslin2_significant_associations.tsv — ranked significant fungal species–metadata associations at FDR < 0.05
- Full MaAsLin2 result table with coefficients, p-values, and FDR-corrected q-values per feature–metadata pair

## How to apply

Load the FunOMIC taxonomic abundance table (FunOMIC-T mapping of the 2,679 metagenomic samples) and/or the FunOMIC-P functional abundance table as feature matrices, and join them with sample metadata (continent, country, city, sample source, gender, age, extraction procedure, use of mechanical lysis) from Supplementary Table S2. Run MaAsLin2 in R (version 4.1.2) with each metadata variable as a fixed effect and age as a random effect, keeping MaAsLin2's default multiple-testing correction. Filter results to significant associations at FDR < 0.05, then rank species–metadata associations (geography, body site, CD/UC/T1D/T2D/ESRD cohorts, bead-beating vs. other extraction) by coefficient and significance. Write the ranked table to maaslin2_significant_associations.tsv. Rationale: FunOMIC profiling of >2,600 samples already revealed species associated with geography, body sites, and diseases; MaAsLin2's multivariable linear models with random effects separate metadata effects from age structure. Notable outcome to expect: Yarrowia lipolytica positively associated with bead-beating, attributed to its higher cell-wall chitin fraction (10.3–18.9%) versus S. cerevisiae, C. albicans, and M. restricta.

## Related tools

- **MaAsLin2** (Multivariable linear association testing with fixed metadata effects and age as random effect, default multiple-testing correction)
- **FunOMIC** (Source of the fungal taxonomic (FunOMIC-T) and functional (FunOMIC-P) abundance feature tables analyzed) — https://github.com/ManichanhLab/FunOMIC
- **R** (Execution environment for the MaAsLin2 association analysis (version 4.1.2))

## Examples

```
fit_data <- MaAsLin2(data=fungal_abundance, metadata=sample_metadata, fixed_effects=c("country","body_site","health_status","bead_beating"), random_effects="age", output="maaslin2_out")
```

## Evaluation signals

- Significance threshold enforced: only associations with FDR (MaAsLin2 default correction) < 0.05 retained in maaslin2_significant_associations.tsv
- Expected replicable finding: Yarrowia lipolytica positively associated with bead-beating (mechanical lysis) among the significant associations
- Geography, body site, and disease cohort (CD/UC/T1D/T2D/ESRD) associations should dominate the top-ranked terms, consistent with the paper's reported signatures
- Sample coverage check: feature tables derive from the full 2,679-sample set with metadata joined from Supplementary Table S2; row counts should reconcile with non-missing metadata
- Random-effect structure verified: age entered as random effect, all other metadata variables as fixed effects in the MaAsLin2 call

## Limitations

- FunOMIC's marker-gene extraction depends on completeness of available fungal genomes, giving lower taxonomy coverage than amplicon databases
- High inter-kingdom conservation of protein-coding genes means bacterial contamination of functional profiles was not totally preventable despite strict mapping thresholds (coverage >95%, identity >99%, e-value < 10e-10)
- No bacterial read filtration step was included before functional annotation in this version; only proposed for a future update

## Evidence
