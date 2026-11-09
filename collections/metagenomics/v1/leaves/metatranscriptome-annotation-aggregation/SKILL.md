---
name: metatranscriptome-annotation-aggregation
description: Use when you have downloaded per-read annotation results files (e.g.
  from MG-RAST v3.6 or DIAMOND BLAST-like output) and need ranked abundance summaries
  of organisms or functions for statistical comparison.
when_to_use_negative:
- Input is still raw FASTQ or preprocessed reads — the preprocessing (Trimmomatic,
  FLASH/PEAR) and MG-RAST annotation steps must complete first
- Input is already an aggregated summary table destined directly for R_organism_script.Rmd
  / DESeq2 statistical analysis
- Eukaryote-focused analysis where MG-RAST annotation quality is poor, since MG-RAST
  is not tailored to eukaryotic organisms and shows an increased mismatch rate
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SAMSA
  role: Pipeline whose Python aggregation scripts convert MG-RAST annotation output
    into abundance summary files
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server (v3.6) whose downloaded per-read results are the aggregation
    input
- name: Python 2.7
  role: Runtime for aggregation scripts (analysis_counter.py, RefSeq_output_reducer.py,
    taxonomy_shifter.py); SAMSA2 aggregation scripts also require Python2
- name: RefSeq_output_reducer.py
  role: Consolidates identical organism matches in summary files
  repo: https://github.com/transcript/SAMSA
- name: taxonomy_shifter.py
  role: Converts organism summary files from Genus level to other taxonomic levels
    using Bacteria_Genus_flattened.tsv
  repo: https://github.com/transcript/SAMSA
- name: DIAMOND_analysis_counter.py
  role: 'SAMSA2 aggregation equivalent: creates a ranked abundance summary of DIAMOND
    results per metatranscriptome file'
  repo: https://github.com/transcript/samsa2
- name: R / DESeq2
  role: Consumes the aggregated summary files for statistical significance analysis
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:f948a98514d1f54d64186647a1fa4aa1650df499d5c0ad243787dd36b4f53657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - DIAMOND_analysis_counter.py
  - FLASH
  - MG-RAST
  - Python 2.7
  - R
  - R / DESeq2
  - RefSeq_output_reducer.py
  - SAMSA
  - Trimmomatic
  - taxonomy_shifter.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metatranscriptome-annotation-aggregation

## Summary

Aggregation phase of the SAMSA pipeline: converts per-read annotation output from MG-RAST (or DIAMOND in SAMSA2) into sorted, tab-delimited organism and functional abundance summary tables that downstream R/DESeq2 analysis consumes.

## When to use

Use when you have downloaded per-read annotation results files (e.g. from MG-RAST v3.6 or DIAMOND BLAST-like output) and need ranked abundance summaries of organisms or functions for statistical comparison.

## When NOT to use

- Input is still raw FASTQ or preprocessed reads — the preprocessing (Trimmomatic, FLASH/PEAR) and MG-RAST annotation steps must complete first
- Input is already an aggregated summary table destined directly for R_organism_script.Rmd / DESeq2 statistical analysis
- Eukaryote-focused analysis where MG-RAST annotation quality is poor, since MG-RAST is not tailored to eukaryotic organisms and shows an increased mismatch rate

## Inputs

- MG-RAST annotation results files (tab-delimited per-read annotations, e.g. RefSeq organism or functional matches)
- DIAMOND per-read annotation output (SAMSA2 workflow)
- Bacteria_Genus_flattened.tsv reference database (for taxonomy_shifter.py)

## Outputs

- Sorted tab-delimited organism summary files (e.g. annotation_summary.tsv / organism summary files)
- Functional category summary files (optionally retaining m5nr IDs via analysis_counter_w-md5s.py)
- Taxonomy-shifted summary files (e.g. phylum-level tables derived from genus-level summaries)
- Thresholded summary files at a chosen cutoff percentage (long_tail_threshold.py)

## How to apply

Parse each annotated results file line by line with a Python 2.7 script, tallying each unique annotation match in a dictionary by incrementing a counter per observed hit. Sort the annotation-count dictionary in descending order of abundance and export it as a tab-delimited summary file formatted for DESeq2 in R. For organism summaries, optionally run RefSeq_output_reducer.py to consolidate identical organism matches, and use taxonomy_shifter.py with the Bacteria_Genus_flattened.tsv reference to move genus-level summaries to other taxonomic levels (e.g. phylum). Use analysis_counter_w-md5s.py when the MG-RAST m5nr ID must be retained for later md5-to-RefSeq API queries. Per the SAMSA2 README, the equivalent DIAMOND aggregation uses DIAMOND_analysis_counter.py to create a ranked abundance summary of DIAMOND results per metatranscriptome file.

## Related tools

- **SAMSA** (Pipeline whose Python aggregation scripts convert MG-RAST annotation output into abundance summary files) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server (v3.6) whose downloaded per-read results are the aggregation input)
- **Python 2.7** (Runtime for aggregation scripts (analysis_counter.py, RefSeq_output_reducer.py, taxonomy_shifter.py); SAMSA2 aggregation scripts also require Python2)
- **RefSeq_output_reducer.py** (Consolidates identical organism matches in summary files) — https://github.com/transcript/SAMSA
- **taxonomy_shifter.py** (Converts organism summary files from Genus level to other taxonomic levels using Bacteria_Genus_flattened.tsv) — https://github.com/transcript/SAMSA
- **DIAMOND_analysis_counter.py** (SAMSA2 aggregation equivalent: creates a ranked abundance summary of DIAMOND results per metatranscriptome file) — https://github.com/transcript/samsa2
- **R / DESeq2** (Consumes the aggregated summary files for statistical significance analysis)

## Examples

```
python analysis_counter.py -usage
```

## Evaluation signals

- Summary file contains exactly one row per unique annotation observed in the input annotation file
- Rows are ordered by non-increasing (descending) abundance
- Per-annotation counts equal the number of corresponding hits in the input file; recomputing the tally reproduces identical counts
- Output is tab-delimited and importable by R_organism_script.Rmd / DESeq2 without schema errors
- After taxonomy_shifter.py, reads at the target level (e.g. phylum) account for the reads present at genus level, with no dropped counts

## Limitations

- Original SAMSA aggregation scripts require Python 2.7, which is deprecated; SAMSA2 notes it 'works mostly with Python3, although there may be some errors not yet caught'
- MG-RAST annotation queue waits can run days to weeks, delaying aggregation, and a dedicated MG-RAST account is required
- Reliable abundance summaries depend on sufficient depth: 40-50 million raw sequences yielding 5-10 million annotated reads per sample

## Evidence

- [discussion] Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis.: "Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis."
- [readme] analysis_counter.py - creates a sorted summary of abundance counts from MG-RAST annotation results files: [evidence span withheld — anchor status: external_doc]
- [readme] After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2.: "After annotation is complete, the results are downloaded, aggregated into summary files, and then analyzed using R and DESeq2."
- [readme] taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [abstract] Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures.: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
