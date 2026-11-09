---
name: isolate-genome-pairwise-scores-extraction
description: Use when isolate or assembly genome pairs have been compared with SynTracker
  and inStrain and you need per-pair APSS and popANI values to build same/different-strain
  classification tables.
when_to_use_negative:
- When samples are sequencing reads rather than assemblies/genomes — SynTracker compares
  metagenomic assemblies or genomes, and inStrain popANI requires read-mapping comparisons,
  not extracted APSS tables
- When no SNP-based comparator is available and only structural-difference information
  is needed — the combined APSS/popANI classification requires both score types
- When you expect the universal APSS 0.95 human-gut cutoff to apply to your species
  without validation — the paper explicitly states thresholds are species-dependent
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Computes per-pair APSS (Average Pairwise Synteny Scores) between isolate genomes
    using genome synteny, output in avg_synteny_scores_all_regions.csv tables
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Computes SNP-based popANI values for every isolate pair used as the complementary
    classification signal
- name: BLAST
  role: NCBI BLAST+ dependency; SynTracker builds a BLAST database from the target
    genomes and searches central regions with 97% minimal identity and 70% minimal
    coverage
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:944bd903e81fd138db6dc51930f4c9dce5ed207b353537e0930af377f6070898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# isolate-genome-pairwise-scores-extraction

## Summary

Extract per-pair Average Pairwise Synteny Scores (APSS) from SynTracker output tables and combine them with SNP-based popANI values from inStrain to classify isolate genome pairs as same or different strain, e.g. using APSS > 0.955 and popANI > 0.99999.

## When to use

Use when isolate or assembly genome pairs have been compared with SynTracker and inStrain and you need per-pair APSS and popANI values to build same/different-strain classification tables.

## When NOT to use

- When samples are sequencing reads rather than assemblies/genomes — SynTracker compares metagenomic assemblies or genomes, and inStrain popANI requires read-mapping comparisons, not extracted APSS tables
- When no SNP-based comparator is available and only structural-difference information is needed — the combined APSS/popANI classification requires both score types
- When you expect the universal APSS 0.95 human-gut cutoff to apply to your species without validation — the paper explicitly states thresholds are species-dependent

## Inputs

- SynTracker output tables '[genome_name]_avg_synteny_scores_all_regions.csv' and subsampled 'avg_synteny_scores_[N]_regions.csv' (N = 40, 60, 80, 100, 200)
- inStrain popANI values computed on every isolate pair from four SRA collections (Supplementary Tables 2-5)
- Isolate genome FASTA files organized as one fasta file per genome in a target directory
- Per-species reference genomes (GCF_900087635.2, NC_000913.3, CP032479.1, GCF_000331185.2)

## Outputs

- Four per-species same/different-strain classification tables corresponding to Fig. 3a-d (66 N. gonorrhoeae rows, 185 hypermutator E. coli rows, H. pylori within-host pairs across six individuals, S. rimosus library pairs)
- Per-pair APSS values extracted from the avg_synteny_scores_all_regions tables
- Per-pair popANI values from inStrain comparisons

## How to apply

Run SynTracker in 'new' mode from the SynTracker_1_4 conda environment (created from SynTracker_env.yml, requires NCBI BLAST+) on each species' target genomes with its per-species reference (GCF_900087635.2 for N. gonorrhoeae, NC_000913.3 for E. coli, CP032479.1 for H. pylori, GCF_000331185.2 for S. rimosus), keeping default parameters (region length 5000, minimal BLAST identity 97, minimal coverage 70, seed=1 for reproducible subsampling). Run inStrain on every isolate pair from the SRA collections to compute popANI. Extract the per-pair APSS values from the SynTracker 'avg_synteny_scores_all_regions' tables, using up to 200 regions per pairwise comparison, which yields maximal average sensitivity (98.6%) and specificity (97%). Apply the chosen same-strain cutoffs — here APSS > 0.955 and popANI > 0.99999 — to produce one classification table per species (66 N. gonorrhoeae comparisons, 185 hypermutator E. coli pairs, H. pylori within-host pairs across six individuals, S. rimosus library pairs). Cutoffs are subjective and species-dependent, so they should be set per research question rather than taken as universal.

## Related tools

- **SynTracker** (Computes per-pair APSS (Average Pairwise Synteny Scores) between isolate genomes using genome synteny, output in avg_synteny_scores_all_regions.csv tables) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Computes SNP-based popANI values for every isolate pair used as the complementary classification signal)
- **BLAST** (NCBI BLAST+ dependency; SynTracker builds a BLAST database from the target genomes and searches central regions with 97% minimal identity and 70% minimal coverage)

## Examples

```
python syntracker.py -target isolate_genomes_dir -ref reference_dir -out SynTracker_output/
```

## Evaluation signals

- Each per-species output CSV exists and contains all pairwise comparisons for its species (e.g., 66 rows for N. gonorrhoeae, 185 for hypermutator E. coli)
- The N. gonorrhoeae Spearman correlation between APSS and popANI over the 66 comparisons equals 0.985
- All 185 hypermutator E. coli pairs are called same strain by both tools (APSS > 0.955 and popANI > 0.99999)
- S. rimosus pairs show a wide APSS range despite clonal popANI values, reproducing the structural-vs-point-mutation signature
- H. pylori within-host subpopulations are split between the two tools, matching the reported per-species divergence

## Limitations

- Strain delineation thresholds (inStrain popANI 99.999%, APSS 0.95) are subjective and depend on the species being considered; the 0.95 cutoff was based on averaging per-species cutoffs and may not be optimal for all settings
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis
- Because classifying strains as same or different is inherently subjective, the authors recommend considering continuous genomic difference measures instead of binary classifications

## Evidence

- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [readme] python syntracker.py -target ... -ref ... -out SynTracker_output/: "python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/"
- [readme] README description of avg_synteny_scores_all_regions output: "The table containing the APSS using all regions (without subsampling) is called `avg_synteny_scores_all_regions.csv`."
- [discussion] Such thresholds are subjective and depend on the species being considered: "Such thresholds are subjective and depend on the species being considered, as different taxa accumulate genomic differences at different rates"
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
