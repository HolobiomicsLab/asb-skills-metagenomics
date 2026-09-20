---
name: colorectal-cancer-metagenome-phenotype-comparison
description: Use when comparing metagenomic sample groups (e.g., healthy vs adenoma
  vs carcinoma) to identify taxa, genes, KOs, EC numbers, or reactions that differ
  significantly across phenotypes in a mapping-based analysis.
when_to_use_negative:
- Input is raw unmapped reads or assemblies — normalization and frequency tables from
  the mapping/CSS stages of CAMAMED must already exist
- The study design is single-cohort with no case-control group structure to compare
  across
- A compositional-bias-aware differential method suited to the specific hypothesis
  is required but the data have not passed through CSS correction; the article found
  results from the two colorectal cancer datasets were not consistent without applying
  the CSS
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: CAMAMED
  role: Provides the med_pipeline environment, CSS-normalized five-level tables, and
    the statistical testing/FDR stage executed end-to-end
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: camamed_pipeline / camamed_pipeline_db images run the comparison stage with
    no separate dependency installation
  repo: https://hub.docker.com
- name: metagenomeSeq
  role: Underpins the normalization used for compositional bias handling in sparse
    data at taxa, gene, KO, EC number and reaction levels before group testing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:2c33cd2376e8e6b48a9cbb79a067b569de91fa243c1669e7a08e056290043f35
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# colorectal-cancer-metagenome-phenotype-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs CAMAMED's comparative case-control analysis stage: loading CSS-normalized five-level frequency tables (taxon, gene, KO, EC number, reaction), testing each feature across healthy/adenoma/carcinoma groups with the Kruskal-Wallis H test (or ANOVA when normality is acceptable), and applying Benjamini-Hochberg FDR correction to flag significantly changed components and potential biomarkers.

## When to use

Use when comparing metagenomic sample groups (e.g., healthy vs adenoma vs carcinoma) to identify taxa, genes, KOs, EC numbers, or reactions that differ significantly across phenotypes in a mapping-based analysis.

## When NOT to use

- Input is raw unmapped reads or assemblies — normalization and frequency tables from the mapping/CSS stages of CAMAMED must already exist
- The study design is single-cohort with no case-control group structure to compare across
- A compositional-bias-aware differential method suited to the specific hypothesis is required but the data have not passed through CSS correction; the article found results from the two colorectal cancer datasets were not consistent without applying the CSS

## Inputs

- CSS-normalized five-level frequency tables from CAMAMED med_pipeline (taxon, gene, KO, EC number, reaction)
- Phenotype/group labels for metagenomic samples (healthy, adenoma, carcinoma)
- Raw metagenomic sequences previously mapped to non-redundant gene catalogs

## Outputs

- differential_abundance_results.csv with per-feature identifier, level, test statistic, raw P-value, and BH-adjusted P-value
- List of significantly changed components / candidate biomarkers at taxon, gene, KO, EC number, and reaction levels

## How to apply

First obtain the normalized five-level frequency tables (taxon, gene, KO, EC number, reaction) emitted by the upstream CAMAMED med_pipeline, which applies cumulative sum-scaling (CSS) at the taxa and gene levels to correct compositional bias in sparse data. For each feature row, run the non-parametric Kruskal-Wallis H test across the three phenotype groups, recording the test statistic and raw P-value; substitute a one-way ANOVA only where normality is judged acceptable. Apply the Benjamini-Hochberg procedure to the raw P-value vector to obtain FDR-adjusted P-values. Write per-feature results (feature identifier, level, statistic, raw P, BH-adjusted P) to differential_abundance_results.csv. Execute inside the CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db) so no separate software dependencies need to be installed.

## Related tools

- **CAMAMED** (Provides the med_pipeline environment, CSS-normalized five-level tables, and the statistical testing/FDR stage executed end-to-end) — https://github.com/mhnb/camamed
- **Docker** (camamed_pipeline / camamed_pipeline_db images run the comparison stage with no separate dependency installation) — https://hub.docker.com
- **metagenomeSeq** (Underpins the normalization used for compositional bias handling in sparse data at taxa, gene, KO, EC number and reaction levels before group testing)

## Examples

```
python camamed_init
```

## Evaluation signals

- Every feature row across all five levels carries a test statistic, raw P-value, and BH-adjusted P-value in differential_abundance_results.csv
- Number of FDR-adjusted P-values equals the number of tested features per level (BH applied to the whole raw P-value vector)
- Adjusted P-values are monotone non-decreasing with raw P-values and lie in [0,1]
- Group labels (healthy, adenoma, carcinoma) match exactly the phenotype assignment used by the med_pipeline stage
- Execution completed inside the CAMAMED Docker image with no external dependency installation

## Limitations

- The two colorectal cancer gut metagenome datasets used in validation are not identified by accession, hindering direct reproduction
- No software version stated for CAMAMED or its component tools, complicating exact reproducibility
- Without applying the CSS normalization step, results from the two colorectal cancer datasets were not consistent, so the comparison stage depends critically on correct upstream normalization
- One-way ANOVA substitution requires a normality judgment; the default is the non-parametric Kruskal-Wallis H test

## Evidence

- [full_text] CAMAMED identifies components that are changed significantly in different sample groups using the non-parametric Kruskal-Wallis H test: [evidence span withheld — anchor status: not_found]
- [abstract] the pipeline enables the user to identify potential biomarkers in case-control metagenomic samples: "the pipeline enables the user to identify potential biomarkers in case-control metagenomic samples by investigating functional differences"
- [intro] CAMAMED uses metagenomeSeq for compositional bias handling in sparse data at all five levels: "CAMAMED uses the metagenomeSeq due to its high capability of handling compositional bias in sparse metagenomic data in taxa, gene, KO, EC number and reaction levels"
- [discussion] without applying the CSS, the results from these datasets were not consistent: "However, without applying the CSS, the results from these datasets were not consistent"
- [readme] Docker images avoid separate dependency installation: "Note: There is no need to install any software dependencies if you use the Docker image."
