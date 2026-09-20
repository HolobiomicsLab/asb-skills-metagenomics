---
name: pearson-correlation-analysis
description: Use when organism abundance tables from two independent annotation sources
  (e.g., mRNA-based RefSeq hits vs. rRNA-based SILVA SSU hits) must be cross-validated
  by correlation at multiple taxonomic levels.
when_to_use_negative:
- Samples are ribodepleted — remaining rRNA sequences should be discarded, so no rRNA-based
  abundance table exists for comparison
- Input is raw FASTQ or per-read annotation output — tables must first be aggregated
  into organism summary files via SAMSA aggregation scripts
- The comparison involves functional (SEED Subsystems) categories rather than organism
  abundances
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: SAMSA
  role: Pipeline whose aggregation and analysis scripts (analysis_counter_w-md5s.py,
    RefSeq_output_reducer.py, taxonomy_shifter.py, R_organism_script.Rmd) build summary
    tables and compute the Pearson correlations
  repo: https://github.com/transcript/SAMSA
- name: MG-RAST
  role: Annotation server producing the organism match results against RefSeq (mRNA)
    and SILVA SSU (rRNA) that feed the summary tables
- name: R
  role: Statistical environment (R_organism_script.Rmd) used to compute Pearson correlations
    between mRNA- and rRNA-based abundance tables
- name: Python
  role: Runs the aggregation and taxonomy-shifting scripts (Python 2.7 dependency
    of SAMSA)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:144a21d7c1c6eb8a99965e957f67f025eaee988735fe240110ef5257851dac57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/SAMSA
  tools:
  - MG-RAST
  - Python
  - R
  - SAMSA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-correlation-analysis

## Summary

Validate organism abundance estimates derived from mRNA counts (RefSeq matches) against rRNA-derived counts (SILVA SSU matches) by computing Pearson correlations at order, family, and genus taxonomic levels across metatranscriptomic samples.

## When to use

Use when organism abundance tables from two independent annotation sources (e.g., mRNA-based RefSeq hits vs. rRNA-based SILVA SSU hits) must be cross-validated by correlation at multiple taxonomic levels.

## When NOT to use

- Samples are ribodepleted — remaining rRNA sequences should be discarded, so no rRNA-based abundance table exists for comparison
- Input is raw FASTQ or per-read annotation output — tables must first be aggregated into organism summary files via SAMSA aggregation scripts
- The comparison involves functional (SEED Subsystems) categories rather than organism abundances

## Inputs

- Non-ribodepleted fecal metatranscriptomic dataset (e.g., Tyk2 knockout vs. wild-type mouse DSS colitis samples from Hainzl et al.)
- MG-RAST annotation results files with RefSeq-derived mRNA organism matches
- MG-RAST annotation results files with SILVA SSU-derived rRNA organism matches
- Bacteria_Genus_flattened.tsv taxonomy reference database for taxonomy_shifter.py

## Outputs

- Bacterial-only organism summary tables at genus, family, and order levels for mRNA and rRNA sources
- Per-sample and averaged Pearson correlation values at each taxonomic level
- correlation_results.tsv with average correlations per level

## How to apply

Build organism summary tables separately with SAMSA's analysis_counter_w-md5s.py and RefSeq_output_reducer.py from RefSeq-derived mRNA matches and SILVA SSU-derived rRNA matches for each sample. Filter both tables to retain only bacterial annotations, discarding non-bacterial hits. Use taxonomy_shifter.py with the Bacteria_Genus_flattened.tsv reference database to convert genus-level summaries up to family and order levels. In R via R_organism_script.Rmd, compute Pearson correlations between the mRNA-based and rRNA-based abundance vectors at each taxonomic level across all samples (here 15 non-ribodepleted fecal samples), then report the average correlation per level. Rationale: agreement between mRNA and rRNA abundance estimates demonstrates that transcriptional activity can be measured from mRNA counts in non-ribodepleted public metatranscriptomes.

## Related tools

- **SAMSA** (Pipeline whose aggregation and analysis scripts (analysis_counter_w-md5s.py, RefSeq_output_reducer.py, taxonomy_shifter.py, R_organism_script.Rmd) build summary tables and compute the Pearson correlations) — https://github.com/transcript/SAMSA
- **MG-RAST** (Annotation server producing the organism match results against RefSeq (mRNA) and SILVA SSU (rRNA) that feed the summary tables)
- **R** (Statistical environment (R_organism_script.Rmd) used to compute Pearson correlations between mRNA- and rRNA-based abundance tables)
- **Python** (Runs the aggregation and taxonomy-shifting scripts (Python 2.7 dependency of SAMSA))

## Examples

```
python taxonomy_shifter.py -i genus_summary.tsv -d Bacteria_Genus_flattened.tsv -o family_summary.tsv
```

## Evaluation signals

- Average Pearson correlation of ~0.99 at order level, ~0.85 at family level, and ~0.81 at genus level across all 15 samples after bacterial-only filtering
- correlation_results.tsv is written and contains one average correlation per taxonomic level (order, family, genus)
- Both mRNA- and rRNA-derived summary tables contain only bacterial annotations after filtering
- Taxonomy_shifter.py output at family/order levels is consistent with the Bacteria_Genus_flattened.tsv reference database mapping
- Each sample contributes matched mRNA and rRNA abundance vectors with the same taxonomic identifiers for pairing

## Limitations

- Correlations were not perfect because there were insufficient mRNA reads in the data set used for direct mRNA/rRNA comparison
- Shifts in microbial population sizes cannot be measured as accurately with metatranscriptomics as with shotgun metagenomics
- MG-RAST is not tailored to work with eukaryotic organisms, and its annotation servers require queue waits of days to weeks

## Evidence

- [methods] R_organism_script.Rmd: "R_organism_script.Rmd - imports organism summary files and performs statistical significance correlations"
- [methods] taxonomy_shifter.py reference database: "taxonomy_shifter.py - converts organism summary files (initially at Genus level) to other taxonomic levels for analysis"
- [methods] Bacteria_Genus_flattened.tsv: "Bacteria_Genus_flattened.tsv - reference database for taxonomy_shifter.py"
- [abstract] mRNA counts validation: "we were able to demonstrate that overall organism transcriptional activity can be measured using mRNA counts"
- [discussion] rRNA correlation caveat: "Another reason that mRNA and rRNA results were not perfectly correlated is that there were insufficient mRNA reads in the data set used for direct comparison."
