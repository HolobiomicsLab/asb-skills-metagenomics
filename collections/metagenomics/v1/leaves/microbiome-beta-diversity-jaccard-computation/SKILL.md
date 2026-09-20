---
name: microbiome-beta-diversity-jaccard-computation
description: Use when a processed microbiome OTU table (e.g., otu.tab.txt or a phyloseq
  object such as biom.Rdata) must be converted into a Jaccard dissimilarity matrix
  for beta-diversity analysis or survival association testing in the MiSurv/MiRKAT
  workflow.
when_to_use_negative:
- When a phylogenetically informed beta-diversity metric (e.g., weighted/unweighted
  UniFrac) is required — Jaccard uses only presence/absence, not the 'tree.tre' phylogeny
  or abundances
- When the input is raw sequence data rather than a processed OTU/feature table —
  run the QIIME 2.0 processing (97% open-reference picking against Greengenes, chimera
  removal) first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: Computes the Jaccard dissimilarity matrix via its 'distance' function (method
    = 'jaccard') and supports rarefaction via 'rarefy_even_depth'
- name: MiSurv
  role: Web cloud / shiny platform whose Data Processing module performs beta-diversity
    calculation and whose Module 3 consumes the Jaccard kernel for survival association
    testing
  repo: https://github.com/wg99526/MiSurvGit
- name: MiRKAT
  role: Underlying kernel-based method used by MiSurv Module 3 (MiRKAT-S) to test
    association between the Jaccard distance kernel and survival responses
- name: shiny
  role: R framework used to launch MiSurv locally from the GitHub repository
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/spectrum.05059-22
    title: 'MiSurv: an Integrative Web Cloud Platform for User-Friendly Microbiome
      Data Analysis with Survival Responses'
  source_content_hash: sha256:d40a7e10c00f06d454992451959bafa33299acba096b3b6948890cd90064089a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wg99526/MiSurvGit
  tools:
  - MiRKAT
  - MiSurv
  - phyloseq
  - shiny
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# microbiome-beta-diversity-jaccard-computation

## Summary

Computes pairwise Jaccard dissimilarity (presence/absence-based beta-diversity) from a processed OTU/feature table using the phyloseq 'distance' function with method='jaccard', producing the distance matrix that MiSurv Module 3 (MiRKAT-S) consumes for microbiome–survival association testing.

## When to use

Use when a processed microbiome OTU table (e.g., otu.tab.txt or a phyloseq object such as biom.Rdata) must be converted into a Jaccard dissimilarity matrix for beta-diversity analysis or survival association testing in the MiSurv/MiRKAT workflow.

## When NOT to use

- When a phylogenetically informed beta-diversity metric (e.g., weighted/unweighted UniFrac) is required — Jaccard uses only presence/absence, not the 'tree.tre' phylogeny or abundances
- When the input is raw sequence data rather than a processed OTU/feature table — run the QIIME 2.0 processing (97% open-reference picking against Greengenes, chimera removal) first

## Inputs

- phyloseq object ('biom.Rdata') from the MiSurv Data folder
- Individual files: 'otu.tab.txt' (OTU count table), 'sam.dat.txt' (sample metadata with group, survival time, event, covariates), 'tax.tab.txt' (taxonomy), 'tree.tre' (phylogenetic tree)
- Subgroup definition (e.g., control vs treatment group from the sample metadata group column)

## Outputs

- Pairwise Jaccard dissimilarity matrix (phyloseq 'dist' object) over the selected samples
- Distance kernel ready for MiSurv Module 3 (MiRKAT-S) association testing

## How to apply

Load the processed study data from the MiSurv repository Data folder as the 'phyloseq' object 'biom.Rdata' or the equivalent files 'otu.tab.txt', 'sam.dat.txt', 'tax.tab.txt', and 'tree.tre'. If a subgroup analysis is required, subset the phyloseq object using the group column of 'sam.dat.txt' while retaining survival time, event status, and covariate columns. Compute the pairwise Jaccard dissimilarity with the phyloseq 'distance' function (method = 'jaccard'), which bases the metric on taxon presence/absence rather than abundance. Optionally precede this with quality-control transformations supported by MiSurv's Data Processing module (library-size filtering, rarefaction via 'rarefy_even_depth' in phyloseq, or proportion transformation) so the Jaccard matrix reflects consistently processed samples. The resulting distance matrix is then used as the kernel input to MiSurv Module 3 (MiRKAT-S) association testing with covariate adjustment.

## Related tools

- **phyloseq** (Computes the Jaccard dissimilarity matrix via its 'distance' function (method = 'jaccard') and supports rarefaction via 'rarefy_even_depth')
- **MiSurv** (Web cloud / shiny platform whose Data Processing module performs beta-diversity calculation and whose Module 3 consumes the Jaccard kernel for survival association testing) — https://github.com/wg99526/MiSurvGit
- **MiRKAT** (Underlying kernel-based method used by MiSurv Module 3 (MiRKAT-S) to test association between the Jaccard distance kernel and survival responses)
- **shiny** (R framework used to launch MiSurv locally from the GitHub repository)

## Examples

```
library(phyloseq); ps <- readRDS("biom.Rdata"); jac <- distance(ps, method = "jaccard")
```

## Evaluation signals

- The resulting distance matrix is symmetric with zero diagonal and bounded in [0, 1], with dimensions matching the number of retained samples
- In the Zhang et al. antibiotic–T1D control-subgroup benchmark, MiRKAT-S on the Jaccard kernel with Sex as covariate yields P = 0.046 (Fig. 3D), reproducible within ~±0.01 rounding tolerance
- The Jaccard matrix is reproducible from both the 'biom.Rdata' phyloseq object and the equivalent individual files 'otu.tab.txt'/'sam.dat.txt'/'tax.tab.txt'/'tree.tre'
- Sample names in the distance matrix match the survival metadata rows retained after subgroup subsetting, with no missing-sample mismatch

## Limitations

- Jaccard is a presence/absence metric, so abundance information in the OTU table is discarded; results may differ from abundance-based dissimilarities
- MiSurv's Data Processing quality controls (kingdom, library size, mean proportion, follow-up period adjustment) must be applied consistently, otherwise the distance matrix will not match the published analysis
- No version numbers are given for the R packages used (e.g., phyloseq, survival, MiRKAT), which may affect exact reproducibility

## Evidence

- [intro] Module 3: Association testing between microbial composition and survival responses.: "Module 3: Association testing between microbial composition and survival responses."
- [readme] data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root): "data transformations (alpha- and beta-diversity calculation, rarefaction, proportion, centered log-ratio, arcsine square root)"
- [discussion] the rarefaction (50) using the "rarefy_even_depth" in the "phyloseq" package: [evidence span withheld — anchor status: not_found]
- [readme] stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’): "stored the all the final processed data that we used in our paper (Gu et al., 2023) in the above Data folder as ‘phyloseq’ format (see the file named ‘biom.Rdata’)"
- [discussion] we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups: "we added the subgroup analysis user option because it is often assumed that the effects of microbiome on survival responses are different between treatment groups"
