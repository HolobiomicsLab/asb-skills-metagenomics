---
name: blast-database-construction
description: 'Use when running SynTracker''s first stage: you have per-sample metagenomic
  assemblies or isolate genomes in a target directory and need them indexed as a BLAST
  database for homologous-region retrieval against reference genome fragments.'
when_to_use_negative:
- Comparing sequences against a generic public database (e.g. nr) rather than a user-provided
  collection of target metagenomic assemblies/genomes — SynTracker requires its own
  DB built from the target directory
- Strain comparison tasks relying on SNP differences alone (e.g. inStrain popANI),
  since synteny-based region retrieval is insensitive to point mutations and targets
  structural variation
- 'Datasets where reads are not assembled: the skill operates on metagenomic assemblies/genomes,
  not raw FASTQ'
edam_operation: http://edamontology.org/operation_2421
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: SynTracker
  role: Pipeline providing syntracker_makeDB.py (builds the uniquely renamed target
    genomes + BLAST database) and the region-generation and BLASTn search steps that
    consume it
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: NCBI BLAST+ used to create the database from target assemblies and to run
    the BLASTn search of central regions (blastcmddb retrieves hit-plus-flank sequences)
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:686c6c87e9c8222adbce75c5fcbe8ae4282bb524f86bd4aff742dae7e5e4d9a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_2421
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# blast-database-construction

## Summary

Build a BLAST database from a user-provided collection of per-sample metagenomic assemblies (or genomes), uniquely renamed via SynTracker's syntracker_makeDB.py, so that SynTracker's 1-kbp reference central regions can be BLASTn-searched with 97% identity / 70% coverage thresholds to retrieve homologous ~5-kbp regions into region bins.

## When to use

Use when running SynTracker's first stage: you have per-sample metagenomic assemblies or isolate genomes in a target directory and need them indexed as a BLAST database for homologous-region retrieval against reference genome fragments.

## When NOT to use

- Comparing sequences against a generic public database (e.g. nr) rather than a user-provided collection of target metagenomic assemblies/genomes — SynTracker requires its own DB built from the target directory
- Strain comparison tasks relying on SNP differences alone (e.g. inStrain popANI), since synteny-based region retrieval is insensitive to point mutations and targets structural variation
- Datasets where reads are not assembled: the skill operates on metagenomic assemblies/genomes, not raw FASTQ

## Inputs

- Per-sample metagenomic assembly FASTA files (one file per sample, all contigs of a sample in a single file) or per-genome FASTA files, stored in a single target directory
- Reference genome FASTA files (complete genomes or contig collections, one file per reference genome) in a reference directory
- NCBI BLAST+ (makeblastdb/BLASTn via SynTracker)

## Outputs

- A blastDB directory containing uniquely renamed target genomes and the BLAST database created by syntracker_makeDB.py
- Per-sample, region-specific bins (region_bins.fasta) of homologous ~5-kbp regions (1-kbp central region plus 2-kbp flanks on each side) passing 97% identity / 70% coverage thresholds with >=2 kbp flanks
- A TSV index mapping each region bin to its sample of origin and reference-region assignment

## How to apply

Organize the inputs as per-sample FASTA files (all contigs of sample X in one file, all files in a single target directory), then run SynTracker's syntracker_makeDB.py to uniquely rename the target sequences and create the BLAST database. SynTracker then fragments each per-species reference into 1-kbp central regions spaced 4 kbp apart (compared region length default 5000) and BLASTn-searches each central region against this database with minimal identity 97% and minimal query coverage 70% (tunable via --identity and --coverage). Retrieved hits must have at least 2 kbp of flanking sequence both upstream and downstream; passing hits are expanded with a 2-kbp region on each side using the 'blastcmddb' command and written into region-specific region_bins.fasta files per sample, accompanied by a TSV index mapping each bin to its sample of origin and reference-region assignment. For large multi-reference datasets, build the database once and reuse it via the -blastDB argument or 'continue_all_genomes' mode to avoid repeating the time-consuming DB-building stage.

## Related tools

- **SynTracker** (Pipeline providing syntracker_makeDB.py (builds the uniquely renamed target genomes + BLAST database) and the region-generation and BLASTn search steps that consume it) — https://github.com/leylabmpi/SynTracker
- **BLAST** (NCBI BLAST+ used to create the database from target assemblies and to run the BLASTn search of central regions (blastcmddb retrieves hit-plus-flank sequences))

## Examples

```
python syntracker_makeDB.py -target Sample_Data/Input_example/Target_genomes/ -out blastDB_output/ && python syntracker.py -blastDB blastDB_output/ -out SynTracker_output/
```

## Evaluation signals

- Every retained bin sequence derives from a BLAST hit meeting >=97% identity and >=70% query coverage, and possesses >=2 kbp of flanking sequence on both sides; hits with <2 kbp flanks are excluded
- The blastDB directory (from syntracker_makeDB.py) contains uniquely renamed target genomes plus the BLAST database files, so -target no longer needs to be supplied
- Each region bin is a ~5-kbp sequence (1-kbp central region + 2-kbp upstream + 2-kbp downstream), stored as region_bins.fasta per sample
- A TSV index accompanies the bins, mapping each bin to its sample of origin and reference-region assignment
- Downstream APSS tables (avg_synteny_scores_*.csv) are produced only for sample pairs with at least n homologous regions

## Limitations

- Hits within <2 kbp both upstream and downstream are excluded, so assemblies with fragmented contigs near region boundaries lose those homologous regions
- Building the BLAST database from many targets can be very time-consuming; use -blastDB or 'continue_all_genomes' mode to avoid repeating it across reference genomes
- Pairs of samples/genomes with fewer than n homologous regions are excluded from downstream APSS analysis; small numbers of regions mostly lower accuracy (maximal sensitivity/specificity 98.6%/97% at 200 regions per pairwise comparison)

## Evidence

- [discussion] BLAST database built from user-provided target assemblies: "SynTracker creates a BLAST database based on a user-provided collection of metagenomic assemblies or genomes."
- [discussion] BLASTn search parameters (97% identity, 70% coverage): "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [readme] syntracker_makeDB.py invocation from README: "python syntracker_makeDB.py -target Sample_Data/Input_example/Target_genomes/ -out blastDB_output/"
- [readme] continue_all_genomes avoids repeating DB building: "without repeating the stage in which a blast database is built from the target genomes (which can be very time-consuming in case of many targets)"
- [methods] identity/coverage CLI defaults: "--identity [blast_identity] Minimal blast identity (optional, default=97)"
