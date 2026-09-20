---
name: r-based-annotation-summary-analysis
description: Use when you have aggregated MG-RAST (or DIAMOND) annotation summary
  files (organism or functional) from one or more metatranscriptome samples and need
  statistical significance testing, abundance ranking, or graphical/tabular summaries
  in R.
when_to_use_negative:
- Input is still raw MG-RAST annotation output or per-read DIAMOND results rather
  than aggregated summary files — run aggregation scripts first
- Data are from eukaryote-dominated samples where MG-RAST annotation is not tailored
  and mismatch rates are elevated
- The analysis requires measuring microbial population size shifts as accurately as
  shotgun metagenomics
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: R_organism_script.Rmd
  role: Imports organism summary files and performs statistical significance correlations
    between sample groups
  repo: github.com/transcript/SAMSA
- name: R_functional_script_w-md5s.Rmd
  role: Imports functional category summary files (with m5nr IDs) and performs statistical
    significance correlations
  repo: github.com/transcript/SAMSA
- name: taxonomy_shifter.py
  role: Converts genus-level organism summary files to other taxonomic levels (using
    Bacteria_Genus_flattened.tsv) before R analysis
  repo: github.com/transcript/SAMSA
- name: run_DESeq_stats.R
  role: SAMSA2 command-line R script using DESeq to compute significantly differing
    features between control vs. experimental groups
  repo: https://www.github.com/transcript/SAMSA2
- name: package_install.Rmd
  role: Installs all necessary R packages required for the R analysis scripts
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:9c85f8c7be6fa6dcf79e849c99c8df138c708447337c9de5de7a695077265f5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R_functional_script_w-md5s.Rmd
  - R_organism_script.Rmd
  - RefSeq_output_reducer.py
  - SAMSA
  - package_install.Rmd
  - run_DESeq_stats.R
  - taxonomy_shifter.py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 22
---

# r-based-annotation-summary-analysis

## Summary

Import SAMSA-generated organism or functional annotation summary files into R (preferably RStudio) and run DESeq2-based statistical analysis to identify abundant species, significant functional differences, and taxonomic-level depletion patterns, with tabular or graphical output.

## When to use

Use when you have aggregated MG-RAST (or DIAMOND) annotation summary files (organism or functional) from one or more metatranscriptome samples and need statistical significance testing, abundance ranking, or graphical/tabular summaries in R.

## When NOT to use

- Input is still raw MG-RAST annotation output or per-read DIAMOND results rather than aggregated summary files — run aggregation scripts first
- Data are from eukaryote-dominated samples where MG-RAST annotation is not tailored and mismatch rates are elevated
- The analysis requires measuring microbial population size shifts as accurately as shotgun metagenomics

## Inputs

- Organism summary files produced by SAMSA aggregation (e.g. RefSeq_output_reducer.py output)
- Functional category summary files (optionally with m5nr IDs from analysis_counter_w-md5s.py)
- Taxonomy-shifted summary files (e.g. phylum-level, generated with Bacteria_Genus_flattened.tsv)
- Sample metadata / control vs. experimental group design (e.g. ribodepleted vs. non-ribodepleted samples)

## Outputs

- Tabular results with p-values and log2FoldChange scores per feature (organism or function)
- Graphical output (plots) of organism-specific or functional activity
- Comparison tables such as per-phylum rRNA depletion percentages and flagged phyla with little or no reduction

## How to apply

After the SAMSA aggregation step produces summary files (e.g. via analysis_counter.py, RefSeq_output_reducer.py, or func_data_trimmer.py), import them into R/RStudio using the provided R scripts: R_organism_script.Rmd for organism summary files and R_functional_script_w-md5s.Rmd for functional category summary files. These scripts perform statistical significance correlations with DESeq2, computing p-values and log2FoldChange scores for each feature (organism or function) between control and experimental groups. For taxonomic-level questions, first run taxonomy_shifter.py with the Bacteria_Genus_flattened.tsv reference database to convert genus-level organism summaries to the desired level (e.g. phylum) before importing into R. Optionally apply long_tail_threshold.py to threshold summary files at a chosen cutoff percentage. Results can be viewed in graphical or tabular format focusing on organism-specific or functional activity; install required packages first with package_install.Rmd.

## Related tools

- **R_organism_script.Rmd** (Imports organism summary files and performs statistical significance correlations between sample groups) — github.com/transcript/SAMSA
- **R_functional_script_w-md5s.Rmd** (Imports functional category summary files (with m5nr IDs) and performs statistical significance correlations) — github.com/transcript/SAMSA
- **taxonomy_shifter.py** (Converts genus-level organism summary files to other taxonomic levels (using Bacteria_Genus_flattened.tsv) before R analysis) — github.com/transcript/SAMSA
- **run_DESeq_stats.R** (SAMSA2 command-line R script using DESeq to compute significantly differing features between control vs. experimental groups) — https://www.github.com/transcript/SAMSA2
- **package_install.Rmd** (Installs all necessary R packages required for the R analysis scripts) — github.com/transcript/SAMSA

## Examples

```
Rscript run_DESeq_stats.R --input organism_summary_files/ --output deseq_results.tsv
```

## Evaluation signals

- R scripts execute without missing-package errors after running package_install.Rmd, and summary files import with expected feature counts
- DESeq2 output contains p-values and log2FoldChange scores for every feature and flags significant differences between control and experimental groups
- For the ribodepletion use case, reproduced overall rRNA reduction falls within the reported 63-82% range and the output identifies Actinobacteria, Cyanobacteria, and Spirochaetes as phyla with little or no depletion
- Taxonomy-shifted inputs align with Bacteria_Genus_flattened.tsv mappings (genus-to-phylum conversion is complete and consistent)
- Graphical and tabular outputs agree on which species or functions are most abundant and significantly different

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, delaying summary file generation before R analysis
- Insufficient mRNA reads in a dataset can prevent direct mRNA/rRNA correlation, weakening conclusions drawn from the R comparison
- Shifts in microbial population sizes cannot be measured as accurately as with shotgun metagenomics; SAMSA2 aggregation scripts require Python2 and may throw uncaught errors under Python3

## Evidence

- [methods] R_organism_script.Rmd imports organism summary files and performs statistical significance correlations: "R_organism_script.Rmd - imports organism summary files and performs statistical significance correlations"
- [readme] R scripts use DESeq to compute most significantly different features between control vs. experimental samples, with p-values and log2FoldChange: [evidence span withheld — anchor status: external_doc]
- [methods] taxonomy_shifter.py converts organism summary files (initially at Genus level) to other taxonomic levels for analysis: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [abstract] SAMSA can summarize and evaluate raw annotation results, identifying abundant species and significant functional differences: "SAMSA can summarize and evaluate raw annotation results, identifying abundant species and significant functional differences between metatranscriptomes."
- [readme] After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
