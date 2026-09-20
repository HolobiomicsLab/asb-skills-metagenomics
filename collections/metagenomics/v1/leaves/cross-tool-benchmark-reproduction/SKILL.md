---
name: cross-tool-benchmark-reproduction
description: Use when a study requires reproducing a published tool-benchmark (same
  input dataset, multiple strain-comparison tools) with ROC/AUC and detection-count
  metrics, e.g. validating SynTracker's reported AUC 0.93 and 112 strain pairs.
when_to_use_negative:
- The goal is continuous genomic-difference measurement rather than same/different-strain
  classification — the article warns that strain delineation thresholds (e.g. popANI
  99.999%, APSS 0.95) are subjective and species-dependent
- Only one tool is being run and no comparative baseline (inStrain/MIDAS/StrainPhlAn)
  is needed, so an AUC/detection-count benchmark adds no information
- Input is raw reads rather than assemblies — SynTracker requires per-sample assembled
  contigs/genomes as target input
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0084
tools:
- name: SynTracker
  role: Primary tool under evaluation; computes APSS from genome synteny over 40 regions
    per pairwise comparison
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Used by SynTracker to build a BLAST database of target assemblies and search
    central regions (97% identity, 70% coverage cutoffs)
- name: inStrain
  role: Comparative baseline tool; strain calls from popANI thresholds (99.999% discussed
    in the article)
- name: MIDAS
  role: Comparative baseline tool; strain calls from ANI thresholds for ROC/AUC benchmarking
- name: StrainPhlAn
  role: Comparative baseline tool; strain calls from ANI thresholds for ROC/AUC benchmarking
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:9b23f9e95193361d3719d4368735765b5093e94ee6cf8769b83e044773507bac
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cross-tool-benchmark-reproduction

## Summary

Reproduce the premature-infant-twin strain-tracking benchmark in which SynTracker (APSS-based synteny scoring) is compared head-to-head against MIDAS, StrainPhlAn and inStrain using AUC and number of detected strain pairs, including phage/plasmid tracking.

## When to use

Use when a study requires reproducing a published tool-benchmark (same input dataset, multiple strain-comparison tools) with ROC/AUC and detection-count metrics, e.g. validating SynTracker's reported AUC 0.93 and 112 strain pairs.

## When NOT to use

- The goal is continuous genomic-difference measurement rather than same/different-strain classification — the article warns that strain delineation thresholds (e.g. popANI 99.999%, APSS 0.95) are subjective and species-dependent
- Only one tool is being run and no comparative baseline (inStrain/MIDAS/StrainPhlAn) is needed, so an AUC/detection-count benchmark adds no information
- Input is raw reads rather than assemblies — SynTracker requires per-sample assembled contigs/genomes as target input

## Inputs

- Per-sample metagenomic assembly FASTA files (premature-infant-twin benchmark, from SRA reads)
- Reference genome FASTA files (per-species, one file per genome, placed in a single reference directory)
- Phage and plasmid reference genomes for the phage/plasmid tracking comparison

## Outputs

- SynTracker APSS tables (avg_synteny_scores_40_regions.csv) per genome and in summary_output/
- ROC curves (sensitivity/specificity points from APSS and popANI/ANI threshold sweeps) for bacterial and phage/plasmid strain sharing
- AUC values per tool (SynTracker, inStrain, MIDAS, StrainPhlAn) and counts of detected strain pairs (e.g. 112 vs 85/92/101)

## How to apply

Obtain the premature-infant-twin benchmark metagenomic assemblies from SRA and organize them as per-sample assembly FASTA files in a target directory, with reference genomes in a reference directory. Run SynTracker in 'new' mode (python syntracker.py -target <target_dir> -ref <ref_dir>) with default parameters: BLAST minimal identity 97%, minimal query coverage 70%, compared region length 5000, seed=1 for reproducibility. Extract the APSS table computed with 40 regions per pair (avg_synteny_scores_40_regions.csv) for bacterial references and the corresponding phage/plasmid reference outputs. Sweep APSS classification thresholds (and popANI/ANI thresholds for inStrain, MIDAS and StrainPhlAn) to classify each sample pair as same-strain or different-strain, compute sensitivity/specificity points to build ROC curves, and derive AUC values and strain-pair counts. Validate against the published result: SynTracker attains AUC = 0.93 with 40 regions per comparison and detects 112 strain pairs (vs 85/92/101 for StrainPhlAn/inStrain/MIDAS), and its AUC on phages and plasmids is significantly higher than inStrain's.

## Related tools

- **SynTracker** (Primary tool under evaluation; computes APSS from genome synteny over 40 regions per pairwise comparison) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Used by SynTracker to build a BLAST database of target assemblies and search central regions (97% identity, 70% coverage cutoffs))
- **inStrain** (Comparative baseline tool; strain calls from popANI thresholds (99.999% discussed in the article))
- **MIDAS** (Comparative baseline tool; strain calls from ANI thresholds for ROC/AUC benchmarking)
- **StrainPhlAn** (Comparative baseline tool; strain calls from ANI thresholds for ROC/AUC benchmarking)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- SynTracker with 40 regions per pairwise comparison reaches AUC = 0.93 on the twin-infant benchmark
- Strain-pair detection counts match the reported ordering and values: 112 (SynTracker) vs 101 (MIDAS), 92 (inStrain), 85 (StrainPhlAn)
- SynTracker's AUC on phages and plasmids is significantly higher than inStrain's (Fig. 6a,b)
- Output files follow the expected schema: [genome name]_avg_synteny_scores_[subsampling length]_regions.csv under [genome_name]/final_output/ and summary_output/
- Repeated runs with seed=1 (default) reproduce identical APSS values; using --no_seed makes scores vary between runs

## Limitations

- Strain delineation thresholds (APSS 0.95, inStrain popANI 99.999%) are subjective, species-dependent, and the APSS 0.95 cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings
- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and sample size (maximal sensitivity/specificity of 98.6% and 97% was achieved with 200 regions per comparison)
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis, and BLAST hits lacking 2 kbp on both upstream and downstream sides are excluded
- The authors recommend avoiding same/different strain classifications in favor of continuous genomic difference measures because classification is inherently subjective

## Evidence

- [discussion] SynTracker outperformed MIDAS and StrainPhlAn and achieved a slightly better performance score than InStrain in terms of accuracy: "SynTracker outperformed MIDAS35 and StrainPhlAn36 and achieved a slightly better performance score than InStrain28 in terms of accuracy"
- [discussion] SynTracker achieved a significantly higher AUC than inStrain when applied to phages and plasmids.: "SynTracker achieved a significantly higher AUC than inStrain when applied to phages and plasmids."
- [discussion] maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [abstract] It outperforms existing tools when tracking strains in metagenomic data and is particularly suited for phages, plasmids and other low-data contexts.: "It outperforms existing tools when tracking strains in metagenomic data and is particularly suited for phages, plasmids and other low-data contexts."
- [readme] The tables containing the APSS in different subsampling lengths are called avg_synteny_scores_[subsampling length]_regions.csv.: "The tables containing the APSS in different subsampling lengths are called `avg_synteny_scores_[subsampling length]_regions.csv`."
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
