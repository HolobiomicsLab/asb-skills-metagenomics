---
name: synteny-score-threshold-sweep
description: Use when APSS tables from SynTracker runs must be turned into binary
  same/different-strain calls, when a cutoff must be calibrated for a new species
  or context, or when benchmarking SynTracker against SNP-based strain-comparison
  tools.
when_to_use_negative:
- When the goal is a continuous measure of genomic difference rather than binary same/different
  strain calls — the authors recommend avoiding such classifications because strain
  delineation is inherently subjective
- When the sample pairs being compared lack sufficient regions (fewer than n) to compute
  a subsampled APSS
- When an uncalibrated single cutoff (e.g. the human-gut APSS 0.95) is applied to
  a species with very different evolvability, since the cutoff was based on averaging
  per-species cutoffs and may not be optimal for all settings
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SynTracker
  role: Generates the APSS tables (subsampling 40/60/80/100/200 or all regions per
    pairwise comparison) whose scores are threshold-swept for ROC/AUC analysis
  repo: https://github.com/leylabmpi/SynTracker/releases
- name: inStrain
  role: Comparative tool whose popANI cutoff (99.999%) classifications are threshold-swept
    in parallel to compute competing ROC/AUC and strain-pair counts
- name: MIDAS
  role: Comparative SNP-based tool swept at its own ANI thresholds for the same benchmark
    ROC/AUC comparison
- name: StrainPhlAn
  role: Comparative marker-based tool swept at its own thresholds for the same benchmark
    ROC/AUC comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:fe08f5b358b27a828ac295850667f9ce9892215f45ba94ea7b01602cafc696c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MIDAS
  - StrainPhlAn
  - SynTracker
  - inStrain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# synteny-score-threshold-sweep

## Summary

Sweeps classification thresholds over SynTracker's average pairwise synteny scores (APSS) to convert continuous strain-relatedness values into same-strain/different-strain calls, building ROC curves and comparing AUC and detected strain pairs against cutoff-based tools like inStrain (popANI 99.999%), MIDAS and StrainPhlAn.

## When to use

Use when APSS tables from SynTracker runs must be turned into binary same/different-strain calls, when a cutoff must be calibrated for a new species or context, or when benchmarking SynTracker against SNP-based strain-comparison tools.

## When NOT to use

- When the goal is a continuous measure of genomic difference rather than binary same/different strain calls — the authors recommend avoiding such classifications because strain delineation is inherently subjective
- When the sample pairs being compared lack sufficient regions (fewer than n) to compute a subsampled APSS
- When an uncalibrated single cutoff (e.g. the human-gut APSS 0.95) is applied to a species with very different evolvability, since the cutoff was based on averaging per-species cutoffs and may not be optimal for all settings

## Inputs

- SynTracker APSS output tables (e.g. avg_synteny_scores_40_regions.csv, avg_synteny_scores_200_regions.csv, avg_synteny_scores_all_regions.csv) per reference genome and in summary_output/
- Raw per-region synteny score tables (synteny_scores_per_region.csv)
- Ground-truth labels of same/different strain for each sample pair (e.g. known twin-infant or donor relationships)
- Reference genomes (FASTA) and per-sample target metagenomic assemblies used to generate the APSS tables

## Outputs

- Sensitivity/specificity points swept across APSS thresholds, arranged as ROC curves (separately for bacterial and phage/plasmid strain sharing)
- AUC values per tool and per comparison type (e.g. SynTracker AUC 0.93 on the twin-infant benchmark)
- Counts of detected strain pairs at the chosen threshold (e.g. 112 for SynTracker vs 85/92/101 for StrainPhlAn/inStrain/MIDAS)
- A chosen same-strain APSS cutoff for the target context (e.g. 0.95 for the human gut)

## How to apply

Run SynTracker to obtain per-reference APSS tables (avg_synteny_scores_40_regions.csv, ..., _all_regions.csv) for bacterial and phage/plasmid references, then sweep candidate APSS cutoffs, labeling each sample pair as same-strain or different-strain at each threshold. At every threshold compute sensitivity and specificity against known strain-sharing truth, and plot the resulting ROC curve separately for bacterial and phage/plasmid comparisons; the article used 40 regions per pairwise comparison for the benchmark and found maximal average sensitivity/specificity of 98.6%/97% with 200 regions. Compute AUC from each ROC and count detected strain pairs at the optimal operating point; in the premature-infant-twin benchmark this yielded AUC 0.93 and 112 strain pairs (vs 85/92/101 for StrainPhlAn/inStrain/MIDAS). Choose the final cutoff by context: 0.95 was set to designate the same strain in the human gut, but thresholds are subjective and species-dependent because different taxa accumulate genomic differences at different rates. Exclude sample pairs with fewer than n regions from downstream analysis, since the APSS subsampling requires n regions per pair.

## Related tools

- **SynTracker** (Generates the APSS tables (subsampling 40/60/80/100/200 or all regions per pairwise comparison) whose scores are threshold-swept for ROC/AUC analysis) — https://github.com/leylabmpi/SynTracker/releases
- **inStrain** (Comparative tool whose popANI cutoff (99.999%) classifications are threshold-swept in parallel to compute competing ROC/AUC and strain-pair counts)
- **MIDAS** (Comparative SNP-based tool swept at its own ANI thresholds for the same benchmark ROC/AUC comparison)
- **StrainPhlAn** (Comparative marker-based tool swept at its own thresholds for the same benchmark ROC/AUC comparison)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- The ROC sweep over APSS thresholds on the premature-infant-twin benchmark with 40 regions per comparison should yield AUC ≈ 0.93, the highest among SynTracker/inStrain/MIDAS/StrainPhlAn
- At the benchmark operating point SynTracker should detect 112 strain pairs (vs 85 for StrainPhlAn, 92 for inStrain, 101 for MIDAS)
- With 200 regions per pairwise comparison, maximal average sensitivity and specificity should approach 98.6% and 97%
- Sample pairs with fewer than n regions must be excluded from the sweep, matching SynTracker's own filter
- On phages and plasmids, SynTracker's AUC should be significantly higher than inStrain's (Fig. 6a,b)

## Limitations

- Strain delineation thresholds are subjective and species-dependent; the APSS 0.95 cutoff was based on averaging per-species cutoffs and may not be optimal for all settings
- There is no single definition for microbial strains; definitions vary by context and researcher, so binary same/different calls should be interpreted cautiously
- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size
- Users must set their own cutoffs considering evolvability, selection, and their definition of 'strain', ideally using a training dataset relevant to their research question

## Evidence

- [discussion] APSS cutoff 0.95: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [discussion] Thresholds are species-dependent: "Such thresholds are subjective and depend on the species being considered, as different taxa accumulate genomic differences at different rates"
- [discussion] Sensitivity/specificity with 200 regions: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] Phage/plasmid AUC comparison: "SynTracker achieved a significantly higher AUC than inStrain when applied to phages and plasmids."
- [readme] APSS output tables define the sweep input: "The second type of output tables, `[genome name]_avg_synteny_scores_[subsampling length]_regions.csv`, gives the APSS"
- [discussion] Minimum-regions filter: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [discussion] Cutoff may not generalize: "this APSS cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings."
