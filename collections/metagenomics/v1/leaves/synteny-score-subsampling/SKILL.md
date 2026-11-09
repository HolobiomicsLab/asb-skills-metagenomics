---
name: synteny-score-subsampling
description: Use when you have SynTracker's raw per-region synteny score table and
  need APSS matrices per subsampling length, or must decide how many regions (n) a
  pairwise comparison needs before it is usable.
when_to_use_negative:
- A sample pair has fewer than n available regions for the chosen n — those pairs
  are excluded from downstream analysis, so a smaller n or the all-regions table is
  needed
- You need point-mutation (SNP) resolution rather than structural-change sensitivity
  — SynTracker's synteny-based APSS is deliberately low sensitivity to SNPs, so an
  SNP-based tool like inStrain or MIDAS fits better
- You require deterministic scores without a fixed seed and have not decided on reproducibility
  vs. resampling variance — without seed=1 APSS values will change between SynTracker
  runs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Performs the random subsampling of n regions per pairwise comparison, averages
    the individual pairwise synteny scores into the APSS, and writes the avg_synteny_scores_[n]_regions
    tables; --no_seed controls the seed=1 default
  repo: https://github.com/leylabmpi/SynTracker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:d93d0239d353a92f86f91f58689e90980bfcedea1a35854b3f8aae9ac516a8e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - SynTracker
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# synteny-score-subsampling

## Summary

Randomly subsample n per-region pairwise synteny scores (default n = 40, 60, 80, 100, 200) for each pair of metagenomic samples or genomes and average them into an Average Pairwise Synteny Score (APSS), balancing accuracy against the number of regions a pair shares.

## When to use

Use when you have SynTracker's raw per-region synteny score table and need APSS matrices per subsampling length, or must decide how many regions (n) a pairwise comparison needs before it is usable.

## When NOT to use

- A sample pair has fewer than n available regions for the chosen n — those pairs are excluded from downstream analysis, so a smaller n or the all-regions table is needed
- You need point-mutation (SNP) resolution rather than structural-change sensitivity — SynTracker's synteny-based APSS is deliberately low sensitivity to SNPs, so an SNP-based tool like inStrain or MIDAS fits better
- You require deterministic scores without a fixed seed and have not decided on reproducibility vs. resampling variance — without seed=1 APSS values will change between SynTracker runs

## Inputs

- Raw per-region synteny scores table (e.g. synteny_scores_per_region.csv) with one score per homologous region pair per sample pair
- Number of regions n per pairwise comparison (default set: 40, 60, 80, 100, 200)
- Subsampling seed setting (default seed=1, or --no_seed)

## Outputs

- APSS matrix per subsampling length: avg_synteny_scores_[n]_regions.tsv/csv (e.g. n = 40, 60, 80, 100, 200)
- APSS matrix over all regions without subsampling: avg_synteny_scores_all_regions.csv
- Per-sample-pair APSS values in [0, 1], with pairs having fewer than n regions excluded

## How to apply

Load the raw per-region synteny scores table (e.g. synteny_scores_per_region.csv) that SynTracker produces, which contains one synteny score per pair of homologous genomic regions per pair of samples. For each pair of samples or genomes, check how many regions are available and exclude pairs with fewer than n regions from downstream analysis. For each eligible pair, randomly subsample n regions per pairwise comparison (defaults n = 40, 60, 80, 100, 200) and average the individual pairwise synteny scores into the APSS; use the default seed=1 so runs are reproducible unless --no_seed is passed. Write one symmetric APSS matrix per n as avg_synteny_scores_[n]_regions.tsv/csv, plus avg_synteny_scores_all_regions.csv when averaging over all available regions without subsampling; all APSS values should lie in [0, 1]. Note the trade-off: small numbers of regions per pairwise comparison mostly result in lower accuracy, and in the paper's benchmark maximal average sensitivity and specificity (98.6% and 97%) were reached with 200 regions per pairwise comparison.

## Related tools

- **SynTracker** (Performs the random subsampling of n regions per pairwise comparison, averages the individual pairwise synteny scores into the APSS, and writes the avg_synteny_scores_[n]_regions tables; --no_seed controls the seed=1 default) — https://github.com/leylabmpi/SynTracker

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output tables named avg_synteny_scores_[subsampling length]_regions.csv exist for each default n (40, 60, 80, 100, 200) under [genome_name]/final_output/ and summary_output/
- All APSS values lie within [0, 1]
- Sample pairs with fewer than n regions are absent from the n-region APSS matrices (excluded from downstream analysis)
- Re-running with the default seed=1 reproduces identical APSS values; setting --no_seed makes averages vary between runs
- The all-regions table (avg_synteny_scores_all_regions.csv) is also present, computed without subsampling

## Limitations

- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis, so low-coverage pairs cannot be compared at a given n
- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size
- Without --no_seed, a seed=1 is set for reproducibility; if no seed is used, average synteny scores may change between SynTracker runs due to subsampling

## Evidence

- [readme] By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples.: "gives the APSS (Average Pairwise Synteny Score) that was calculated by subsampling N regions per pair of samples"
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "This means that the average synteny scores may change between SynTracker runs due to the subsampling. By default, a seed=1 is set"
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
