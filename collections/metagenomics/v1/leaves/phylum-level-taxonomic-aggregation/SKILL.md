---
name: phylum-level-taxonomic-aggregation
description: Use when a phyloseq object or OTU table with taxonomic annotations needs
  to be collapsed to phylum-level CLR-transformed abundances for survival-association
  modeling or treatment-group comparison.
when_to_use_negative:
- Input is raw sequence reads or a non-rarefied feature table that has not passed
  the Data Processing module quality controls (kingdom, library size, mean proportion,
  taxonomic name filters)
- The analysis requires OTU/genus-level resolution or compositional methods that assume
  absolute counts rather than CLR-transformed phylum summaries
- The study design involves continuous or binary (non-survival) responses, for which
  MiCloud rather than MiSurv is the intended platform
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MiSurv
  role: Web/R Shiny platform whose Data Processing module performs CLR transformation
    and whose Module 3 consumes CLR-transformed phylum abundances for survival association
    testing
  repo: https://github.com/wg99526/MiSurvGit
- name: phyloseq
  role: R package used to load 'biom.Rdata', aggregate OTUs to the phylum rank (taxonomic
    glom), and compute relative abundances
  repo: https://github.com/wg99526/MiSurvGit
- name: QIIME2
  role: Upstream pipeline that produced the feature table, taxonomic annotations,
    and phylogenetic tree that feed the aggregation step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:b6204e916035615861973723db47ee71bb96748aab1beb8f249f1f1ac68c31a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiSurv
  - QIIME2
  - phyloseq
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# phylum-level-taxonomic-aggregation

## Summary

Aggregates OTU/feature counts to the phylum rank, converts them to relative abundance, and applies a centered log-ratio (CLR) transformation so phylum-level abundances can be entered into downstream models (e.g., Cox proportional hazards via MiSurv Module 3) on the T1D mouse microbiome data.

## When to use

Use when a phyloseq object or OTU table with taxonomic annotations needs to be collapsed to phylum-level CLR-transformed abundances for survival-association modeling or treatment-group comparison.

## When NOT to use

- Input is raw sequence reads or a non-rarefied feature table that has not passed the Data Processing module quality controls (kingdom, library size, mean proportion, taxonomic name filters)
- The analysis requires OTU/genus-level resolution or compositional methods that assume absolute counts rather than CLR-transformed phylum summaries
- The study design involves continuous or binary (non-survival) responses, for which MiCloud rather than MiSurv is the intended platform

## Inputs

- phyloseq object 'biom.Rdata' from the MiSurvGit Data folder
- Four-file alternative: 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre'
- Sample metadata with treatment and Sex variables
- Taxonomic annotation table (RDP classifier / Greengenes-based) supporting phylum-rank assignment

## Outputs

- Phylum-level relative abundance table
- CLR-transformed phylum abundance matrix (one column per phylum, one row per sample)
- Downstream table such as hazard_ratios.csv (HR = exp(coef)) per phylum

## How to apply

Load the processed phyloseq object 'biom.Rdata' (or the four individual files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', 'tree.tre') from the MiSurvGit Data folder into R (>4.2.0). Use phyloseq's taxonomic-glom functionality to merge OTU counts at the phylum rank, then convert counts to relative abundance (proportions per sample). Apply a centered log-ratio (CLR) transformation to each phylum abundance vector — the MiSurv Data Processing module offers CLR among its data transformations alongside proportion, rarefaction, and arcsine square root. The resulting CLR phylum values serve as covariates in MiSurv Module 3 association testing (Cox proportional hazards with covariates such as Sex) or as features in Module 4 prediction modeling. Correctness is judged by whether downstream models (e.g., phylum-level HRs for Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796 in the control subgroup) reproduce the Fig. 6 reference values.

## Related tools

- **MiSurv** (Web/R Shiny platform whose Data Processing module performs CLR transformation and whose Module 3 consumes CLR-transformed phylum abundances for survival association testing) — https://github.com/wg99526/MiSurvGit
- **phyloseq** (R package used to load 'biom.Rdata', aggregate OTUs to the phylum rank (taxonomic glom), and compute relative abundances) — https://github.com/wg99526/MiSurvGit
- **QIIME2** (Upstream pipeline that produced the feature table, taxonomic annotations, and phylogenetic tree that feed the aggregation step)

## Examples

```
library(phyloseq); ps <- readRDS('biom.Rdata'); ps.phylum <- transform_sample_counts(tax_glom(ps, taxrank='Phylum'), function(x) x/sum(x)); ps.clr <- microbiome::transform(ps.phylum, 'clr')
```

## Evaluation signals

- Reproduced phylum-level Cox-model HRs match Fig. 6 reference values within tolerance: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796 in the control (non-antibiotic) subgroup
- CLR-transformed phylum values are centered so that each sample's phylum CLR vector has zero mean (compositional constraint honored)
- Phylum-level table row/column counts match the number of samples in the treatment-subset metadata and the number of distinct phyla in tax.tab.txt
- Resulting hazard_ratios.csv contains exactly one exp(coef) row per modeled phylum, with Sex included as a covariate in every model
- Aggregation reproduces from both 'biom.Rdata' and the four individual files (otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre)

## Limitations

- Rarefaction (rarefy_even_depth in phyloseq) and quality-control thresholds applied upstream affect the aggregated abundances; results are not robust to skipping the Data Processing module's QC (kingdom, library size, mean proportion, taxonomic name)
- CLR transformation cannot be applied to samples containing zeros without an additional zero-handling step not specified in the article
- No software version numbers are given for R packages used (e.g., phyloseq, survival), so exact reproducibility of the aggregation may depend on package versions
- The microbiome effect on survival can differ between treatment groups, so phylum-level associations should be assessed with the subgroup analysis option rather than pooled across groups

## Evidence

- [readme] data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root): "data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)"
- [readme] stored as phyloseq format biom.Rdata plus four individual files: "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] recent version of R (>4.2.0) is needed: "For local implementation using this GitHub repository, recent version of **R (>4.2.0)** is needed."
- [intro] Module 3 association testing description: "Module 3: Association testing between microbial composition and survival responses."
- [discussion] subgroup analysis rationale: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [discussion] upstream feature table construction with QIIME 2.0: "We used QIIME 2.0 [16] to obtain the feature table, taxonomic annotations, and phylogenetic tree"
