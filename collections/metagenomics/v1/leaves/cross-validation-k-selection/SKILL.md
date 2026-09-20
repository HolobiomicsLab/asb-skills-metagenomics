---
name: cross-validation-k-selection
description: Use when inferring population structure from genome-wide biallelic SNP
  data (VCF- or PLINK-derived BED files) and the number of ancestral populations (K)
  is unknown and must be estimated.
when_to_use_negative:
- The number of ancestral populations (K) is already known or fixed by the study design
  — run ADMIXTURE at the chosen K only.
- The goal is phylogenetic relationship inference or migration-edge analysis — use
  the TreeMix sub-workflow instead.
- The SNP set is not LD-pruned and no recoding of non-integer chromosome IDs to integers
  has been done, so tools expecting integer chromosome IDs or linkage equilibrium
  cannot read the data correctly.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3299
tools:
- name: ADMIXTURE
  role: Runs ancestry inference over the user-defined range of K values on the PLINK
    BED file; each run's log provides the CV error used for K selection (v 7.0.2 in
    the pipeline)
  repo: https://github.com/Popgen48/scalepopgen
- name: PLINK
  role: Converts VCF to binary PED (BED) format, recodes non-integer chromosome IDs
    to integers, and LD-prunes SNPs with user-defined LD parameters (PLINK2) before
    structure analysis
  repo: https://github.com/Popgen48/scalepopgen
- name: Python
  role: Pipeline scripts collect CV errors from ADMIXTURE logs, plot CV error vs.
    K, and plot the optimal-K Q-matrix with Bokeh/Plotly
  repo: https://github.com/Popgen48/scalepopgen
- name: PONG
  role: Interactive visualization of ADMIXTURE Q-matrices; the workflow generates
    PONG input files (used together with pophelper for Q-matrix plotting)
  repo: https://github.com/Popgen48/scalepopgen
- name: Nextflow
  role: Workflow management system orchestrating the explore-genetic-structure sub-workflow
    including ADMIXTURE runs and K selection
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:d555d005b728cbda6abe988cdf688698c2640a713faa7f54749bbd0eefb46392
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - ADMIXTURE
  - Bokeh
  - Eigensoft
  - Nextflow
  - PLINK
  - PONG
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-validation-k-selection

## Summary

Select the optimal number of ancestral clusters (K) in ADMIXTURE by running ADMIXTURE over a user-defined range of K on a LD-pruned PLINK BED file, parsing each run's log for the cross-validation (CV) error, and choosing the K with the lowest CV error for downstream Q-matrix visualization.

## When to use

Use when inferring population structure from genome-wide biallelic SNP data (VCF- or PLINK-derived BED files) and the number of ancestral populations (K) is unknown and must be estimated.

## When NOT to use

- The number of ancestral populations (K) is already known or fixed by the study design — run ADMIXTURE at the chosen K only.
- The goal is phylogenetic relationship inference or migration-edge analysis — use the TreeMix sub-workflow instead.
- The SNP set is not LD-pruned and no recoding of non-integer chromosome IDs to integers has been done, so tools expecting integer chromosome IDs or linkage equilibrium cannot read the data correctly.

## Inputs

- Filtered PLINK binary (BED/BIM/FAM) genotype files listed in a plink_input.csv sheet (prefix,bed,bim,fam)
- Tab-delimited sample map file (individual IDs and population IDs, extension .map, no header)
- User-defined range of K values (number of ancestral components)
- User-defined PLINK2 LD-pruning parameters

## Outputs

- CV-error table/plot containing a CV-error value for every tested K
- ADMIXTURE Q-matrices for each tested K
- Interactive Q-matrix plot (Bokeh/Plotly) of ancestry proportions for the optimal K
- PONG input files for interactive visualization of ADMIXTURE results

## How to apply

First LD-prune the SNP set with PLINK2 using user-defined LD parameters, since ADMIXTURE assumes loci are in linkage equilibrium; if chromosome IDs are non-integer, recode them to integers with PLINK. Run ADMIXTURE (v 7.0.2) on the filtered PLINK binary (BED) genotype file across a user-defined range of K (number of ancestral components). Parse each run's log file to collect the CV error, then plot CV error against K to identify a suitable value of K; the K with the lowest CV error is selected. Plot the optimal K's Q-matrix with the pipeline's Python Bokeh/Plotly scripts keeping user-supplied or random color-population pairs consistent, and generate PONG input files for interactive visualization of all Q-matrices (with tools such as pong and pophelper).

## Related tools

- **ADMIXTURE** (Runs ancestry inference over the user-defined range of K values on the PLINK BED file; each run's log provides the CV error used for K selection (v 7.0.2 in the pipeline)) — https://github.com/Popgen48/scalepopgen
- **PLINK** (Converts VCF to binary PED (BED) format, recodes non-integer chromosome IDs to integers, and LD-prunes SNPs with user-defined LD parameters (PLINK2) before structure analysis) — https://github.com/Popgen48/scalepopgen
- **Python** (Pipeline scripts collect CV errors from ADMIXTURE logs, plot CV error vs. K, and plot the optimal-K Q-matrix with Bokeh/Plotly) — https://github.com/Popgen48/scalepopgen
- **PONG** (Interactive visualization of ADMIXTURE Q-matrices; the workflow generates PONG input files (used together with pophelper for Q-matrix plotting)) — https://github.com/Popgen48/scalepopgen
- **Nextflow** (Workflow management system orchestrating the explore-genetic-structure sub-workflow including ADMIXTURE runs and K selection) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile docker -params-file parameters.yml -qs 10
```

## Evaluation signals

- The CV-error table contains a CV-error value for every tested K in the user-defined range.
- The selected optimal K is the one with the lowest CV error across the tested range.
- The Q-matrix plot renders ancestry proportions for all samples at the optimal K.
- Color-population pairs in the Q-matrix plot remain consistent (user-supplied or randomly allocated).
- Breed clustering in the case study (442 cattle WGS samples, 30 breeds) is qualitatively consistent with the published results.

## Limitations

- Chromosome IDs must be integers for downstream tools; non-integer IDs must first be recoded (e.g. with PLINK).
- ADMIXTURE should be run on LD-pruned data (user-defined PLINK2 LD parameters) since it assumes linkage equilibrium; unpruned data can bias structure inference.
- CV-error-based K selection identifies 'a suitable value of K' — the lowest-CV K may not match biologically expected population subdivision and should be checked against Q-matrix plots.

## Evidence

- [intro] plot cross-validation (CV) error to identify a suitable value of K: "plot cross-validation (CV) error to identify a suitable value of K"
- [intro] plot Q-matrices of ADMIXTURE outputs using tools such as pong and pophelper: "plot Q-matrices of ADMIXTURE outputs using tools such as pong (Behr et al. 2016) and pophelper (Francis 2017)"
- [methods] user-defined pruning of the data based on LD parameters of Plink2: "user-defined pruning of the data based on LD parameters of Plink2 (Chang et al. 2015), before supplying the input file to its respective analysis"
- [abstract] principal component analysis, admixture with identifying optimal K-values: "(ii) principal component analysis, admixture with identifying optimal K-values"
- [intro] conversion of VCF to binary PED bed (BED) file format using plink: "conversion of VCF to binary PED bed (BED) file format using plink (Purcell et al. 2007)"
