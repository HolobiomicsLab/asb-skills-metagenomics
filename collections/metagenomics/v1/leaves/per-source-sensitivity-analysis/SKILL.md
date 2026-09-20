---
name: per-source-sensitivity-analysis
description: Use when benchmarking a metagenomic fecal source tracking tool against
  mesocosms of known composition and you need per-source accuracy, e.g. to detect
  under-performing sources like cow.
when_to_use_negative:
- Input is a single environmental metagenome without known ground-truth source composition
  — sensitivity cannot be computed without spiked-in reference truth
- You need overall (pooled) tool performance rather than source-resolved breakdown
  — the aggregate sensitivity/specificity near 0.90 is a separate metric
- The attribution tool does not report per-source estimated cell fractions, since
  the 0.01% LOD call cannot be applied
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_0622
tools:
- name: SourceApp
  role: Produces the per-source attribution result tables and estimated cell fractions
    that are scored against ground truth
  repo: https://github.com/bglindner/SourceApp
- name: Python
  role: Used to load the attribution .csv outputs and compute per-source sensitivity/specificity
    statistics
- name: CoverM
  role: Read-mapping coverage calculator underlying genome-to-metagenome mapping used
    in the SourceApp framework
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:0193cca38b2dbfe0bedd9a15c8708503daeafa2ddee2f4392bffa0244d2ff2f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bglindner/SourceApp
  tools:
  - CoverM
  - Python
  - SourceApp
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# per-source-sensitivity-analysis

## Summary

A benchmarking skill that computes per-source sensitivity and specificity for fecal source attribution (cat, chicken, cow, dog, goat, pig, sewage, septage) by comparing SourceApp attribution calls against ground-truth spike-in cell fractions from mesocosm metagenomes.

## When to use

Use when benchmarking a metagenomic fecal source tracking tool against mesocosms of known composition and you need per-source accuracy, e.g. to detect under-performing sources like cow.

## When NOT to use

- Input is a single environmental metagenome without known ground-truth source composition — sensitivity cannot be computed without spiked-in reference truth
- You need overall (pooled) tool performance rather than source-resolved breakdown — the aggregate sensitivity/specificity near 0.90 is a separate metric
- The attribution tool does not report per-source estimated cell fractions, since the 0.01% LOD call cannot be applied

## Inputs

- SourceApp attribution result tables (.csv) for all mesocosm metagenomes
- Ground-truth cellular fraction values from Table S1 of the Supporting Information
- List of true spike-in source categories per mesocosm (from the mesocosm design, e.g. PRJNA1161616)

## Outputs

- source_attribution_accuracy.csv with one row per fecal source category, containing sensitivity and specificity values in [0, 1]
- Per-source false-negative/false-positive counts identifying under-performing sources

## How to apply

Load the SourceApp attribution result tables (.csv) for all mesocosm metagenomes and the ground-truth cellular fraction values (Table S1). Apply the paper's attribution logic: score a source as 'present' when at least one source-specific genome is detected in the metagenome AND its estimated cell fraction exceeds the limit of detection of 0.01% relative abundance. For each of the 8 fecal source categories (cat, chicken, cow, dog, goat, pig, sewage, septage), compute per-source sensitivity as the fraction of mesocosms in which the source was truly spiked in and correctly attributed, and per-source specificity from false-positive attributions. Write one row per source to source_attribution_accuracy.csv. The rationale is that overall sensitivity/specificity (~0.90) can mask source-specific failure modes: all mesocosms containing cow spike-in signal were false negatives, attributed to under-sampling of the cow fecal microbiome (goat database: 759 species vs 43 species in the cow database).

## Related tools

- **SourceApp** (Produces the per-source attribution result tables and estimated cell fractions that are scored against ground truth) — https://github.com/bglindner/SourceApp
- **Python** (Used to load the attribution .csv outputs and compute per-source sensitivity/specificity statistics)
- **CoverM** (Read-mapping coverage calculator underlying genome-to-metagenome mapping used in the SourceApp framework) — https://github.com/wwood/CoverM

## Examples

```
python compute_source_sensitivity.py --attribution sourceapp_results/*.csv --ground-truth table_s1.csv --lod 0.01 --output source_attribution_accuracy.csv
```

## Evaluation signals

- The output table contains exactly 8 source rows (cat, chicken, cow, dog, goat, pig, sewage, septage)
- All sensitivity and specificity values fall within [0, 1]
- Cow shows the worst per-source performance (highest false-negative rate), with all cow-spiked mesocosms reported as false negatives
- Recomputing sensitivity from the confusion counts (true positives / (true positives + false negatives)) reproduces the table values
- Sources flagged 'present' in the ground truth but absent from the attribution tables are counted as false negatives, not dropped

## Limitations

- All mesocosms containing cow spike-in signal were reported as false negatives, reflecting under sampling of the cow fecal microbiome (goat database: 759 species vs 43 species in the cow database)
- Sensitivity depends on database completeness per source; SourceApp works best when users provide supplementary reference genomes to bolster the default database
- Performance results are specific to the optimal parameters (93% identity, 70% overlap, 0% masking, LOD 0.01%) and may differ under other settings

## Evidence

- [abstract] SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes.: "SourceApp’s performance varied substantially by source, with cows being underestimated due to under sampling of cow fecal microbiomes."
- [methods] cow LOD attribution logic: "the estimated cell fraction for a source must exceed the parameter set for limit of detection (0.01% by default)"
- [results] cow false negatives: "false negatives for cows across all samples point to a poor understanding of the prokaryotic content of cow feces at the genomic level"
- [results] optimal parameters: "93% minimum read-to-genome alignment identity, 70% minimum alignment-to-read overlap, 0% target genome masking, and a limit of detection (LOD) of 0.01% relative abundance"
- [readme] README supplementary genomes: "SourceApp works best when the user is able to provide supplementary reference genomes to bolster SourceApp's default genomic reference database."
