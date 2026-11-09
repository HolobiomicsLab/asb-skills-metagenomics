---
name: metagenomic-assembly-strain-comparison
description: Use when tracking shared or closely related strains between pairs of
  metagenomic assemblies (or genomes) of the same species, especially when structural
  variation, low-data contexts (phages/plasmids), or SNP-insensitivity is required
  rather than popANI/SNP counting.
when_to_use_negative:
- The target sequences are not conspecific with any available reference genome — SynTracker
  requires per-species reference genomes and only compares strains of the same species
- The question demands SNP-level resolution of point-mutation differences only — popANI/SNP
  tools (inStrain, MIDAS, StrainPhlAn) are better suited; SynTracker is deliberately
  low-sensitivity to SNPs
- Inputs are raw reads rather than assembled contigs — samples must be provided as
  per-sample metagenomic assemblies, not FASTQ
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: 'Main pipeline: fragments references, BLAST-searches regions against target
    assemblies, computes synteny scores and APSS via ''new''/''continue'' modes'
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: BLASTn search of 1-kbp central regions against the BLAST database built from
    target metagenomic assemblies, with 97% minimal identity and 70% minimal query
    coverage; blastcmddb retrieves 2-kbp flanking regions
- name: DECIPHER R package
  role: FindSynteny function (maxGap and maxSep set to 15) performs all-versus-all
    pairwise alignments within region bins to identify syntenic blocks for scoring
- name: phangorn R package
  role: Builds UPGMA phylogenetic trees from APSS-converted synteny distances
- name: inStrain
  role: Complementary popANI-based tool for intersecting comparisons and distinguishing
    hypermutator vs hyper-recombinant species
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:a95d3dc057b3982ec3b90c2f2cb5ef4dc902761da4ed9784b2f3bff8ff17cf22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - DECIPHER R package
  - MIDAS
  - StrainPhlAn
  - SynTracker
  - inStrain
  - phangorn R package
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metagenomic-assembly-strain-comparison

## Summary

Compare conspecific microbial strains across metagenomic assemblies or genomes using SynTracker's genome-synteny pipeline: BLAST-based homologous-region retrieval followed by DECIPHER synteny alignment and Average Pairwise Synteny Score (APSS) computation. It enables strain tracking where SNP-based tools fail, including phages and plasmids.

## When to use

Use when tracking shared or closely related strains between pairs of metagenomic assemblies (or genomes) of the same species, especially when structural variation, low-data contexts (phages/plasmids), or SNP-insensitivity is required rather than popANI/SNP counting.

## When NOT to use

- The target sequences are not conspecific with any available reference genome — SynTracker requires per-species reference genomes and only compares strains of the same species
- The question demands SNP-level resolution of point-mutation differences only — popANI/SNP tools (inStrain, MIDAS, StrainPhlAn) are better suited; SynTracker is deliberately low-sensitivity to SNPs
- Inputs are raw reads rather than assembled contigs — samples must be provided as per-sample metagenomic assemblies, not FASTQ

## Inputs

- Reference genome(s) in FASTA (complete or multi-contig, one .fasta per genome) in a reference directory
- Per-sample metagenomic assemblies (one .fasta of all contigs per sample) or target genomes in a target directory
- Optional: pre-built BLAST database directory produced by syntracker_makeDB.py for distributed batches

## Outputs

- [genome_name]_synteny_scores_per_region.csv — raw per-region pairwise synteny scores per reference genome
- [genome_name]_avg_synteny_scores_[40|60|80|100|200]_regions.csv — APSS tables after region subsampling
- [genome_name]_avg_synteny_scores_all_regions.csv — APSS using all available regions without subsampling
- summary_output/ combined tables across all reference genomes (synteny_scores_per_region.csv and APSS tables)

## How to apply

Organize per-sample metagenomic assemblies (one .fasta per sample) into a target directory and per-species reference genomes into a reference directory, then run SynTracker in 'new' mode (`python syntracker.py -target <dir> -ref <dir>`) with default parameters (region length 5000, BLAST minimal identity 97%, minimal query coverage 70%, seed=1 for reproducibility). SynTracker fragments each reference into 1-kbp central regions with 2-kbp flanks, BLAST-searches them against the target BLAST database, excludes hits lacking 2 kbp on both sides, and computes per-region synteny scores with the DECIPHER FindSynteny function (maxGap and maxSep both 15), retaining only pairwise alignments with minimal overlap of 4,800 bp. It then randomly subsamples n regions per sample pair (default outputs at 40, 60, 80, 100, 200 regions, plus all-regions) and averages pairwise synteny scores into APSS, written to `[genome_name]_avg_synteny_scores_[n]_regions.csv` and a combined `summary_output/` table. Classify sample pairs as same-strain with an APSS cutoff (e.g., 0.95 in the human gut) or use continuous APSS values, converting to synteny distances (|APSS − 1|) for UPGMA tree building. Validate that APSS scores are ≤ 1, that pairs with fewer than n regions are excluded, and, where a benchmark exists, that sensitivity/specificity (e.g., 98.6%/97% at 200 regions) match reported values.

## Related tools

- **SynTracker** (Main pipeline: fragments references, BLAST-searches regions against target assemblies, computes synteny scores and APSS via 'new'/'continue' modes) — https://github.com/leylabmpi/SynTracker
- **BLAST** (BLASTn search of 1-kbp central regions against the BLAST database built from target metagenomic assemblies, with 97% minimal identity and 70% minimal query coverage; blastcmddb retrieves 2-kbp flanking regions)
- **DECIPHER R package** (FindSynteny function (maxGap and maxSep set to 15) performs all-versus-all pairwise alignments within region bins to identify syntenic blocks for scoring)
- **phangorn R package** (Builds UPGMA phylogenetic trees from APSS-converted synteny distances)
- **inStrain** (Complementary popANI-based tool for intersecting comparisons and distinguishing hypermutator vs hyper-recombinant species)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- All computed synteny scores are ≤ 1 (maximal value of 1 for a single block with 100% overlap)
- Output directory contains per-genome final_output/ tables plus a summary_output/ APSS table; APSS CSVs exist for each default subsampling length (40, 60, 80, 100, 200) and all-regions
- Sample pairs with fewer than n detected regions are excluded from the subsampled APSS tables
- On the premature-infant-twin benchmark with 40 regions per comparison, APSS-based classification attains AUC = 0.93 and detects 112 strain pairs (vs 85/92/101 for StrainPhlAn/inStrain/MIDAS)
- Maximal average sensitivity and specificity of 98.6% and 97% are reproduced when using 200 regions per pairwise comparison

## Limitations

- Strain delineation cutoffs (e.g., APSS 0.95, inStrain popANI 99.999%) are subjective and species-dependent; the 0.95 cutoff was an average of per-species cutoffs and may not be optimal for all settings — users should set context-specific cutoffs with a relevant training dataset
- A small number of regions per pairwise comparison mostly results in lower accuracy; there is a trade-off between accuracy and sample size
- Without --no_seed, a seed=1 is set for subsampling reproducibility; using --no_seed makes APSS values vary between runs
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis, which can drop low-coverage or low-data comparisons (e.g., small phage/plasmid genomes)
- Continuing an interrupted run requires 'continue' mode; the BLAST database building stage is time-consuming for many targets, motivating the syntracker_makeDB.py two-stage workflow

## Evidence

- [abstract] We introduce SynTracker, a tool that compares microbial strains using genome synteny-the order of sequence blocks in homologous genomic regions-in pairs of metagenomic assemblies or genomes.: "We introduce SynTracker, a tool that compares microbial strains using genome synteny-the order of sequence blocks in homologous genomic regions-in pairs of metagenomic assemblies or genomes."
- [discussion] performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis: "only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis"
- [discussion] Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis.: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [discussion] we set an APSS score cutoff of 0.95 to designate the same strain in the human gut: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [readme] These data should be organized in per-sample assembly files - i.e., all the contigs assembled from sample X would be kept in a single file.: "These data should be organized in per-sample assembly files - i.e., all the contigs assembled from sample X would be kept in a single file."
- [readme] By default, a seed=1 is set to enable reproducibility between different runs.: "By default, a seed=1 is set to enable reproducibility between different runs."
- [readme] The second type of output tables, `[genome name]_avg_synteny_scores_[subsampling length]_regions.csv`, gives the APSS (Average Pairwise Synteny Score): "The second type of output tables, `[genome name]_avg_synteny_scores_[subsampling length]_regions.csv`, gives the APSS (Average Pairwise Synteny Score)"
