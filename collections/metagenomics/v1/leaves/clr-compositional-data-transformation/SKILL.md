---
name: clr-compositional-data-transformation
description: Use when OTU/feature count tables (e.g., otu.tab.txt from the MiSurv
  Data folder) must be compositionally normalized before linear-regression differential
  abundance or Cox proportional hazards models that relate taxon abundance to survival
  outcomes.
when_to_use_negative:
- The analysis target is alpha/beta diversity or survival modeling that consumes diversity
  measures rather than taxon-level compositional abundances.
- The response is a continuous or binary variable instead of a survival (time-to-event)
  response — MiSurv is designed for survival responses, while MiCloud handles continuous/binary
  outcomes.
- The count table has already been transformed (e.g., proportions or arcsine square
  root) and another normalization is inappropriate for the planned regression.
tools:
- name: MiSurv
  role: Provides the data transformations step (centered log-ratio among rarefaction,
    proportion, and arcsine square root options) and the analytic modules (2 and 3)
    that consume CLR-transformed abundances
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to hold the phyloseq object (biom.Rdata), perform rarefaction
    via rarefy_even_depth, and aggregate counts to taxonomic ranks before CLR transformation
  repo: https://github.com/wg99526/MiSurvGit
- name: R
  role: Runtime environment (version >4.2.0) for local implementation of MiSurv and
    the CLR-based regression analyses
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:7289db3b31d2158f4427bbf39d3b19c1f55883b6b5af9285c70119ef0f4cc7bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - R
  - phyloseq
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clr-compositional-data-transformation

## Summary

Apply the centered log-ratio (CLR) transformation to microbiome taxonomic count data as implemented in MiSurv's data transformations step, producing compositional normalized taxon abundances that serve as the basis for covariate-adjusted differential abundance testing (phylum-to-genus) and Cox-model association testing of microbial composition with survival responses.

## When to use

Use when OTU/feature count tables (e.g., otu.tab.txt from the MiSurv Data folder) must be compositionally normalized before linear-regression differential abundance or Cox proportional hazards models that relate taxon abundance to survival outcomes.

## When NOT to use

- The analysis target is alpha/beta diversity or survival modeling that consumes diversity measures rather than taxon-level compositional abundances.
- The response is a continuous or binary variable instead of a survival (time-to-event) response — MiSurv is designed for survival responses, while MiCloud handles continuous/binary outcomes.
- The count table has already been transformed (e.g., proportions or arcsine square root) and another normalization is inappropriate for the planned regression.

## Inputs

- Processed OTU count table (otu.tab.txt) from the MiSurvGit Data folder
- Sample metadata (sam.dat.txt) including treatment group, Sex, and survival responses
- Taxonomy table (tax.tab.txt) and phylogenetic tree (tree.tre), or the combined phyloseq object biom.Rdata

## Outputs

- CLR-transformed taxon abundance matrix (phylum to genus rank) suitable for regression
- Table of significant taxa with BH-adjusted P-values (significant_taxa_fdr5.tsv; expected 66 rows at FDR 5%)
- Hazard-ratio table per taxon from CLR-based Cox models (hazard_ratios.csv)

## How to apply

Load the processed phyloseq object biom.Rdata (or its four files otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) from the MiSurvGit Data folder into R (>4.2.0), optionally after rarefaction with rarefy_even_depth in phyloseq. Select the centered log-ratio transformation among MiSurv's data transformation options (which include rarefaction, proportion, centered log-ratio, and arcsine square root), aggregating counts to the target taxonomic rank (phylum to genus). Use CLR-transformed abundances as the response in linear regression of each taxon on treatment group adjusted for Sex (Module 2), then apply Benjamini-Hochberg FDR at 5%; or feed CLR phylum abundances into Cox proportional hazards models with Sex as a covariate (Module 3), optionally within the control subgroup via the subgroup-analysis option. The CLR transform addresses the compositional nature of 16S rRNA-derived counts so that regression coefficients reflect shifts in relative abundance rather than sequencing-depth artifacts.

## Related tools

- **MiSurv** (Provides the data transformations step (centered log-ratio among rarefaction, proportion, and arcsine square root options) and the analytic modules (2 and 3) that consume CLR-transformed abundances) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to hold the phyloseq object (biom.Rdata), perform rarefaction via rarefy_even_depth, and aggregate counts to taxonomic ranks before CLR transformation) — https://github.com/wg99526/MiSurvGit
- **R** (Runtime environment (version >4.2.0) for local implementation of MiSurv and the CLR-based regression analyses) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- After CLR-based linear regression on group adjusted for Sex and BH FDR correction at 5%, the significant-taxon table contains exactly 66 rows, matching Figures 4 and 5.
- CLR-based Cox models in the control subgroup reproduce the reported hazard ratios: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796.
- Every output taxon retains its rank assignment (phylum through genus) from tax.tab.txt after transformation.
- CLR-transformed values are computed per sample across taxa, so per-sample vectors of transformed abundances are comparable and no raw counts remain in downstream model inputs.

## Limitations

- The MiSurv repository README provides no version numbers for the R packages used (e.g., phyloseq, survival, MiRKAT), so exact reproduction of the transformation code may vary with package versions.
- No software license is mentioned for MiSurv, and no changelog is available, limiting provenance checks on the transformation implementation.
- Hardware/server capacity of the MiSurv web instance may need improvement, implying possible limits when running the transformation pipeline at scale online.

## Evidence

- [readme] MiSurv README data transformations listing: "quality controls (kingdom, library size, mean proportion, taxonomic name), and (4) data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine"
- [intro] Module 3 definition: "Module 3: Association testing between microbial composition and survival responses."
- [discussion] Rarefaction tooling: [evidence span withheld — anchor status: not_found]
- [discussion] Subgroup rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [readme] Example data location: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
