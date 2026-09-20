---
name: metagenomic-metadata-regression-modeling
description: Use when you have a microbial feature abundance table (taxonomic or functional
  profiles) plus sample metadata (e.g. geography, health status, extraction method)
  and need significant feature–variable associations from multivariable linear models.
when_to_use_negative:
- Input is already a differential-abundance or association result table (no regression
  needed — proceed directly to filtering/ranking).
- Goal is ordination or between-sample community-level testing (e.g. adonis2 from
  the vegan R package on beta diversity) rather than per-taxon metadata regression.
- Feature matrix is amplicon-derived ITS data rather than shotgun metagenomic abundances
  produced by FunOMIC-T mapping.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: MaAsLin2
  role: Multivariable linear regression of fungal abundances against metadata fixed
    effects with age as random effect and default multiple-testing correction
- name: R
  role: Runtime environment (version 4.1.2) in which MaAsLin2 association analysis
    is executed and results are filtered at FDR < 0.05
- name: FunOMIC
  role: Produces the taxonomic (FunOMIC-T) and functional (FunOMIC-P) fungal abundance
    matrices that serve as the regression feature inputs
  repo: https://github.com/ManichanhLab/FunOMIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:bc0532429804ed20e0805f679ee6eb2a3fb89d278505baa44066e8e61bf1560b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metagenomic-metadata-regression-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the FunOMIC study's MaAsLin2 multivariable regression linking fungal taxonomic/functional abundances from 2,679 human metagenome samples to clinical and demographic metadata, filtering associations at FDR < 0.05.

## When to use

Use when you have a microbial feature abundance table (taxonomic or functional profiles) plus sample metadata (e.g. geography, health status, extraction method) and need significant feature–variable associations from multivariable linear models.

## When NOT to use

- Input is already a differential-abundance or association result table (no regression needed — proceed directly to filtering/ranking).
- Goal is ordination or between-sample community-level testing (e.g. adonis2 from the vegan R package on beta diversity) rather than per-taxon metadata regression.
- Feature matrix is amplicon-derived ITS data rather than shotgun metagenomic abundances produced by FunOMIC-T mapping.

## Inputs

- FunOMIC-T fungal taxonomic abundance table (species-level, from 2,679 human metagenome samples)
- FunOMIC-P fungal functional abundance table (mapped with Diamond blastx, e-value < 10e-10)
- Sample metadata table (Supplementary Table S2: continent, country, city, sample source, gender, age, extraction procedure, use of mechanical lysis)

## Outputs

- maaslin2_significant_associations.tsv (ranked table of fungal species–metadata associations with coefficient, p-value, and FDR-adjusted q-value at FDR < 0.05)
- Ranked list of significant associations by disease cohort (CD/UC/T1D/T2D/ESRD), geography, body site, and extraction method (bead-beating)

## How to apply

Load the FunOMIC-T fungal taxonomic abundance table (mapped from 2,679 metagenomic samples) and/or the FunOMIC-P functional abundance table as the feature matrices, joining them with sample metadata (continent, country, city, sample source, gender, age, extraction procedure, use of mechanical lysis) from Supplementary Table S2. Run MaAsLin2 in R (version 4.1.2), supplying each metadata variable as a fixed effect and age as the random effect, keeping MaAsLin2's default multiple-testing correction. Filter the MaAsLin2 output to significant associations at FDR (false discovery rate) < 0.05. Rank species–metadata associations (geography, body site, disease cohorts such as CD/UC/T1D/T2D/ESRD, extraction method such as bead-beating) by coefficient and significance, and write them to maaslin2_significant_associations.tsv. Note that the underlying abundances come from reads kept only if >80% of their length aligned with q-score over 30, which shapes the feature matrix quality.

## Related tools

- **MaAsLin2** (Multivariable linear regression of fungal abundances against metadata fixed effects with age as random effect and default multiple-testing correction)
- **R** (Runtime environment (version 4.1.2) in which MaAsLin2 association analysis is executed and results are filtered at FDR < 0.05)
- **FunOMIC** (Produces the taxonomic (FunOMIC-T) and functional (FunOMIC-P) fungal abundance matrices that serve as the regression feature inputs) — https://github.com/ManichanhLab/FunOMIC

## Examples

```
Rscript -e "MaAsLin2::MaAsLin2(input_data=abund, input_metadata=meta, output='maaslin2_out', fixed_effects=c('health_status','country','body_site','extraction'), random_effects='age', max_significance=0.05)"
```

## Evaluation signals

- All rows in maaslin2_significant_associations.tsv have FDR-adjusted q-values < 0.05 (MaAsLin2 default BH correction).
- Regression model includes age as the random effect and the specified metadata variables as fixed effects; residuals/coefficients are finite and non-degenerate.
- A known replicate result is recovered: Yarrowia lipolytica positively associated with bead-beating extraction, attributed to its 10.3–18.9% cell-wall chitin fraction.
- Significant associations span the expected metadata categories (geography, body sites, disease cohorts such as CD/UC/T1D/T2D/ESRD, extraction method).
- Row counts of the feature table match the 2,679 metagenomic sample set before regression; samples join 1:1 with Supplementary Table S2 metadata.

## Limitations

- Extraction and quality of FunOMIC single-copy marker genes depend on the completeness of available fungal genomes, which may result in lower fungal taxonomy coverage compared with amplicon databases — affecting which taxa can show associations.
- High inter-kingdom conservation of protein-coding genes means bacterial contamination of functional profiles was not totally preventable, even with strict Diamond thresholds (coverage >95%, identity >99%, e-value < 10e-10), which can confound functional metadata associations.
- Association power depends on the metadata design (age as random effect); cohorts without balanced metadata covariates may yield spurious fixed-effect estimates.

## Evidence
