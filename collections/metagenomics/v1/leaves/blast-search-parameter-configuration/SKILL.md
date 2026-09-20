---
name: blast-search-parameter-configuration
description: Use when running SynTracker on metagenomic assemblies or genomes to detect
  conspecific strain sharing, and the BLAST identity/coverage, region length, and
  subsampling seed must be set for reproducible APSS computation.
when_to_use_negative:
- Input is raw sequencing reads rather than metagenomic assemblies/genomes — SynTracker
  requires assembled targets
- The analysis goal is SNP-level quantification rather than structural/synteny-based
  strain comparison (SNP-focused tools like inStrain, MIDAS or StrainPhlAn are more
  appropriate)
- A pre-existing BLAST database from syntracker_makeDB.py is being reused but you
  still pass -target, which is unnecessary and conflicting
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Pipeline that builds the BLAST database from target assemblies, searches reference
    central regions, and computes APSS scores
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: NCBI BLAST+ used for the homology search with 97% minimal identity and 70%
    minimal query coverage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:813b056fb6d772fddc33450c1a0151b40253351e833d9f7be2c1aaa0cf4c1520
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# blast-search-parameter-configuration

## Summary

Configure the BLASTn search stage of SynTracker so reference-genome central regions are matched against target metagenomic assemblies using the paper's validated thresholds (97% minimal identity, 70% minimal query coverage, 5000 bp compared regions, seed=1).

## When to use

Use when running SynTracker on metagenomic assemblies or genomes to detect conspecific strain sharing, and the BLAST identity/coverage, region length, and subsampling seed must be set for reproducible APSS computation.

## When NOT to use

- Input is raw sequencing reads rather than metagenomic assemblies/genomes — SynTracker requires assembled targets
- The analysis goal is SNP-level quantification rather than structural/synteny-based strain comparison (SNP-focused tools like inStrain, MIDAS or StrainPhlAn are more appropriate)
- A pre-existing BLAST database from syntracker_makeDB.py is being reused but you still pass -target, which is unnecessary and conflicting

## Inputs

- Target directory of per-sample metagenomic assembly FASTA files (or one FASTA per assembled genome)
- Reference genome directory with one FASTA per species (complete genomes or collections of contigs in a single file)
- Optional pre-built BLAST database directory produced by syntracker_makeDB.py

## Outputs

- Per-genome raw table [genome_name]_synteny_scores_per_region.csv
- APSS tables [genome_name]_avg_synteny_scores_[N]_regions.csv (N = 40, 60, 80, 100, 200 by default) plus avg_synteny_scores_all_regions.csv
- Summary output tables under summary_output/ aggregating all reference genomes

## How to apply

Organize per-sample metagenomic assemblies as single FASTA files in a target directory and reference genomes (complete or multi-contig, one FASTA per genome) in a reference directory. Run SynTracker in 'new' mode, which first builds a BLAST database from the targets (optionally pre-built with syntracker_makeDB.py for distributed batches) and then performs a BLAST search of each 1-kbp central region with --identity 97 (default) and --coverage 70 (default), with -length 5000 controlling the compared region length. After each hit, SynTracker retrieves a 2-kbp region on each side and excludes hits lacking >=2 kbp both upstream and downstream before pairwise synteny scoring. Keep the default seed=1 (do not pass --no_seed) so subsampling of n regions per pairwise comparison is reproducible across runs. These defaults reproduce the premature-infant-twin benchmark (AUC 0.93, 112 strain pairs with 40 regions per pairwise comparison).

## Related tools

- **SynTracker** (Pipeline that builds the BLAST database from target assemblies, searches reference central regions, and computes APSS scores) — https://github.com/leylabmpi/SynTracker
- **BLAST** (NCBI BLAST+ used for the homology search with 97% minimal identity and 70% minimal query coverage)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Output contains avg_synteny_scores_40_regions.csv (and 60/80/100/200/all variants) per genome and under summary_output/
- BLAST hits shorter than 2 kbp on either side of a hit are absent from downstream region tables (exclusion filter applied)
- Re-running with default seed=1 yields identical APSS values across runs; passing --no_seed changes scores between runs
- On the premature-infant-twin benchmark, 40-region APSS classification yields AUC = 0.93 and 112 detected strain pairs (vs 101 for MIDAS, 92 for inStrain, 85 for StrainPhlAn)

## Limitations

- Strain delineation thresholds such as APSS 0.95 are subjective and species-dependent; users must set their own cutoffs using a relevant training dataset
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis

## Evidence

- [discussion] BLAST search with default identity/coverage thresholds: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [methods] Seed parameter for reproducibility: "--no_seed         Set no seed for the subsampling of n regions per pairwise (optional)."
- [discussion] Exclusion filter for hits without flanking regions: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [readme] Default APSS subsampling output: "By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples"
- [results] Benchmark performance validating the configuration: [evidence span withheld — anchor status: not_found]
