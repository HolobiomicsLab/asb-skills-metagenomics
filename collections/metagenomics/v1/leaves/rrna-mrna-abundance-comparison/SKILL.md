---
name: rrna-mrna-abundance-comparison
description: Use when working with publicly available non-ribodepleted metatranscriptomic
  data (e.
when_to_use_negative:
- Samples are already ribodepleted — remaining rRNA reads are too sparse for a direct
  mRNA/rRNA correlation, and the article notes 'there were insufficient mRNA reads
  in the data set used for direct comparison' in such settings
- The goal is measuring shifts in microbial population sizes, which 'cannot be measured
  as accurately as using shotgun metagenomics' — use shotgun metagenomics instead
- Input data consists of short single reads (<100 bp) without joined paired-end reads,
  which SAMSA does not support for accurate abundance measures
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: SAMSA
  role: Pipeline whose aggregation scripts (analysis_counter_w-md5s.py, RefSeq_output_reducer.py)
    build organism summary files from RefSeq mRNA and SILVA SSU rRNA annotation results
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server producing the RefSeq organism matches (mRNA) and SILVA SSU
    matches (rRNA) used as abundance sources
- name: taxonomy_shifter.py
  role: Converts genus-level organism summary files to family and order levels using
    the Bacteria_Genus_flattened.tsv reference database
  repo: https://github.com/transcript/SAMSA
- name: R_organism_script.Rmd
  role: Imports organism summary files and computes Pearson correlations between mRNA-based
    and rRNA-based abundances per taxonomic level
  repo: https://github.com/transcript/SAMSA
- name: RefSeq_output_reducer.py
  role: Consolidates identical organism matches in summary files prior to correlation
    analysis
  repo: https://github.com/transcript/SAMSA
- name: Python 2.7
  role: Runs the SAMSA aggregation and taxonomy-shifting scripts
- name: R
  role: Runs R_organism_script.Rmd for statistical correlation analysis of abundance
    summaries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:c97b2aff28cf0109613cbead4319fb2a1a428cffb4b3306d4e4d152b319bfff3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - MG-RAST
  - Python 2.7
  - R
  - R_organism_script.Rmd
  - RefSeq_output_reducer.py
  - SAMSA
  - taxonomy_shifter.py
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# rrna-mrna-abundance-comparison

## Summary

Validates organism abundance estimates from mRNA counts (RefSeq matches) against rRNA-based estimates (SILVA SSU matches) in non-ribodepleted fecal metatranscriptomes using the SAMSA aggregation and analysis scripts, yielding Pearson correlations as a measure of transcriptional-activity measurement reliability.

## When to use

Use when working with publicly available non-ribodepleted metatranscriptomic data (e.g., fecal gut microbiome samples) and you need to validate mRNA-derived organism abundances against rRNA-derived abundances, or demonstrate that organism transcriptional activity can be measured from mRNA counts alone.

## When NOT to use

- Samples are already ribodepleted — remaining rRNA reads are too sparse for a direct mRNA/rRNA correlation, and the article notes 'there were insufficient mRNA reads in the data set used for direct comparison' in such settings
- The goal is measuring shifts in microbial population sizes, which 'cannot be measured as accurately as using shotgun metagenomics' — use shotgun metagenomics instead
- Input data consists of short single reads (<100 bp) without joined paired-end reads, which SAMSA does not support for accurate abundance measures

## Inputs

- Non-ribodepleted fecal metatranscriptomic samples (e.g., 15 Tyk2 knockout vs. wild-type mouse DSS colitis samples, Hainzl et al., ref 12)
- MG-RAST annotation results files with RefSeq-derived mRNA organism matches
- MG-RAST annotation results files with SILVA SSU-derived rRNA organism matches
- Bacteria_Genus_flattened.tsv reference database for taxonomy_shifter.py

## Outputs

- Bacterial-only mRNA-based organism summary tables at order, family, and genus levels
- Bacterial-only rRNA-based organism summary tables at order, family, and genus levels
- correlation_results.tsv with average Pearson correlations per taxonomic level (expected approx. 0.99 order, 0.85 family, 0.81 genus)

## How to apply

Retrieve all non-ribodepleted metatranscriptomic samples (e.g., the 15-sample Tyk2 knockout vs. wild-type mouse DSS colitis dataset from Hainzl et al.) and use SAMSA's analysis_counter_w-md5s.py plus RefSeq_output_reducer.py to build organism summary files separately for RefSeq-derived mRNA matches and SILVA SSU-derived rRNA matches. Filter both summary tables to retain only bacterial annotations, discarding non-bacterial hits. Apply taxonomy_shifter.py with the Bacteria_Genus_flattened.tsv reference database to convert genus-level summaries up to family and order levels. In R via R_organism_script.Rmd, compute Pearson correlations between mRNA-based and rRNA-based organism abundances at each taxonomic level across all samples, and report the average correlation per level (writing results to correlation_results.tsv). The rationale is that if ribosomal RNA depletion does not equally deplete ribosomes from all species, remaining rRNA counts can serve as an independent abundance proxy to validate mRNA-based measures of organism transcriptional activity.

## Related tools

- **SAMSA** (Pipeline whose aggregation scripts (analysis_counter_w-md5s.py, RefSeq_output_reducer.py) build organism summary files from RefSeq mRNA and SILVA SSU rRNA annotation results) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server producing the RefSeq organism matches (mRNA) and SILVA SSU matches (rRNA) used as abundance sources)
- **taxonomy_shifter.py** (Converts genus-level organism summary files to family and order levels using the Bacteria_Genus_flattened.tsv reference database) — https://github.com/transcript/SAMSA
- **R_organism_script.Rmd** (Imports organism summary files and computes Pearson correlations between mRNA-based and rRNA-based abundances per taxonomic level) — https://github.com/transcript/SAMSA
- **RefSeq_output_reducer.py** (Consolidates identical organism matches in summary files prior to correlation analysis) — https://github.com/transcript/SAMSA
- **Python 2.7** (Runs the SAMSA aggregation and taxonomy-shifting scripts)
- **R** (Runs R_organism_script.Rmd for statistical correlation analysis of abundance summaries)

## Examples

```
python analysis_counter_w-md5s.py sample_mrna_annotations.txt > mrna_summary.txt && python RefSeq_output_reducer.py mrna_summary.txt > mrna_reduced.txt && python taxonomy_shifter.py mrna_reduced.txt Bacteria_Genus_flattened.tsv > mrna_family.txt
```

## Evaluation signals

- Average Pearson correlation between mRNA-based and rRNA-based abundances across all 15 samples should approximate 0.99 at order level, 0.85 at family level, and 0.81 at genus level after bacterial-only filtering
- Both mRNA- and rRNA-based summary tables contain only bacterial annotations after filtering (non-bacterial hits discarded)
- Taxonomy-shifter outputs at family and order levels are consistent with the Bacteria_Genus_flattened.tsv reference database mappings
- correlation_results.tsv is written with one average correlation value per taxonomic level (order, family, genus)
- Each sample's summary files were built from both RefSeq mRNA matches and SILVA SSU rRNA matches using analysis_counter_w-md5s.py and RefSeq_output_reducer.py

## Limitations

- mRNA and rRNA results were not perfectly correlated because 'there were insufficient mRNA reads in the data set used for direct comparison'
- Ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should otherwise be discarded — this skill instead repurposes them, so it only applies to non-ribodepleted data
- MG-RAST is not tailored to work with eukaryotic organisms, limiting annotation coverage to bacterial-dominated comparisons; the workflow therefore filters to bacterial annotations
- MG-RAST annotation servers require queue waits of days to weeks, which can slow the annotation step considerably

## Evidence

- [results] mRNA-based organism abundance estimates consistently correlated with rRNA-based estimates: "we were able to demonstrate that overall organism transcriptional activity can be measured using mRNA counts"
- [methods] taxonomy_shifter.py converts genus-level summaries up to family and order: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [methods] R_organism_script.Rmd performs the statistical correlations on organism summaries: "R_organism_script.Rmd - imports organism summary files and performs statistical significance correlations"
- [abstract] Remaining rRNA reads should normally be discarded due to unequal depletion: "ribosomal RNA depletion does not equally deplete ribosomes from all species within a sample, and remaining rRNA sequences should be discarded"
- [readme] RefSeq_output_reducer.py consolidates identical organism matches before correlation: "RefSeq_output_reducer.py - consolidates identical organism matches in summary files"
