---
name: apss-aggregation-and-subsampling
description: Use when you have SynTracker's raw synteny_scores_per_region.csv output
  and need APSS values per pair of samples or genomes, with subsampled or all-region
  aggregation for comparisons or cutoff-based strain designation (e.g., APSS 0.95).
when_to_use_negative:
- When the goal is SNP-sensitive strain discrimination — the article shows SynTracker/APSS
  has low sensitivity to SNPs (minimal APSS ~0.981–0.995 for SNP-only simulations
  vs ~0.103–0.206 for indel-only), so combine it with an SNP-based tool like MIDAS
  or StrainPhlAn instead of relying on APSS alone.
- When sample pairs have fewer than n available homologous regions — such pairs are
  excluded from downstream analysis, so APSS with large N is inappropriate for sparse
  or highly divergent comparisons.
- When you need a universal same/different strain threshold — the APSS 0.95 cutoff
  is subjective, species-dependent, and may not be optimal for all settings.
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0102
tools:
- name: SynTracker
  role: Computes per-region synteny scores, randomly subsamples n regions per pairwise
    comparison, and outputs the APSS tables (avg_synteny_scores_all_regions.csv and
    per-N tables).
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: 'Upstream: SynTracker creates a BLAST database from the target assemblies
    and searches 1-kbp central regions of each reference with 97% minimal identity
    and 70% minimal query coverage, defining the regions whose scores feed the APSS.'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:0dcbaf394e5b15d83065c2c6c760cdb75d0ff6755caf6fa7b0fa770a3bc17b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# apss-aggregation-and-subsampling

## Summary

Aggregates SynTracker's per-region pairwise synteny scores into Average Pairwise Synteny Scores (APSS) via random subsampling of n regions per sample pair (default N = 40, 60, 80, 100, 200, or all regions), producing the avg_synteny_scores CSV outputs used for strain relatedness and benchmarking.

## When to use

Use when you have SynTracker's raw synteny_scores_per_region.csv output and need APSS values per pair of samples or genomes, with subsampled or all-region aggregation for comparisons or cutoff-based strain designation (e.g., APSS 0.95).

## When NOT to use

- When the goal is SNP-sensitive strain discrimination — the article shows SynTracker/APSS has low sensitivity to SNPs (minimal APSS ~0.981–0.995 for SNP-only simulations vs ~0.103–0.206 for indel-only), so combine it with an SNP-based tool like MIDAS or StrainPhlAn instead of relying on APSS alone.
- When sample pairs have fewer than n available homologous regions — such pairs are excluded from downstream analysis, so APSS with large N is inappropriate for sparse or highly divergent comparisons.
- When you need a universal same/different strain threshold — the APSS 0.95 cutoff is subjective, species-dependent, and may not be optimal for all settings.

## Inputs

- synteny_scores_per_region.csv (raw per-region pairwise synteny scores from a SynTracker run)
- SynTracker run configured with reference genome directory and target genome/metagenome assembly directory
- Choice of subsampling N (default 40, 60, 80, 100, 200 regions per pair) or all-regions mode

## Outputs

- avg_synteny_scores_40_regions.csv, _60, _80, _100, _200 (APSS per pair with subsampling)
- avg_synteny_scores_all_regions.csv (APSS per pair using all available regions, no subsampling)
- Per-genome tables under [genome_name]/final_output/ and combined tables under summary_output/

## How to apply

Run SynTracker in 'new' mode (python syntracker.py -target <target_dir> -ref <ref_dir>) with default parameters (region length 5000, minimal BLAST identity 97, minimal BLAST coverage 70), letting the script subsample n genomic regions per pair of metagenomic samples or genomes and average the individual pairwise synteny scores into APSS. By default SynTracker reports APSS for N = 40, 60, 80, 100 and 200 regions per pair and additionally computes APSS over all available regions in avg_synteny_scores_all_regions.csv. A seed=1 is applied by default to the subsampling so runs are reproducible; pass --no_seed only if stochastic variation across runs is acceptable. Pairs of samples or genomes with fewer than n available regions are excluded from downstream analysis, so inspect region counts per pair before interpreting APSS. The article's benchmarking found maximal average sensitivity and specificity (98.6% and 97%) using 200 regions per pairwise comparison, while small region counts mostly lower accuracy, so prefer the largest N that all pairs support. For strain designation in the human gut, the authors set an APSS cutoff of 0.95, but note this was an average across per-species cutoffs and should be tailored per context.

## Related tools

- **SynTracker** (Computes per-region synteny scores, randomly subsamples n regions per pairwise comparison, and outputs the APSS tables (avg_synteny_scores_all_regions.csv and per-N tables).) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Upstream: SynTracker creates a BLAST database from the target assemblies and searches 1-kbp central regions of each reference with 97% minimal identity and 70% minimal query coverage, defining the regions whose scores feed the APSS.)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output tables named avg_synteny_scores_[N]_regions.csv (N = 40, 60, 80, 100, 200) and avg_synteny_scores_all_regions.csv exist under summary_output/ and per-genome final_output/ directories.
- APSS values are bounded in [0, 1]; SNP-only simulated populations should give minimal APSS ~0.995/0.995/0.981 and indel-only ~0.103/0.103/0.206 (Welch t-test P < 2.2e-16) in the benchmark setting.
- Two runs with the default seed=1 yield identical APSS values; only runs with --no_seed may vary due to subsampling.
- Pairs reported in the APSS tables have at least n regions in synteny_scores_per_region.csv; pairs with fewer regions should be absent.
- Accuracy benchmarking expectation: 200 regions per comparison achieves maximal average sensitivity 98.6% and specificity 97%; much smaller N should show lower accuracy.

## Limitations

- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size.
- The APSS 0.95 same-strain cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings; thresholds are subjective and depend on the species being considered.
- By default a seed=1 is set for subsampling; without the seed (or with --no_seed) APSS may change between runs due to stochastic subsampling.
- SynTracker/APSS is insensitive to SNPs and sensitive to indels, so APSS should be interpreted as a structural-variation (synteny) signal, not a point-mutation measure.

## Evidence

- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [readme] The table containing the APSS using all regions (without subsampling) is called avg_synteny_scores_all_regions.csv.: "The table containing the APSS using all regions (without subsampling) is called `avg_synteny_scores_all_regions.csv`."
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
