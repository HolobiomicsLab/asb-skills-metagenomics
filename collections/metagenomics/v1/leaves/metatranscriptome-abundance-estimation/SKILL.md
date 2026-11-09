---
name: metatranscriptome-abundance-estimation
description: Use when raw FASTQ metatranscriptome reads (paired-end or >100 bp) must
  be converted into organism abundance summaries, or when mRNA-derived abundance must
  be validated against rRNA-derived abundance in non-ribodepleted samples.
when_to_use_negative:
- Samples are already ribodepleted — remaining rRNA reads are meant to be discarded,
  so no mRNA/rRNA correlation validation is possible
- Samples have fewer than ~40-50 million raw reads, yielding under the 5-10 million
  annotated reads needed for accurate abundance measures
- The goal is quantitative microbial population sizing — metatranscriptomics measures
  transcriptional activity, not population size shifts as accurately as shotgun metagenomics
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: MG-RAST
  role: Annotation server that assigns organism (RefSeq) and rRNA (SILVA SSU) annotations
    to metatranscriptome reads; requires an MG-RAST account and supports batch download
    via MG-RAST_API_downloader.py
  repo: github.com/transcript/SAMSA
- name: SAMSA
  role: Pipeline providing aggregation scripts (analysis_counter_w-md5s.py, RefSeq_output_reducer.py,
    taxonomy_shifter.py) and R analysis scripts that turn MG-RAST output into abundance
    summaries and correlations
  repo: github.com/transcript/SAMSA
- name: R
  role: Runs R_organism_script.Rmd to import organism summary files, compute Pearson
    correlations across taxonomic levels, and perform DESeq2 statistical testing
  repo: github.com/transcript/SAMSA
- name: Python 2.7
  role: Runs the SAMSA aggregation scripts that condense per-read annotation results
    into sorted abundance summary files
  repo: github.com/transcript/SAMSA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:ba64dfc4ac6bf23708db0c233af1a0adec080bf86409e90a2603587dbb5e4a98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - SAMSA
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metatranscriptome-abundance-estimation

## Summary

Estimates organism abundance and transcriptional activity from metatranscriptomic data using SAMSA aggregation scripts over MG-RAST/RefSeq annotations, including validating mRNA-based abundance against rRNA (SILVA SSU)-based estimates in non-ribodepleted samples.

## When to use

Use when raw FASTQ metatranscriptome reads (paired-end or >100 bp) must be converted into organism abundance summaries, or when mRNA-derived abundance must be validated against rRNA-derived abundance in non-ribodepleted samples.

## When NOT to use

- Samples are already ribodepleted — remaining rRNA reads are meant to be discarded, so no mRNA/rRNA correlation validation is possible
- Samples have fewer than ~40-50 million raw reads, yielding under the 5-10 million annotated reads needed for accurate abundance measures
- The goal is quantitative microbial population sizing — metatranscriptomics measures transcriptional activity, not population size shifts as accurately as shotgun metagenomics

## Inputs

- Raw FASTQ metatranscriptome files (paired-end or long reads >100 bp)
- MG-RAST annotation results files (RefSeq organism matches and SILVA SSU rRNA matches)
- Bacteria_Genus_flattened.tsv taxonomy reference database

## Outputs

- Sorted organism abundance summary files (aggregated by analysis_counter_w-md5s.py and RefSeq_output_reducer.py)
- Family- and order-level organism summaries from taxonomy_shifter.py
- Pearson correlation values between mRNA- and rRNA-based abundances per taxonomic level (e.g. written to correlation_results.tsv)
- DESeq2 statistical significance tables with p-values and log2FoldChange scores

## How to apply

Preprocess FASTQ files with Trimmomatic (adaptor/quality trimming) and FLASH or PEAR (paired-end joining), requiring 40-50 million raw reads per sample to yield the 5-10 million annotated reads needed for accurate abundance. Submit cleaned reads to MG-RAST for annotation, then download results and run SAMSA aggregation scripts (analysis_counter_w-md5s.py to preserve m5nr IDs, RefSeq_output_reducer.py to consolidate identical organism matches). Build separate organism summary tables from RefSeq-derived mRNA matches and SILVA SSU-derived rRNA matches, filter both to bacterial annotations only, and use taxonomy_shifter.py with Bacteria_Genus_flattened.tsv to shift genus-level summaries to family and order levels. Finally, compute Pearson correlations between mRNA- and rRNA-based abundances per taxonomic level in R via R_organism_script.Rmd (with DESeq2 for differential testing between control and experimental groups).

## Related tools

- **MG-RAST** (Annotation server that assigns organism (RefSeq) and rRNA (SILVA SSU) annotations to metatranscriptome reads; requires an MG-RAST account and supports batch download via MG-RAST_API_downloader.py) — github.com/transcript/SAMSA
- **SAMSA** (Pipeline providing aggregation scripts (analysis_counter_w-md5s.py, RefSeq_output_reducer.py, taxonomy_shifter.py) and R analysis scripts that turn MG-RAST output into abundance summaries and correlations) — github.com/transcript/SAMSA
- **R** (Runs R_organism_script.Rmd to import organism summary files, compute Pearson correlations across taxonomic levels, and perform DESeq2 statistical testing) — github.com/transcript/SAMSA
- **Python 2.7** (Runs the SAMSA aggregation scripts that condense per-read annotation results into sorted abundance summary files) — github.com/transcript/SAMSA

## Examples

```
python analysis_counter_w-md5s.py annotated_results_file.txt organism_summary.txt && python RefSeq_output_reducer.py organism_summary.txt reduced_summary.txt
```

## Evaluation signals

- Each sample yields 5-10 million annotated reads from 40-50 million raw sequences; fewer suggests insufficient depth
- mRNA- vs rRNA-based organism abundances should show average Pearson correlations of ~0.99 (order), ~0.85 (family), and ~0.81 (genus) across bacterial-filtered annotations
- Summary files are sorted by abundance and identical organism matches have been consolidated by RefSeq_output_reducer.py
- DESeq2 outputs contain assigned p-values and log2FoldChange scores for each organism or functional feature
- Taxonomy-shifted summaries at family/order levels are consistent with the Bacteria_Genus_flattened.tsv mapping

## Limitations

- MG-RAST annotation servers require queue waits of days to weeks, making the annotation step slow
- Ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded
- Insufficient mRNA reads in the comparison dataset prevented perfect mRNA/rRNA correlation
- Shifts in microbial population sizes cannot be measured as accurately as with shotgun metagenomics

## Evidence

- [abstract] Each sample needs 40-50 million raw sequences, expected to yield 5-10 million annotated reads for accurate abundance measures: "Each sample needs 40-50 million raw sequences, which can be expected to yield the 5-10 million annotated reads necessary for accurate abundance measures."
- [abstract] mRNA-based abundance correlates with rRNA-based abundance: "we were able to demonstrate that overall organism transcriptional activity can be measured using mRNA counts"
- [discussion] SAMSA pipeline four steps: "Our pipeline fulfills the need for four main steps in metatranscriptomic data analysis: preprocessing, annotation, aggregation, and analysis."
- [methods] taxonomy_shifter.py purpose: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [abstract] remaining rRNA should be discarded: "ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded"
- [readme] SAMSA takes MG-RAST outputs for further analysis: "SAMSA, this pipeline, is designed to work in conjunction with MG-RAST as the annotation engine.  These scripts take the outputs from MG-RAST and help further analyze and break down the results."
