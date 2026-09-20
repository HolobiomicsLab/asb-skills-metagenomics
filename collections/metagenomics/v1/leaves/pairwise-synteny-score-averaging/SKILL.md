---
name: pairwise-synteny-score-averaging
description: Use when you already have raw per-region synteny scores (e.g. synteny_scores_per_region.csv)
  and need a robust pairwise relatedness matrix over sample pairs, controlling for
  unequal region counts per pair.
when_to_use_negative:
- 'You only need SNP-based strain differences: SynTracker''s synteny score has low
  sensitivity to SNPs, so use an SNP-based tool instead (combining both detects strains
  enriched in either point mutations or structural changes)'
- Pairs of samples have fewer than n available homologous regions — such pairs are
  excluded from downstream analysis and cannot receive an n-region APSS
- 'You need a universal same-strain cutoff: the APSS 0.95 cutoff was averaged across
  per-species cutoffs and may not be optimal for all settings; thresholds are subjective
  and species-dependent'
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Generates the raw per-region synteny score table and performs the random subsampling
    of n regions per pairwise comparison to compute the APSS, controlled by --no_seed
    (default seed=1) and the fixed n options (40, 60, 80, 100, 200)
  repo: https://github.com/leylabmpi/SynTracker
- name: DECIPHER R package
  role: 'Upstream: the FindSynteny function performs the all-versus-all pairwise alignments
    (maxGap and maxSep both 15) that yield the per-region synteny scores which are
    then averaged into APSS'
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:71b5c0c3f9155055dc19bf2e916e3b90fc34869f88e53d949f49d6651f199ab4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - DECIPHER R package
  - SynTracker
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pairwise-synteny-score-averaging

## Summary

Compute the Average Pairwise Synteny Score (APSS) by randomly subsampling n per-region pairwise synteny scores per pair of metagenomic samples or genomes and averaging them, with reproducible seeding and an all-regions variant.

## When to use

Use when you already have raw per-region synteny scores (e.g. synteny_scores_per_region.csv) and need a robust pairwise relatedness matrix over sample pairs, controlling for unequal region counts per pair.

## When NOT to use

- You only need SNP-based strain differences: SynTracker's synteny score has low sensitivity to SNPs, so use an SNP-based tool instead (combining both detects strains enriched in either point mutations or structural changes)
- Pairs of samples have fewer than n available homologous regions — such pairs are excluded from downstream analysis and cannot receive an n-region APSS
- You need a universal same-strain cutoff: the APSS 0.95 cutoff was averaged across per-species cutoffs and may not be optimal for all settings; thresholds are subjective and species-dependent

## Inputs

- Per-region raw synteny scores table (e.g. synteny_scores_per_region.csv) with one synteny score per pair of homologous genomic regions per pair of samples
- Choice of subsampling length n (default options 40, 60, 80, 100, 200 regions per pairwise comparison)
- Seed setting (default seed=1, or --no_seed to disable)

## Outputs

- APSS table per subsampling length: avg_synteny_scores_[n]_regions.csv (values in [0, 1])
- All-regions APSS table without subsampling: avg_synteny_scores_all_regions.csv
- Symmetric pairwise APSS matrix over all sample pairs per reference genome (under [genome_name]/final_output/) and combined summary tables (under summary_output/)

## How to apply

Load the raw per-region synteny score table produced by SynTracker (one score per pair of homologous regions per pair of samples). For each pair of samples or genomes, check the number of available regions and exclude pairs with fewer than n regions from downstream analysis. For each eligible pair, randomly subsample n regions (by default n = 40, 60, 80, 100, 200 per pair of samples) using the default seed=1 for reproducibility between runs, unless --no_seed is set, and average the individual pairwise synteny scores to obtain the APSS. Also produce an all-regions APSS table without subsampling (avg_synteny_scores_all_regions.csv) when maximum accuracy is preferred over a fixed region count. Rationale: the article reports maximal average sensitivity and specificity (98.6% and 97%) using 200 regions per pairwise comparison, while small numbers of regions mostly lower accuracy, so subsampling length is an explicit accuracy/sample-size trade-off. Note pairs with fewer than n regions are excluded, and the APSS cutoff of 0.95 is a subjective, species-dependent threshold for designating the same strain.

## Related tools

- **SynTracker** (Generates the raw per-region synteny score table and performs the random subsampling of n regions per pairwise comparison to compute the APSS, controlled by --no_seed (default seed=1) and the fixed n options (40, 60, 80, 100, 200)) — https://github.com/leylabmpi/SynTracker
- **DECIPHER R package** (Upstream: the FindSynteny function performs the all-versus-all pairwise alignments (maxGap and maxSep both 15) that yield the per-region synteny scores which are then averaged into APSS)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- All APSS values lie in the [0, 1] range (score of 1 corresponds to a single synteny block with 100% overlap)
- Output files exist for each default subsampling length: avg_synteny_scores_40/60/80/100/200_regions.csv, plus avg_synteny_scores_all_regions.csv
- Sample pairs with fewer than n regions are absent from the corresponding n-region APSS table
- Repeated runs with the default seed=1 produce identical APSS values; runs with --no_seed may differ between runs due to subsampling
- APSS values computed with more regions (e.g. 200) should match or improve the reported maximal sensitivity/specificity of 98.6%/97% relative to smaller n

## Limitations

- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and sample size
- By default a seed=1 is set for reproducibility; using --no_seed means average synteny scores may change between runs due to subsampling
- The APSS cutoff of 0.95 to designate the same strain was based on averaging different per-species cutoffs and may not be optimal for all settings; thresholds are subjective and species-dependent
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis

## Evidence

- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [readme] By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples.: "By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples."
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
