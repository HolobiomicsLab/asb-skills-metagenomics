---
name: sample-pair-matrix-construction
description: Use when you have a raw per-region synteny score table (synteny_scores_per_region.csv)
  from SynTracker and need per-sample-pair APSS values at fixed subsampling lengths
  to build pairwise distance matrices or phylogenies.
when_to_use_negative:
- A sample pair has fewer than n regions available — such pairs are excluded from
  downstream analysis
- You need SNP-level or point-mutation resolution rather than structural/synteny differences
  — an SNP-based tool is more appropriate
- You require fully deterministic per-region contributions — with --no_seed the average
  synteny scores may change between SynTracker runs due to subsampling
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Computes per-region synteny scores via BLAST + FindSynteny and performs random
    subsampling of n regions per sample pair to produce APSS matrices at each default
    subsampling length
  repo: https://github.com/leylabmpi/SynTracker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:1ef9dbef18343530855e3ceac4db282171099b6e6da11163a6144f5b6c597da5
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
yes_rate: 0.7
n_claims: 10
---

# sample-pair-matrix-construction

## Summary

Assemble SynTracker's subsampled average pairwise synteny scores (APSS) into a symmetric pairwise matrix over all sample pairs for each subsampling length n (default n = 40, 60, 80, 100, 200), writing avg_synteny_scores_[n]_regions.tsv files that can be converted to synteny distances for UPGMA tree building.

## When to use

Use when you have a raw per-region synteny score table (synteny_scores_per_region.csv) from SynTracker and need per-sample-pair APSS values at fixed subsampling lengths to build pairwise distance matrices or phylogenies.

## When NOT to use

- A sample pair has fewer than n regions available — such pairs are excluded from downstream analysis
- You need SNP-level or point-mutation resolution rather than structural/synteny differences — an SNP-based tool is more appropriate
- You require fully deterministic per-region contributions — with --no_seed the average synteny scores may change between SynTracker runs due to subsampling

## Inputs

- Raw per-region synteny scores table (e.g. synteny_scores_per_region.csv) from SynTracker
- Subsampling length n (default n = 40, 60, 80, 100, 200)
- Random seed setting (default seed=1, or --no_seed)

## Outputs

- APSS matrix per subsampling length: avg_synteny_scores_[n]_regions.tsv with APSS values in [0, 1]
- APSS table using all available regions without subsampling: avg_synteny_scores_all_regions.csv
- Per-genome analogues under [genome_name]/final_output/ and combined tables under summary_output/

## How to apply

Load synteny_scores_per_region.csv, which contains one synteny score per pair of homologous genomic regions per pair of samples. For each sample or genome pair, check the number of available regions and exclude pairs with fewer than n regions from downstream analysis. For each eligible pair, randomly subsample n regions (default n = 40, 60, 80, 100, 200) using the default seed=1 for reproducibility unless --no_seed is set, and average the individual pairwise synteny scores to obtain the APSS. Assemble the APSS values into a symmetric pairwise matrix over all sample pairs for each n and write it as avg_synteny_scores_[n]_regions.tsv, ensuring all APSS values lie in [0, 1]; also emit avg_synteny_scores_all_regions.csv using all available regions without subsampling. Note that small numbers of regions per pairwise comparison mostly result in lower accuracy, so there is a trade-off between accuracy and sample size.

## Related tools

- **SynTracker** (Computes per-region synteny scores via BLAST + FindSynteny and performs random subsampling of n regions per sample pair to produce APSS matrices at each default subsampling length) — https://github.com/leylabmpi/SynTracker

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output matrix is symmetric over all eligible sample pairs, with pairs having fewer than n regions excluded
- All APSS values lie within [0, 1]
- One output table exists per default subsampling length (n = 40, 60, 80, 100, 200) plus an all-regions table
- Runs with the default seed=1 reproduce identical APSS values; runs with --no_seed may differ between runs
- Maximum sensitivity/specificity benchmarks: 98.6% and 97% when using 200 regions per pairwise comparison

## Limitations

- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size
- Pairs with fewer than n regions are excluded from downstream analysis
- With --no_seed the average synteny scores may change between SynTracker runs due to the random subsampling
- The APSS 0.95 same-strain cutoff is an average of per-species cutoffs and may not be optimal for all settings

## Evidence

- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [readme] By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples.: "By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples."
- [readme] --no_seed         Set no seed for the subsampling of n regions per pairwise (optional).: "--no_seed         Set no seed for the subsampling of n regions per pairwise (optional)."
- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
