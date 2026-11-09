---
name: roc-curve-performance-evaluation
description: Use when comparing binary classifiers of sample pairs (same-strain vs
  different-strain) that output continuous scores on a labeled benchmark, and you
  need threshold-free performance (ROC/AUC, sensitivity, specificity) across tools.
when_to_use_negative:
- The article's authors caution that classifying strains as same or different is inherently
  subjective and thresholds (e.g., inStrain popANI 99.999%, APSS 0.95) are species-dependent,
  so this benchmarking approach is wrong when a continuous genomic-difference measure
  is more appropriate than a binary classification
- The score table has fewer than n usable regions per sample pair — SynTracker excludes
  pairs with fewer than n regions from downstream analysis, so those pairs cannot
  contribute valid ROC points
- No ground-truth labels exist for sample-pair relatedness; ROC/AUC cannot be computed
  without a labeled benchmark
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: SynTracker
  role: Produces the APSS score matrix whose thresholds are swept to build the ROC
    curve; run in 'new' mode with default BLAST parameters (identity 97%, coverage
    70%, region length 5000, seed=1)
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Comparative strain-comparison tool whose popANI scores are threshold-swept
    to build a competing ROC curve
- name: MIDAS
  role: Comparative strain-comparison tool whose ANI scores are threshold-swept for
    the benchmark comparison
- name: StrainPhlAn
  role: Comparative strain-comparison tool whose scores are threshold-swept for the
    benchmark comparison
- name: BLAST
  role: Underlying search engine (NCBI BLAST+) used by SynTracker to find homologous
    regions feeding the APSS scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:d93ff6a04649eef9c9bb70a9f9e5188ac5179c79b211786e7d9687dc4de0274c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - MIDAS
  - StrainPhlAn
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# roc-curve-performance-evaluation

## Summary

Benchmark a pairwise strain-classification score (e.g., SynTracker's APSS) by sweeping classification thresholds to build ROC curves and compute AUC, enabling head-to-head comparison of strain-tracking tools (SynTracker vs MIDAS, StrainPhlAn, inStrain) on a labeled ground-truth dataset.

## When to use

Use when comparing binary classifiers of sample pairs (same-strain vs different-strain) that output continuous scores on a labeled benchmark, and you need threshold-free performance (ROC/AUC, sensitivity, specificity) across tools.

## When NOT to use

- The article's authors caution that classifying strains as same or different is inherently subjective and thresholds (e.g., inStrain popANI 99.999%, APSS 0.95) are species-dependent, so this benchmarking approach is wrong when a continuous genomic-difference measure is more appropriate than a binary classification
- The score table has fewer than n usable regions per sample pair — SynTracker excludes pairs with fewer than n regions from downstream analysis, so those pairs cannot contribute valid ROC points
- No ground-truth labels exist for sample-pair relatedness; ROC/AUC cannot be computed without a labeled benchmark

## Inputs

- APSS table from SynTracker (e.g., avg_synteny_scores_40_regions.csv) for bacterial references and for phage/plasmid references
- popANI scores from inStrain and ANI-based scores from MIDAS and StrainPhlAn on the same sample pairs
- Ground-truth labels of same-strain vs different-strain sample pairs (premature-infant-twin benchmark, 112 strain pairs)
- ROC sweep threshold grid

## Outputs

- ROC curves (sensitivity/specificity points) for each tool, computed separately for bacterial and phage/plasmid strain sharing
- AUC values per tool (SynTracker AUC = 0.93 with 40 regions per comparison)
- Counts of detected strain pairs per tool (SynTracker 112 vs StrainPhlAn/inStrain/MIDAS 85/92/101)
- Sensitivity/specificity values at selected operating thresholds (e.g., 98.6%/97% with 200 regions)

## How to apply

Extract the continuous pairwise score matrix from each tool — for SynTracker, the average pairwise synteny score (APSS) table such as avg_synteny_scores_40_regions.csv, for inStrain popANI, and ANI-based scores for MIDAS and StrainPhlAn. Sweep APSS/score thresholds across the range, classifying each sample pair as same-strain or different-strain at each cutoff, and record sensitivity (true-positive rate) and specificity (true-negative rate) at each point to construct the ROC curve. Compute the area under each ROC curve and count detected strain pairs at the chosen operating threshold (e.g., APSS 0.95 to designate the same strain in the human gut). Compare tools on both AUC and number of detected strain pairs; for SynTracker, use seed=1 (default) so subsampling of n regions per pairwise comparison is reproducible. Note the region-count trade-off: 40 regions gave AUC 0.93 with 112 detected strain pairs, while 200 regions per pairwise comparison yielded maximal average sensitivity and specificity of 98.6% and 97%.

## Related tools

- **SynTracker** (Produces the APSS score matrix whose thresholds are swept to build the ROC curve; run in 'new' mode with default BLAST parameters (identity 97%, coverage 70%, region length 5000, seed=1)) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Comparative strain-comparison tool whose popANI scores are threshold-swept to build a competing ROC curve)
- **MIDAS** (Comparative strain-comparison tool whose ANI scores are threshold-swept for the benchmark comparison)
- **StrainPhlAn** (Comparative strain-comparison tool whose scores are threshold-swept for the benchmark comparison)
- **BLAST** (Underlying search engine (NCBI BLAST+) used by SynTracker to find homologous regions feeding the APSS scores)

## Examples

```
python syntracker.py -target target_dir/ -ref ref_dir/ --identity 97 --coverage 70; # then sweep thresholds on avg_synteny_scores_40_regions.csv to build the ROC
```

## Evaluation signals

- SynTracker attains AUC = 0.93 with 40 regions per pairwise comparison on the premature-infant-twin benchmark, the highest among compared tools
- SynTracker detects 112 strain pairs, versus 85/92/101 for StrainPhlAn/inStrain/MIDAS
- SynTracker's AUC on phages and plasmids is significantly higher than inStrain's
- With 200 regions per pairwise comparison, sensitivity and specificity reach ~98.6% and 97%; smaller region counts mostly lower accuracy
- Running SynTracker with the default seed=1 reproduces the same subsampled APSS values across runs, so ROC points are stable

## Limitations

- Strain delineation thresholds (inStrain popANI 99.999%, APSS 0.95) are subjective and depend on the species, since different taxa accumulate genomic differences at different rates; the APSS 0.95 cutoff was an average of per-species cutoffs and may not be optimal for all settings
- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and sample size
- The benchmark's ground-truth classification is itself subjective — the authors recommend avoiding same/different strain classifications in favor of continuous genomic difference measures where possible

## Evidence

- [results] In the premature-infant-twin benchmark, SynTracker achieved the highest AUC value (0.93): [evidence span withheld — anchor status: not_found]
- [discussion] Sweep thresholds for ROC: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] APSS cutoff for same strain: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [readme] Reproducible subsampling: "By default, a seed=1 is set to enable reproducibility between different runs."
- [readme] APSS output tables: "The tables containing the APSS in different subsampling lengths are called `avg_synteny_scores_[subsampling length]_regions.csv`."
