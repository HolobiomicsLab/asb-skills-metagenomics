---
name: welch-t-test-statistical-comparison
description: Use when two independent groups of per-pair scores (e.g., APSS from SNP-only
  vs indel-only Bacmeta simulations) must be compared statistically, and per-point
  variances or sample sizes may be unequal.
when_to_use_negative:
- When the two score distributions are paired rather than independent (e.g., the same
  genome pair measured under both regimes) — a paired test would be required instead.
- When only the 40/60/80/100/200 subsampled APSS tables are used without verifying
  the all-regions APSS, since subsampling with seed=1 introduces run-to-run reproducibility
  assumptions.
- When the goal is effect-size or nonparametric inference on APSS distributions whose
  normality or variance structure has not been checked.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: SynTracker
  role: Produces the APSS and per-region synteny score tables that form the two distributions
    being compared with Welch's t-test
  repo: github.com/leylabmpi/SynTracker
- name: BLAST
  role: Underlying search (97% minimal identity, 70% minimal query coverage) generating
    the homologous region alignments from which synteny scores are computed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:27ea6f4cd903a26c8f989cecc546794c6896c18c2c205b97c5e3ef3e5ab28ec3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - SynTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# welch-t-test-statistical-comparison

## Summary

Apply Welch's t-test to compare Average Pairwise Synteny Score (APSS) distributions between experimental regimes — here, SNP-based vs indel-based in silico evolved populations — without assuming equal variances, to establish whether SynTracker synteny scores differ significantly by mutation mode.

## When to use

Use when two independent groups of per-pair scores (e.g., APSS from SNP-only vs indel-only Bacmeta simulations) must be compared statistically, and per-point variances or sample sizes may be unequal.

## When NOT to use

- When the two score distributions are paired rather than independent (e.g., the same genome pair measured under both regimes) — a paired test would be required instead.
- When only the 40/60/80/100/200 subsampled APSS tables are used without verifying the all-regions APSS, since subsampling with seed=1 introduces run-to-run reproducibility assumptions.
- When the goal is effect-size or nonparametric inference on APSS distributions whose normality or variance structure has not been checked.

## Inputs

- synteny_scores_per_region.csv (raw per-region pairwise synteny scores and BLAST identities per mutation regime and time point)
- avg_synteny_scores_all_regions.csv (APSS calculated using all available regions per pair, no subsampling)
- Group labels for each pairwise comparison (SNP-only vs indel-only regime, per time point)

## Outputs

- Per-time-point Welch's t-test statistic and two-sided P-value comparing SNP-based vs indel-based APSS distributions
- Reproduced minimal APSS values per regime/time point (expected: ~0.995/0.995/0.981 for SNP, ~0.103/0.103/0.206 for indel)
- Significance call for Fig. 2a,b (P < 2.2e-16)

## How to apply

Extract per-region pairwise synteny scores from synteny_scores_per_region.csv for each time point under each mutation regime (SNP-only 1e-6 events/nt/generation vs indel-only 1e-7), then compute the APSS over all available regions per pair using the avg_synteny_scores_all_regions.csv table (no subsampling, seed=1 default for reproducibility). Run Welch's t-test per time point comparing the SNP-based vs indel-based APSS distributions; Welch's variant is chosen because it does not assume equal variances between regimes. Verify that the minimal APSS values reproduce the reported pattern — SNP-based ~0.995/0.995/0.981 vs indel-based ~0.103/0.103/0.206 despite the tenfold higher SNP mutation frequency — and that the Welch t-test yields P < 2.2e-16 as reported for Fig. 2a,b.

## Related tools

- **SynTracker** (Produces the APSS and per-region synteny score tables that form the two distributions being compared with Welch's t-test) — github.com/leylabmpi/SynTracker
- **BLAST** (Underlying search (97% minimal identity, 70% minimal query coverage) generating the homologous region alignments from which synteny scores are computed)

## Examples

```
python -c "from scipy import stats; t, p = stats.ttest_ind(snp_apss, indel_apss, equal_var=False); print(t, p)"
```

## Evaluation signals

- Minimal APSS values match the reported SNP-based results (~0.995, ~0.995, ~0.981) and indel-based results (~0.103, ~0.103, ~0.206) across the three time points.
- Welch's t-test P-value is below 2.2e-16 for each time-point comparison, reproducing Fig. 2a,b significance.
- The number of pairwise comparisons per time point equals 190 (20 genomes per time point), matching the Bacmeta simulation design.
- APSS values are derived from avg_synteny_scores_all_regions.csv (all regions, no subsampling) rather than a subsampled table.
- SynTracker runs used default parameters (minimal BLAST identity 97, minimal coverage 70, region length 5000, seed=1) so scores are reproducible across runs.

## Limitations

- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size when subsampling APSS.
- APSS values may change between SynTracker runs if --no_seed is used, since the subsampling of n regions per pairwise is then unseeded.
- SynTracker is more sensitive to indels than SNPs (minimal APSS ~0.10-0.21 for indel-only regimes even with a tenfold lower mutation rate), so APSS differences reflect mutation mode, not only mutation load.

## Evidence

- [other] Compare the SNP-based vs indel-based APSS distributions per time point using Welch's t-test.: [evidence span withheld — anchor status: not_found]
- [other] Expected APSS pattern and significance: [evidence span withheld — anchor status: not_found]
- [readme] APSS table used for the comparison: "This table outputs the APSS (Average Pairwise Synteny Scores) calculated using all the available regions per each pair of samples without subsampling."
- [readme] Reproducibility of runs: "By default, a seed=1 is set to enable reproducibility between different runs."
