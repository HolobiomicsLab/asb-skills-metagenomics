---
name: blast-identity-coverage-filtering
description: Use when running SynTracker strain-tracking analyses and you need to
  filter BLASTn hits of reference central regions against target assemblies/genomes
  by identity and coverage thresholds, or when reproducing sensitivity tests over
  these parameters.
when_to_use_negative:
- Input is raw sequencing reads rather than metagenomic assemblies or assembled genomes
  — SynTracker compares assemblies/genomes, not reads, so BLAST-based filtering cannot
  be applied directly
- Strains/species are so divergent that homologous central regions fall below the
  97% minimal BLAST identity threshold — hits will be filtered out and pairs with
  fewer than n regions are excluded from downstream analysis
- You need SNP-level strain discrimination (e.g., popANI-style comparison with inStrain
  or MIDAS) — SynTracker's synteny approach is explicitly insensitive to SNPs, so
  identity/coverage filtering here does not capture point-mutation differences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Pipeline that fragments reference genomes into central regions, performs the
    BLASTn search with 97%/70% identity/coverage filters, retrieves 2-kbp flanks via
    blastcmddb, and computes downstream synteny scores and APSS
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Search engine used to build the target-genome database and identify homologous
    central-region hits subject to the minimal identity and coverage thresholds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:a645ebb143201bdd8883cf98116b751951560e8207d37aa582d989ec90a67c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# blast-identity-coverage-filtering

## Summary

Apply SynTracker's default BLAST-based filters — minimal 97% identity and 70% query coverage — to retain only high-confidence homologous region hits between a reference genome and target genomes/metagenomes before synteny scoring, plus exclusion of hits lacking ≥2 kbp of flanking sequence on both sides.

## When to use

Use when running SynTracker strain-tracking analyses and you need to filter BLASTn hits of reference central regions against target assemblies/genomes by identity and coverage thresholds, or when reproducing sensitivity tests over these parameters.

## When NOT to use

- Input is raw sequencing reads rather than metagenomic assemblies or assembled genomes — SynTracker compares assemblies/genomes, not reads, so BLAST-based filtering cannot be applied directly
- Strains/species are so divergent that homologous central regions fall below the 97% minimal BLAST identity threshold — hits will be filtered out and pairs with fewer than n regions are excluded from downstream analysis
- You need SNP-level strain discrimination (e.g., popANI-style comparison with inStrain or MIDAS) — SynTracker's synteny approach is explicitly insensitive to SNPs, so identity/coverage filtering here does not capture point-mutation differences

## Inputs

- Per-species reference genome FASTA files (complete genomes or contig collections, one species per file, all in a single reference directory)
- Per-sample metagenomic assembly FASTAs or assembled genome FASTAs (one file per sample/genome, all in a single target directory)

## Outputs

- Filtered BLAST hit set of homologous central regions (≥97% identity, ≥70% query coverage, with ≥2 kbp flanks on both sides) for retrieval of 5-kbp regions
- Per-region synteny scores and BLAST identities (e.g., synteny_scores_per_region.csv) downstream of the filtered hits
- APSS tables (avg_synteny_scores_[N]_regions.csv and avg_synteny_scores_all_regions.csv) built from the filtered regions

## How to apply

SynTracker first fragments each per-species reference into 1-kbp central regions and builds a BLAST database from the user-provided target metagenomic assemblies or genomes. Each central region is searched with BLASTn against this database, retaining hits with minimal identity of 97% and minimal query coverage of 70%; these defaults are exposed via the --identity and --coverage command-line arguments and can be adjusted if a data type demands it. Hits within <2 kbp of both upstream and downstream sequence are then excluded from further analysis so that the 2-kbp flanking regions required for downstream FindSynteny alignment can be retrieved with blastcmddb. The filtered hit set feeds the 5-kbp region length (default) used for pairwise synteny-score calculation, so overly permissive thresholds would admit spurious homologs into APSS computation. Reproducibility of filtering relies on running in 'new' mode with default seed=1 for subsequent region subsampling.

## Related tools

- **SynTracker** (Pipeline that fragments reference genomes into central regions, performs the BLASTn search with 97%/70% identity/coverage filters, retrieves 2-kbp flanks via blastcmddb, and computes downstream synteny scores and APSS) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Search engine used to build the target-genome database and identify homologous central-region hits subject to the minimal identity and coverage thresholds)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/ --identity 97 --coverage 70
```

## Evaluation signals

- All retained BLAST hits have identity ≥97% and query coverage ≥70%; changing these thresholds via --identity/--coverage changes the number of hits monotonically
- Every retained hit has ≥2 kbp of sequence both upstream and downstream; hits within <2 kbp flanks are absent from downstream FindSynteny comparisons
- Downstream output files (synteny_scores_per_region.csv, avg_synteny_scores_all_regions.csv) exist under [genome_name]/final_output/ and summary_output/ and are populated for retained regions
- With a SNP-only simulation regime at 1e-6 events per nucleotide per generation, minimal APSS remains very high (~0.995/0.995/0.981), confirming filtering admits true homologs despite high SNP frequency
- Running with default seed=1 yields reproducible subsampled APSS values across repeated runs; --no_seed makes them vary between runs

## Limitations

- The 97% identity and 70% coverage defaults are parameter choices, not validated per-dataset optima; pairs of samples or genomes with fewer than n regions surviving the filters are excluded from downstream analysis
- SynTracker is explicitly insensitive to SNPs (sensitivity to SNPs is low and robust to sequencing errors), so identity/coverage filtering will not capture point-mutation-driven divergence
- Adjusting the default BLAST thresholds or the region length (-length, default=5000) without re-validating downstream APSS cutoffs (e.g., the APSS 0.95 same-strain cutoff, which is species-dependent and subjective) can compromise interpretation

## Evidence

- [discussion] Create a BLAST database from user-provided metagenomic assemblies or genomes and search central regions with 97% minimal identity and 70% minimal query coverage: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] Exclude BLAST hits lacking 2-kbp flanks: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [methods] CLI exposure of the thresholds: "--identity [blast_identity]
                    Minimal blast identity (optional, default=97)"
- [discussion] Exclusion of under-resourced pairs: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
- [abstract] Low SNP sensitivity rationale: "SynTracker has low sensitivity to SNPs, has no database requirement and is robust to sequencing errors."
