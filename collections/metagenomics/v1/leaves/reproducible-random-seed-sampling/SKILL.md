---
name: reproducible-random-seed-sampling
description: Use when averaging per-region pairwise synteny scores into APSS matrices
  over all sample pairs and runs must be reproducible despite random subsampling of
  regions.
when_to_use_negative:
- A sample pair has fewer than n homologous regions detected — such pairs are excluded
  from downstream analysis rather than subsampled
- The goal is to compare sequences without homologous genomic regions or when SNP-based
  resolution of point mutations is the target (SynTracker is insensitive to SNPs;
  combine with an SNP-based tool instead)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Performs the random subsampling of n regions per pairwise comparison and averages
    individual pairwise synteny scores into APSS matrices; supports --no_seed to disable
    the default seed=1
  repo: https://github.com/leylabmpi/SynTracker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:ba9384efb5a87b392c56e57261464899a1948582c81b5e7e84d79768629a769a
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# reproducible-random-seed-sampling

## Summary

Randomly subsample n homologous genomic regions per pairwise comparison (default n = 40, 60, 80, 100, 200) and average the per-region synteny scores into an APSS, using a fixed seed=1 so repeated SynTracker runs are reproducible.

## When to use

Use when averaging per-region pairwise synteny scores into APSS matrices over all sample pairs and runs must be reproducible despite random subsampling of regions.

## When NOT to use

- A sample pair has fewer than n homologous regions detected — such pairs are excluded from downstream analysis rather than subsampled
- The goal is to compare sequences without homologous genomic regions or when SNP-based resolution of point mutations is the target (SynTracker is insensitive to SNPs; combine with an SNP-based tool instead)

## Inputs

- Raw per-region synteny scores table (synteny_scores_per_region.csv) with one synteny score per pair of homologous genomic regions per pair of metagenomic samples or genomes
- Chosen subsampling lengths n (default 40, 60, 80, 100, 200 regions per pairwise comparison)
- Seed setting (default seed=1) or --no_seed flag

## Outputs

- APSS matrix per subsampling length: avg_synteny_scores_[n]_regions.csv (per genome) and summary_output/avg_synteny_scores_[n]_regions.csv (all genomes combined)
- APSS matrix using all available regions without subsampling: avg_synteny_scores_all_regions.csv

## How to apply

Load the raw per-region synteny scores table (e.g. synteny_scores_per_region.csv) that SynTracker produces, one score per pair of homologous genomic regions per pair of samples. For each sample pair, exclude pairs with fewer than n regions from downstream analysis, since subsampling n regions is impossible otherwise. For each eligible pair, randomly subsample n regions (default n = 40, 60, 80, 100, 200) with a fixed seed=1 — only set --no_seed if run-to-run variability is acceptable — and average the individual pairwise synteny scores to obtain the APSS. Write the resulting symmetric pairwise APSS matrix for each subsampling length as avg_synteny_scores_[n]_regions.csv, and additionally compute avg_synteny_scores_all_regions.csv using all available regions without subsampling. Verify all APSS values lie in [0, 1]; an APSS cutoff of 0.95 is used in the paper to designate the same strain in the human gut.

## Related tools

- **SynTracker** (Performs the random subsampling of n regions per pairwise comparison and averages individual pairwise synteny scores into APSS matrices; supports --no_seed to disable the default seed=1) — https://github.com/leylabmpi/SynTracker

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output tables named avg_synteny_scores_[40|60|80|100|200]_regions.csv exist for each reference genome and in summary_output/
- All APSS values lie in [0, 1] (synteny distances are computed as |APSS − 1|)
- Re-running with default seed=1 yields identical APSS matrices between runs; identical inputs with --no_seed may differ between runs
- Pairs of samples with fewer than n available regions are absent from the subsampled APSS matrix but appear in avg_synteny_scores_all_regions.csv
- Comparisons using 200 regions per pairwise comparison achieve maximal average sensitivity and specificity of 98.6% and 97% in the paper's benchmark

## Limitations

- Subsampling with --no_seed makes average synteny scores change between SynTracker runs; keep seed=1 (the default) for reproducibility
- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size
- The APSS 0.95 same-strain cutoff was based on averaging per-species cutoffs and may not be optimal for all settings — researchers should set context-specific cutoffs

## Evidence

- [readme] Subsampling with fixed seed: "--no_seed         Set no seed for the subsampling of n regions per pairwise (optional)."
- [readme] Default seed for reproducibility: "By default, a seed=1 is set to enable reproducibility between different runs."
- [readme] Default subsampling lengths: "By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples."
- [discussion] APSS calculation definition: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [discussion] Exclusion of pairs with too few regions: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
