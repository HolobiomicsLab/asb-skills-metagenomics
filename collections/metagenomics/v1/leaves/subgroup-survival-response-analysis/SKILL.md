---
name: subgroup-survival-response-analysis
description: Use when a survival (time-to-event) endpoint such as time-to-T1D-onset
  must be associated with microbiome features — CLR taxa via Cox models, Jaccard/unifrac
  beta-diversity via MiRKAT-S, or genus-level random survival forests — within one
  treatment subgroup of sample metadata.
when_to_use_negative:
- The goal is comparing survival curves or composition between treatment groups (MiSurv
  Modules 1–2), not within-group microbiome–survival association (Modules 3–4).
- The response is continuous or binary rather than survival (time-to-event) — the
  paper directs such data to MiCloud instead of MiSurv.
- 'Input is raw 16S rRNA sequence data: this skill starts from the already-processed
  phyloseq object produced via QIIME 2.0 feature-table/taxonomy/tree construction
  and rarefaction.'
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MiSurv
  role: Web cloud platform / local shiny app providing the subgroup-analysis option
    in Modules 3 (association testing) and 4 (prediction modeling) with survival responses
  repo: https://github.com/wg99526/MiSurvGit
- name: MiRKAT
  role: MiRKAT-S kernel-based association testing of beta-diversity (Jaccard dissimilarity)
    against survival responses within the control subgroup, adjusting for Sex
- name: phyloseq
  role: Loads the biom.Rdata phyloseq object, subsets to the control subgroup, and
    computes Jaccard distance via distance(method='jaccard')
- name: randomForestSRC
  role: rfsrc fits the 10,000-tree random survival forest on CLR-transformed genus
    taxa for subgroup prediction modeling and VIMP extraction
- name: shiny
  role: Launches the MiSurv app locally for interactive subgroup survival analysis
  repo: https://github.com/wg99526/MiSurvGit
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_004
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:336f07e883a4a84fdb7cdaf676f19e6b2bf35dfad71658d1cff69360ab054f4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiRKAT
  - MiSurv
  - phyloseq
  - randomForestSRC
  - shiny
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# subgroup-survival-response-analysis

## Summary

MiSurv subgroup analysis option for testing microbiome–survival relationships within a single treatment group (e.g., the non-antibiotic control arm of the Zhang et al. NOD-mouse T1D trial), rather than pooling across treatment groups, because microbiome effects on survival may differ between groups.

## When to use

Use when a survival (time-to-event) endpoint such as time-to-T1D-onset must be associated with microbiome features — CLR taxa via Cox models, Jaccard/unifrac beta-diversity via MiRKAT-S, or genus-level random survival forests — within one treatment subgroup of sample metadata.

## When NOT to use

- The goal is comparing survival curves or composition between treatment groups (MiSurv Modules 1–2), not within-group microbiome–survival association (Modules 3–4).
- The response is continuous or binary rather than survival (time-to-event) — the paper directs such data to MiCloud instead of MiSurv.
- Input is raw 16S rRNA sequence data: this skill starts from the already-processed phyloseq object produced via QIIME 2.0 feature-table/taxonomy/tree construction and rarefaction.

## Inputs

- biom.Rdata phyloseq object from the MiSurvGit Data folder (or otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre)
- Sample metadata (sam.dat.txt) with treatment/group, survival time, event/censored status, and Sex covariate columns
- OTU count table and taxonomy table (tax.tab.txt) for rank-level aggregation (phylum/genus)

## Outputs

- Per-taxon Cox proportional hazards table of CLR-abundance hazard ratios (hazard_ratios.csv) for the subgroup
- MiRKAT-S association P-value for the subgroup kernel test (e.g., mirkat_s_jaccard_control_pvalue.csv)
- Random survival forest output: top-10 genus VIMP importance table and OOB error-rate-by-trees curve (cf. Fig. 7A/7B)

## How to apply

Load the processed phyloseq object biom.Rdata (or the equivalent otu.tab.txt, sam.dat.txt, tax.tab.txt, tree.tre) from the MiSurvGit Data folder into R (>4.2.0), then subset samples to one treatment group using the group column of sam.dat.txt, keeping survival time, event/censoring status, and covariates such as Sex. For Module 3 association testing, aggregate OTUs to the target rank (phylum or genus), convert to relative abundance, apply a centered log-ratio (CLR) transformation, and fit per-taxon Cox proportional hazards models with Sex as covariate, extracting exp(coef) hazard ratios (e.g., control-subgroup phylum HRs: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796). For kernel-based beta-diversity testing, compute pairwise Jaccard dissimilarity with phyloseq::distance(method='jaccard') on the subgroup OTU table and run MiRKAT-S with Sex adjustment, reporting the P-value (published: 0.046 for control-group Jaccard, Fig. 3D). For Module 4 prediction, fit a random survival forest with randomForestSRC::rfsrc on CLR genus taxa against time-to-T1D-onset using 10,000 ensembled trees, confirm out-of-bag error converges by 10,000 trees, and rank genera by VIMP. The subgroup restriction is the key decision point: the paper's authors added this option explicitly because microbiome effects on survival responses are often assumed to differ between treatment groups.

## Related tools

- **MiSurv** (Web cloud platform / local shiny app providing the subgroup-analysis option in Modules 3 (association testing) and 4 (prediction modeling) with survival responses) — https://github.com/wg99526/MiSurvGit
- **MiRKAT** (MiRKAT-S kernel-based association testing of beta-diversity (Jaccard dissimilarity) against survival responses within the control subgroup, adjusting for Sex)
- **phyloseq** (Loads the biom.Rdata phyloseq object, subsets to the control subgroup, and computes Jaccard distance via distance(method='jaccard'))
- **randomForestSRC** (rfsrc fits the 10,000-tree random survival forest on CLR-transformed genus taxa for subgroup prediction modeling and VIMP extraction)
- **shiny** (Launches the MiSurv app locally for interactive subgroup survival analysis) — https://github.com/wg99526/MiSurvGit

## Examples

```
library(shiny); runGitHub("MiSurvGit", "wg99526", ref = "main")
```

## Evaluation signals

- Control-subgroup phylum CLR Cox models reproduce the Fig. 6 hazard ratios within tolerance: Verrucomicrobia 1.253, Proteobacteria 1.236, Firmicutes 1.262, Actinobacteria 0.759, Tenericutes 0.796.
- MiRKAT-S Jaccard test in the control subgroup (Sex-adjusted) yields P ≈ 0.046 (±0.01), matching Fig. 3D.
- Random survival forest OOB error-rate curve visually converges/stabilizes by 10,000 ensembled trees (cf. Fig. 7B).
- Ranked genus VIMP table reproduces the Fig. 7A top-10 influential genera.
- Subsetting check: the analysis sample contains only the target treatment group and retains survival time, event status, and Sex columns without dropped metadata.

## Limitations

- Subgroup restriction reduces the analysis to one treatment arm (e.g., control non-antibiotic mice), so power depends on within-group sample size; the option exists precisely because effects may differ between groups, so pooled results cannot substitute.
- No version numbers are documented for the R packages used (e.g., phyloseq, survival, MiRKAT), which may hinder exact reproduction.
- Results depend on upstream processing choices (rarefy_even_depth rarefaction, 97% Greengenes open-reference OTU picking, CLR transformation) and require R >4.2.0; no software license is mentioned for MiSurv.

## Evidence

- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
- [intro] Module 3: Association testing between microbial composition and survival responses.: "Module 3: Association testing between microbial composition and survival responses."
- [intro] MiSurv is designed to deal with survival responses while MiCloud (36) is for continuous or binary responses: "MiSurv is designed to deal with survival responses while MiCloud (36) is for continuous or binary responses"
- [readme] We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "We stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [readme] **Module 3** : Association testing between microbial composition and survival responses with or without covariate adjustment(s).: "**Module 3** : Association testing between microbial composition and survival responses with or without covariate adjustment(s)."
