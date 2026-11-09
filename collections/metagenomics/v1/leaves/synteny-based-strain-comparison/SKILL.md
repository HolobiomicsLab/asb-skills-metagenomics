---
name: synteny-based-strain-comparison
description: Use when comparing strains of the same microbial species from metagenomic
  assemblies or isolate genomes — especially when SNP-based tools (inStrain popANI,
  MIDAS, StrainPhlAn) call pairs clonal but structural differences may exist.
when_to_use_negative:
- When the goal is purely SNP-level differentiation of hypermutator strains — SynTracker
  has low sensitivity to SNPs, so an SNP-based tool like inStrain popANI or read-mapping
  analysis is the appropriate primary method
- When pairs of samples/genomes share fewer than n homologous regions — SynTracker
  excludes such pairs from downstream analysis
- When a universal same-strain cutoff is expected — the authors warn that thresholds
  (e.g., popANI 99.999%, APSS 0.95) are subjective, species-dependent, and tailored
  cutoffs require a training dataset relevant to the research question
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: 'Primary pipeline: fragments reference genomes, BLAST-searches target assemblies,
    computes per-alignment synteny scores and APSS'
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: NCBI BLAST+ creates the BLAST database of target genomes and searches 1-kbp
    central reference regions with 97% identity / 70% coverage filters
- name: DECIPHER R package
  role: FindSynteny function (maxGap and maxSep 15) performs pairwise alignments to
    identify synteny blocks
- name: inStrain
  role: Complementary SNP-based tool computing popANI; intersected with SynTracker
    comparisons to classify hypermutators vs hyper-recombinants
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:31eb410df21389090826b21852e596fe34fc21d80045345056ed64e1016f1501
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - DECIPHER R package
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# synteny-based-strain-comparison

## Summary

Compare conspecific microbial strains using genome synteny via SynTracker: BLAST-derived homologous regions per reference genome are scored for synteny block conservation and averaged into APSS values, detecting structural variation that SNP-based tools miss.

## When to use

Use when comparing strains of the same microbial species from metagenomic assemblies or isolate genomes — especially when SNP-based tools (inStrain popANI, MIDAS, StrainPhlAn) call pairs clonal but structural differences may exist.

## When NOT to use

- When the goal is purely SNP-level differentiation of hypermutator strains — SynTracker has low sensitivity to SNPs, so an SNP-based tool like inStrain popANI or read-mapping analysis is the appropriate primary method
- When pairs of samples/genomes share fewer than n homologous regions — SynTracker excludes such pairs from downstream analysis
- When a universal same-strain cutoff is expected — the authors warn that thresholds (e.g., popANI 99.999%, APSS 0.95) are subjective, species-dependent, and tailored cutoffs require a training dataset relevant to the research question

## Inputs

- Reference genome FASTA file(s) per species (complete or multi-contig in a single .fasta), e.g., GCF_900087635.2 (N. gonorrhoeae), NC_000913.3 (E. coli), CP032479.1 (H. pylori), GCF_000331185.2 (S. rimosus)
- Target directory of per-sample metagenomic assembly FASTAs or per-isolate genome FASTAs (all in one directory)

## Outputs

- Per-reference [genome_name]_avg_synteny_scores_[N]_regions.csv tables (APSS for 40/60/80/100/200 subsampled regions) in [genome_name]/final_output/
- [genome_name]_avg_synteny_scores_all_regions.csv (APSS using all available regions, no subsampling)
- [genome_name]_synteny_scores_per_region.csv (raw per-region synteny scores)
- Combined multi-genome tables in summary_output/ (avg_synteny_scores_all_regions.csv, synteny_scores_per_region.csv)
- Downstream same/different strain classification tables and hypergeometric species enrichment results (species_enrichment_results.csv) when combined with inStrain popANI

## How to apply

Install SynTracker from the latest GitHub release inside the 'SynTracker_1_4' conda environment created from SynTracker_env.yml (requires NCBI BLAST+). Run in 'new' mode: 'python syntracker.py -target <target_dir> -ref <ref_dir> -out SynTracker_output/' with default parameters — compared region length 5000, minimal BLAST identity 97, minimal BLAST coverage 70, seed=1 for reproducible subsampling. The pipeline fragments each per-species reference into 1-kbp central regions, BLASTs them against a database of the target assemblies, retrieves hits plus 2-kbp flanks, and uses FindSynteny (maxGap and maxSep 15; minimal overlap 4,800 bp) to compute pairwise synteny scores that are averaged into APSS over n randomly subsampled regions (200 regions per pair gives maximal sensitivity/specificity of 98.6%/97%). Extract per-pair APSS from avg_synteny_scores_all_regions.csv or the subsampled tables, exclude pairs with fewer than n regions, and for strain classification combine APSS with inStrain popANI (e.g., APSS > 0.955 and popANI > 0.99999; APSS 0.95 for human gut). Rationale: SynTracker has low sensitivity to SNPs but detects structural differences, so pairing it with popANI classifies species as hypermutators vs hyper-recombinants.

## Related tools

- **SynTracker** (Primary pipeline: fragments reference genomes, BLAST-searches target assemblies, computes per-alignment synteny scores and APSS) — https://github.com/leylabmpi/SynTracker
- **BLAST** (NCBI BLAST+ creates the BLAST database of target genomes and searches 1-kbp central reference regions with 97% identity / 70% coverage filters)
- **DECIPHER R package** (FindSynteny function (maxGap and maxSep 15) performs pairwise alignments to identify synteny blocks)
- **inStrain** (Complementary SNP-based tool computing popANI; intersected with SynTracker comparisons to classify hypermutators vs hyper-recombinants)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Each output CSV exists and contains all expected pairwise comparisons for its species (e.g., 66 rows for N. gonorrhoeae comparisons, 185 for hypermutator E. coli pairs)
- Spearman correlation between APSS and popANI over the 66 N. gonorrhoeae comparisons equals 0.985
- APSS values fall within the expected range; with seed=1 (default), re-running SynTracker reproduces the same subsampled APSS values
- Combined analysis recovers the expected signatures: all 185 hypermutator E. coli pairs called same strain by both tools, S. rimosus showing a wide APSS range despite clonal popANI, and enrichment of Phocaeicola vulgatus/Bacteroides fragilis/Alistipes putredinis (hypermutators) vs Phocaeicola massiliensis/Streptococcus thermophilus/Streptococcus gallolyticus (hyper-recombinators)
- Using 200 regions per pairwise comparison yields maximal average sensitivity (98.6%) and specificity (97%)

## Limitations

- The APSS 0.95 cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings; strain delineation thresholds are subjective and species-dependent
- Small numbers of regions per pairwise comparison mostly result in lower accuracy — a trade-off exists between accuracy and sample size
- Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis, and classifying genomes into strains is inherently subjective; the authors recommend favoring continuous genomic difference measures over same/different classifications

## Evidence

- [abstract] SynTracker has low sensitivity to SNPs, no database requirement, and is robust to sequencing errors: "SynTracker has low sensitivity to SNPs, has no database requirement and is robust to sequencing errors."
- [discussion] n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [discussion] Maximal sensitivity and specificity with 200 regions per comparison: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] Pairs with fewer than n regions are excluded from downstream analysis: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [discussion] APSS cutoff of 0.95 designates the same strain in the human gut: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
- [methods] Minimal blast identity default is 97: "--identity [blast_identity]
                    Minimal blast identity (optional, default=97)"
- [readme] Conda environment creation from SynTracker_env.yml: "Create a new conda environment using the ‘SynTracker_env.yml’ file (located in the root directory of SynTrakcer)"
