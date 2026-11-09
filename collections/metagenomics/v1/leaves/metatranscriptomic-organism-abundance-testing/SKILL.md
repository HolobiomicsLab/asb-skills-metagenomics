---
name: metatranscriptomic-organism-abundance-testing
description: Use when you have MG-RAST (or DIAMOND) annotation results for multiple
  metatranscriptome samples in control vs. experimental groups and need to identify
  organisms whose mRNA abundance differs significantly between groups.
when_to_use_negative:
- Input is raw FASTQ or per-read annotation output that has not yet been aggregated
  into organism summary files — preprocessing/aggregation (Trimmomatic, FLASH/PEAR,
  MG-RAST annotation, analysis_counter) must come first
- The question is functional (pathway/subsystem) rather than organism-level differential
  activity — use R_functional_script_w-md5s.Rmd on functional summary files instead
- You need precise measurements of microbial population size shifts — the paper notes
  mRNA-based metatranscriptomics cannot measure population size shifts as accurately
  as shotgun metagenomics
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3308
tools:
- name: R_organism_script.Rmd
  role: Imports organism summary files and performs statistical significance correlations
    (the core organism-level DESeq2 test)
  repo: https://github.com/transcript/SAMSA
- name: DESeq2
  role: Negative-binomial normalization and differential abundance model fit between
    the two experimental groups
- name: long_tail_threshold.py
  role: Thresholds summary files at a chosen cutoff percentage to remove rare low-abundance
    taxa before testing
  repo: https://github.com/transcript/SAMSA
- name: taxonomy_shifter.py
  role: Converts organism summary files (initially at Genus level) to other taxonomic
    levels for analysis
  repo: https://github.com/transcript/SAMSA
- name: RefSeq_output_reducer.py
  role: Consolidates identical organism matches in summary files before statistics
  repo: https://github.com/transcript/SAMSA
- name: SAMSA
  role: 'Parent pipeline: preprocessing, MG-RAST annotation, aggregation, and analysis;
    also SAMSA2 with DIAMOND as a local alternative'
  repo: https://github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:d8e6a15d20125f39d5334c3a37b6bcd3e3515fea3f0194dca8cd4ef11bb77f9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - DESeq2
  - R
  - R_organism_script.Rmd
  - RefSeq_output_reducer.py
  - SAMSA
  - long_tail_threshold.py
  - taxonomy_shifter.py
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metatranscriptomic-organism-abundance-testing

## Summary

Statistical testing of organism-level transcriptional activity between experimental groups using SAMSA-aggregated organism summary files analyzed with R and DESeq2, validated on Tyk2 knockout vs. wild-type mouse DSS colitis gut metatranscriptomes.

## When to use

Use when you have MG-RAST (or DIAMOND) annotation results for multiple metatranscriptome samples in control vs. experimental groups and need to identify organisms whose mRNA abundance differs significantly between groups.

## When NOT to use

- Input is raw FASTQ or per-read annotation output that has not yet been aggregated into organism summary files — preprocessing/aggregation (Trimmomatic, FLASH/PEAR, MG-RAST annotation, analysis_counter) must come first
- The question is functional (pathway/subsystem) rather than organism-level differential activity — use R_functional_script_w-md5s.Rmd on functional summary files instead
- You need precise measurements of microbial population size shifts — the paper notes mRNA-based metatranscriptomics cannot measure population size shifts as accurately as shotgun metagenomics

## Inputs

- Aggregated organism summary files from SAMSA (analysis_counter.py or RefSeq_output_reducer.py output)
- Sample metadata grouping samples into control vs. experimental groups (e.g., 9 Tyk2−/− vs. 6 wild-type)
- Optionally, functional category summary files for paired functional testing via R_functional_script_w-md5s.Rmd

## Outputs

- Tabular organism-level differential abundance results with p-values and fold changes (adjusted p < 0.05 significance calls)
- Summary of significantly differing taxa, e.g., 1.558-fold Enterobacteriaceae increase in day 3 Tyk2−/− samples

## How to apply

Import the aggregated organism summary files into R with R_organism_script.Rmd (R/RStudio). Optionally restrict rare taxa using long_tail_threshold.py, which thresholds summary files at a chosen cutoff percentage, to remove low-abundance noise before testing; taxonomy_shifter.py with Bacteria_Genus_flattened.tsv can convert Genus-level summaries to other taxonomic levels. Normalize read counts and fit the DESeq2 negative-binomial model comparing groups (e.g., 9 Tyk2−/− samples vs. 6 wild-type controls). Correct for multiple testing and extract organisms with adjusted p < 0.05; in the Tyk2 example this recovered a 1.558-fold increase in Enterobacteriaceae (p = 0.0055), increased Escherichia and Providencia, and decreased Butyrivibrio (p = 0.03). Write organism-level significance results (p-values, fold changes) to tabular output files.

## Related tools

- **R_organism_script.Rmd** (Imports organism summary files and performs statistical significance correlations (the core organism-level DESeq2 test)) — https://github.com/transcript/SAMSA
- **DESeq2** (Negative-binomial normalization and differential abundance model fit between the two experimental groups)
- **long_tail_threshold.py** (Thresholds summary files at a chosen cutoff percentage to remove rare low-abundance taxa before testing) — https://github.com/transcript/SAMSA
- **taxonomy_shifter.py** (Converts organism summary files (initially at Genus level) to other taxonomic levels for analysis) — https://github.com/transcript/SAMSA
- **RefSeq_output_reducer.py** (Consolidates identical organism matches in summary files before statistics) — https://github.com/transcript/SAMSA
- **SAMSA** (Parent pipeline: preprocessing, MG-RAST annotation, aggregation, and analysis; also SAMSA2 with DIAMOND as a local alternative) — https://github.com/transcript/SAMSA

## Examples

```
Rscript run_DESeq_stats.R
```

## Evaluation signals

- DESeq2 multiple-testing-corrected p-values computed; significant organisms meet adjusted p < 0.05
- Tyk2 benchmark reproduced: 1.558-fold Enterobacteriaceae increase in day 3 samples (p = 0.0055), increased Escherichia and Providencia, decreased Butyrivibrio (p = 0.03), >300 differentially expressed protein-coding transcripts
- Summary files import correctly into R without missing/malformed organism rows after consolidation by RefSeq_output_reducer.py
- Thresholded summaries (long_tail_threshold.py) retain the tested taxa while removing low-abundance long-tail noise
- Results written to tabular output with fold change and significance columns for each organism

## Limitations

- Organism transcriptional activity from mRNA counts reflects active transcription, not microbial population size, which cannot be measured as accurately as with shotgun metagenomics
- Only taxa with sufficient annotated mRNA reads (5–10 million annotated reads per sample from 40–50 million raw sequences) yield accurate abundance measures
- SAMSA v1 depends on MG-RAST annotation servers, where queue waits may delay results by days to weeks; MG-RAST is not tailored to eukaryotic organisms
- Rare taxa in the long tail of abundance distributions can add low-abundance noise unless thresholded before testing

## Evidence

- [abstract] we were able to demonstrate that overall organism transcriptional activity can be measured using mRNA counts: "we were able to demonstrate that overall organism transcriptional activity can be measured using mRNA counts"
- [methods] R_organism_script.Rmd imports organism summary files and performs statistical significance correlations: "R_organism_script.Rmd - imports organism summary files and performs statistical significance correlations"
- [methods] long_tail_threshold.py permits thresholding of summary files at a chosen cutoff percentage: "long_tail_threshold.py - permits thresholding of summary files at a chosen cutoff percentage"
- [readme] After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2.: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
- [discussion] shifts in microbial population sizes cannot be measured as accurately as using shotgun metagenomics: "shifts in microbial population sizes cannot be measured as accurately as using shotgun metagenomics"
