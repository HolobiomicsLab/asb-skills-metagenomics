---
name: interactive-manhattan-plot-visualization
description: Use when signatures-of-selection scan results (iHS, XP-EHH, SweepFinder2
  CLR values) from scalepopgen need to be visualized as interactive per-population
  Manhattan plots to inspect candidate regions under selection.
when_to_use_negative:
- The underlying selection scan has not yet been run — the plots require iHS/XP-EHH
  and SweepFinder2 result files as input, not raw VCFs.
- The analysis goal is publication-quality static figures rather than interactive
  exploration of flagged selection regions.
- The task concerns the other scalepopgen sub-workflows (filtering/basic statistics,
  genetic structure, TreeMix phylogeny) rather than signatures-of-selection results.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: Python
  role: language of the workflow's result-collection and visualization scripts
  repo: https://github.com/Popgen48/scalepopgen
- name: Plotly
  role: rendering the interactive Manhattan plots of selection scan results
  repo: https://github.com/Popgen48/scalepopgen
- name: selscan
  role: produces iHS and XP-EHH scan results that are visualized
  repo: https://github.com/Popgen48/scalepopgen
- name: SweepFinder2
  role: produces per-population composite likelihood ratio scan results that are visualized
  repo: https://github.com/Popgen48/scalepopgen
- name: Nextflow
  role: orchestrates the sub-workflow that runs the scans and the plotting scripts
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:1ff288fce042a46b05529fa4a2e5821daf5e503e71c62ef04f35fbae8c5fbff5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - BEAGLE
  - Conda
  - Nextflow
  - Plotly
  - Python
  - SHAPEIT5
  - Singularity
  - SweepFinder2
  - est-sfs
  - selscan
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# interactive-manhattan-plot-visualization

## Summary

Generate interactive Manhattan plots with the scalepopgen workflow's Python (Plotly) scripts to visualize results of the signatures-of-selection scans (iHS, XP-EHH, SweepFinder2), flagging genomic regions above a user-defined selection threshold with Ensembl location links when species annotation is available.

## When to use

Use when signatures-of-selection scan results (iHS, XP-EHH, SweepFinder2 CLR values) from scalepopgen need to be visualized as interactive per-population Manhattan plots to inspect candidate regions under selection.

## When NOT to use

- The underlying selection scan has not yet been run — the plots require iHS/XP-EHH and SweepFinder2 result files as input, not raw VCFs.
- The analysis goal is publication-quality static figures rather than interactive exploration of flagged selection regions.
- The task concerns the other scalepopgen sub-workflows (filtering/basic statistics, genetic structure, TreeMix phylogeny) rather than signatures-of-selection results.

## Inputs

- selscan iHS result files per population (from phased biallelic SNP haplotypes)
- selscan XP-EHH result files per population pair
- SweepFinder2 composite likelihood ratio (CLR) result files per population
- user-defined selection threshold parameter

## Outputs

- interactive Manhattan plots (Plotly) for each analysis except XP-EHH and each population
- Ensembl location links for regions above the user-defined selection threshold (when species annotation is available)

## How to apply

After running the scalepopgen signatures-of-selection sub-workflow (Beagle/SHAPEIT5 phasing, selscan iHS and XP-EHH, SweepFinder2 CLR), collect the scan result files and pass them to the workflow's Python scripts built on Plotly. Produce one interactive Manhattan plot for each analysis except XP-EHH and for each population, since XP-EHH is a pairwise population comparison. Mark regions whose statistic exceeds the user-defined selection threshold and, if annotation resources for the species are present in the Ensembl database, attach Ensembl location links to those flagged regions. Verify that plots are generated for every expected analysis/population combination and that flagged regions match the threshold applied to the result files.

## Related tools

- **Python** (language of the workflow's result-collection and visualization scripts) — https://github.com/Popgen48/scalepopgen
- **Plotly** (rendering the interactive Manhattan plots of selection scan results) — https://github.com/Popgen48/scalepopgen
- **selscan** (produces iHS and XP-EHH scan results that are visualized) — https://github.com/Popgen48/scalepopgen
- **SweepFinder2** (produces per-population composite likelihood ratio scan results that are visualized) — https://github.com/Popgen48/scalepopgen
- **Nextflow** (orchestrates the sub-workflow that runs the scans and the plotting scripts) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume
```

## Evaluation signals

- An interactive Manhattan plot file is generated for each analysis except XP-EHH and for each population.
- Regions flagged as under selection correspond exactly to positions whose statistic exceeds the user-defined selection threshold in the selscan/SweepFinder2 result files.
- When species annotation exists in Ensembl, flagged regions carry Ensembl location links; when it does not, no links are added.
- The plotted statistics match the values in the collected scan result files (iHS, XP-EHH, SweepFinder2 CLR).
- Running the workflow with -resume reproduces the same set of plots for the same inputs and threshold.

## Limitations

- XP-EHH results are excluded from per-analysis Manhattan plotting because it is a pairwise population comparison; plotting coverage is 'each analysis except XP-EHH'.
- Ensembl location links for flagged regions are only available when annotation resources for the species under investigation are present in the Ensembl database.
- The threshold is user-defined, so the set of flagged regions depends entirely on that parameter choice.

## Evidence

- [methods] Collect the scan results and produce interactive Manhattan plots with the workflow's Python scripts (Plotly) for each analysis except XP-EHH and each population: [evidence span withheld — anchor status: not_found]
- [methods] with Ensembl location links for regions above the user-defined selection threshold when species annotation is available: [evidence span withheld — anchor status: not_found]
- [methods] if the annotation resources for the species under investigation are present in the Ensembl database: "if the annotation resources for the species under investigation are present in the Ensembl database"
- [abstract] several Python and R scripts are also provided to collect and visualize the results: "several Python and R scripts are also provided to collect and visualize the results"
- [abstract] implementing single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection: "implementing single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection"
