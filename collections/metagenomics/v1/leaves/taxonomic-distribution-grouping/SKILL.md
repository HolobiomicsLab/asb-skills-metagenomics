---
name: taxonomic-distribution-grouping
description: Use when you have per-feature taxonomy assignments (e.g. QIIME2 Naive
  Bayes or BLAST predictions on 16S mock-community data) and need abundance profiles
  grouped at a chosen taxonomic level for composition comparison.
when_to_use_negative:
- Input is already a grouped taxonomic composition table at the target level (re-grouping
  is redundant)
- You need strain- or ASV-level resolution rather than aggregated rank-level compositions
- The data are not 16S amplicon taxonomic assignments (e.g. metagenomic functional
  profiles), since the MiCoNE Group step is defined for taxonomy levels
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MiCoNE
  role: Provides the Group transform that collapses abundances to the requested tax_levels
  repo: https://github.com/segrelab/MiCoNE
- name: QIIME2
  role: Runs Naive BayES taxonomy assignment (gg_13_8_99_515_806 classifier, confidence
    0.7) whose outputs are grouped
- name: BLAST
  role: Alternative assigner against ncbi_refseq (max_accepts 10, perc_identity 0.8,
    evalue 0.001, min_consensus 0.51) whose outputs are grouped
- name: Python
  role: Loads distributions and computes Bray-Curtis dissimilarity per grouped level
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:67ca1762088cac6e7a16f5330ae75e9b0361e2b271530d6b83faf5c05b722744
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - BLAST
  - MiCoNE
  - Python
  - QIIME2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# taxonomic-distribution-grouping

## Summary

Aggregates predicted or observed taxonomic abundance assignments into higher-level distributions (Phylum, Class, Order, Family, Genus, Species) using the MiCoNE Group transform, so that composition accuracy (e.g. Bray-Curtis dissimilarity against mock-community expectations) can be assessed at each taxonomic level.

## When to use

Use when you have per-feature taxonomy assignments (e.g. QIIME2 Naive Bayes or BLAST predictions on 16S mock-community data) and need abundance profiles grouped at a chosen taxonomic level for composition comparison.

## When NOT to use

- Input is already a grouped taxonomic composition table at the target level (re-grouping is redundant)
- You need strain- or ASV-level resolution rather than aggregated rank-level compositions
- The data are not 16S amplicon taxonomic assignments (e.g. metagenomic functional profiles), since the MiCoNE Group step is defined for taxonomy levels

## Inputs

- Predicted taxonomic abundance assignments from QIIME2 Naive Bayes (gg_13_8_99_515_806 classifier, confidence 0.7) or BLAST (ncbi_refseq, perc_identity 0.8)
- Expected taxonomic composition tables of the MiCoNE mock datasets (Zenodo record 7051556, data.zip)
- MiCoNE pipeline configuration with the Group step's tax_levels option

## Outputs

- Grouped abundance profiles at Phylum, Class, Order, Family, Genus, and Species levels
- Bray-Curtis dissimilarity values per database and taxonomic level (bray_curtis_dissimilarity.csv)

## How to apply

Run taxonomy assignment first (e.g. Naive Bayes classifier 'gg_13_8_99_515_806' with confidence 0.7 in QIIME2, or BLAST against 'ncbi_refseq' with max_accepts=10, perc_identity=0.8, evalue=0.001, min_consensus=0.51), then apply the MiCoNE Group transform with tax_levels set to ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] to collapse predicted abundances to the desired level. For each level, compute the Bray-Curtis dissimilarity between the grouped predicted distribution and the expected mock-community composition, writing all values (e.g. to bray_curtis_dissimilarity.csv). The rationale is that database accuracy varies by taxonomic rank, so grouping exposes rank-specific residual dissimilarity rather than a single pooled comparison.

## Related tools

- **MiCoNE** (Provides the Group transform that collapses abundances to the requested tax_levels) — https://github.com/segrelab/MiCoNE
- **QIIME2** (Runs Naive BayES taxonomy assignment (gg_13_8_99_515_806 classifier, confidence 0.7) whose outputs are grouped)
- **BLAST** (Alternative assigner against ncbi_refseq (max_accepts 10, perc_identity 0.8, evalue 0.001, min_consensus 0.51) whose outputs are grouped)
- **Python** (Loads distributions and computes Bray-Curtis dissimilarity per grouped level)

## Examples

```
micone init -w full -o mock_community_pipeline && bash run.sh
```

## Evaluation signals

- bray_curtis_dissimilarity.csv exists and its dissimilarity column contains values in [0, 1]
- Dissimilarity values exceed 0.5 at Family and Genus levels and 0.9 at Species level for all three databases
- No single taxonomy database (Greengenes, SILVA, NCBI RefSeq) is strictly best across all levels, matching the Fig. 3C pattern
- The grouped profiles' taxa match the tax_levels list ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'] supported by the MiCoNE Group step
- If values fall outside the expected pattern, the output is flagged for expert review

## Limitations

- Dissimilarity is high for all databases at deeper ranks, so grouping alone cannot discriminate which database is best in absolute terms
- Species-level grouping inherits taxonomy-assignment failures (e.g. low-confidence or non-consensus BLAST hits), inflating dissimilarity
- The MiCoNE data directory must be manually downloaded from Zenodo (record 7051556); missing data blocks the pipeline

## Evidence

- [methods] Group transform exposes rank-specific tax_levels for grouping: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [readme] MiCoNE data must be manually downloaded from Zenodo before pipeline use: "The data directory (`nf_micone/data`) needs to be manually downloaded using this [link](https://zenodo.org/record/7051556/files/data.zip?download=1)."
- [intro] Different tools lead to different composition inferences, motivating standardized comparison: "The different methods and tools can lead to vastly different inferences of community compositions and co-occurrence networks"
- [readme] MiCoNE is a Python package for exploring tool choices in 16S processing: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."
