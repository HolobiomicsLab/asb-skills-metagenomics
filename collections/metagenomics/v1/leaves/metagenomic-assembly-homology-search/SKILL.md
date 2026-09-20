---
name: metagenomic-assembly-homology-search
description: Use when you have a single reference genome (or per-species references)
  and per-sample metagenomic assemblies/genomes, and need to extract conserved homologous
  regions with flanking context before downstream strain-comparison analyses.
when_to_use_negative:
- Input is raw sequencing reads rather than assemblies/genomes — SynTracker requires
  per-sample assembled contigs in FASTA form, not FASTQ
- The analysis goal is SNP-level strain differentiation only — the article notes most
  strain-comparison methods quantify SNP differences; SynTracker targets structural
  variation via genome synteny, and hits below 97% identity/70% coverage will be lost
- A single sample/assembly is provided — homologous-region binning and downstream
  APSS comparisons require pairs of samples or genomes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Pipeline orchestrating reference fragmentation into 1-kbp central regions,
    BLASTn search, flank retrieval, and per-region binning of homologous regions
  repo: https://github.com/leylabmpi/SynTracker/releases
- name: BLAST
  role: BLASTn search of each reference central region against a BLAST database built
    from the target metagenomic assemblies, using blastcmddb to retrieve 2-kbp flanks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:47e48df145324f328d8a02908ee934e094e57c61dad198e34395353b062cdc8e
schema_version: 0.2.0
metadata:
  license_tier: open
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
n_claims: 13
---

# metagenomic-assembly-homology-search

## Summary

Identifies homologous genomic regions in per-sample metagenomic assemblies by BLASTn-searching fragmented reference-genome central regions (97% identity, 70% query coverage) and binning each hit plus 2-kbp flanks per region and sample, as the first stage of the SynTracker strain-tracking pipeline.

## When to use

Use when you have a single reference genome (or per-species references) and per-sample metagenomic assemblies/genomes, and need to extract conserved homologous regions with flanking context before downstream strain-comparison analyses.

## When NOT to use

- Input is raw sequencing reads rather than assemblies/genomes — SynTracker requires per-sample assembled contigs in FASTA form, not FASTQ
- The analysis goal is SNP-level strain differentiation only — the article notes most strain-comparison methods quantify SNP differences; SynTracker targets structural variation via genome synteny, and hits below 97% identity/70% coverage will be lost
- A single sample/assembly is provided — homologous-region binning and downstream APSS comparisons require pairs of samples or genomes

## Inputs

- Reference genome FASTA file(s) (complete genomes or contigs of the same species in a single .fasta file, in a reference directory)
- Per-sample metagenomic assembly FASTA files (or single-genome FASTA files) organized in a target directory
- Optional parameters: -length region length (default 5000), --identity (default 97), --coverage (default 70)

## Outputs

- region_bins.fasta files containing hit-plus-2-kbp-flank sequences organized into region-specific bins per sample
- A TSV index mapping each region bin to its sample of origin and reference-region assignment
- A BLAST database of uniquely renamed target genomes (from syntracker_makeDB.py)

## How to apply

Fragment each reference genome into 1-kbp central regions spaced 4 kbp apart (SynTracker's region-generation step, with -length controlling the compared-region length, default 5000 bp). Build a BLAST database from uniquely renamed target metagenomic assemblies using syntracker_makeDB.py, or supply the target directory via -target. Run a BLASTn search of each central region against the database with a minimal identity of 97% and minimal query coverage of 70% (tunable via --identity and --coverage). Exclude hits lacking at least 2 kbp both upstream and downstream of the hit, and retrieve a 2-kbp region on each side of each passing hit using the 'blastcmddb' command. Write each extracted hit-plus-flank sequence (~5 kbp) into region-specific bins per sample as region_bins.fasta, accompanied by a TSV index mapping each bin to its sample of origin and reference-region assignment.

## Related tools

- **SynTracker** (Pipeline orchestrating reference fragmentation into 1-kbp central regions, BLASTn search, flank retrieval, and per-region binning of homologous regions) — https://github.com/leylabmpi/SynTracker/releases
- **BLAST** (BLASTn search of each reference central region against a BLAST database built from the target metagenomic assemblies, using blastcmddb to retrieve 2-kbp flanks)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Every retained bin sequence derives from a BLAST hit meeting the 97% identity and 70% query-coverage thresholds
- Each retained hit possesses at least 2 kbp of flanking sequence on both upstream and downstream sides; bins from insufficiently flanked hits are excluded
- Each bin sequence is approximately the expected ~5-kbp length (1-kbp central region plus 2×2-kbp flanks, per -length default 5000)
- The TSV index maps each region bin to exactly one sample of origin and one reference-region assignment, with no unmapped bins
- The BLAST database contains uniquely renamed target genomes (verifiable from the syntracker_makeDB.py output directory)

## Limitations

- Hits within <2 kbp both upstream and downstream are excluded, so regions near contig ends or in poorly assembled samples are systematically lost
- The default 97% identity / 70% coverage BLAST thresholds limit recovery of divergent or distantly related conspecific regions
- Pairs of samples or genomes with fewer than n homologous regions are excluded from downstream analysis, and a small number of regions per pairwise comparison mostly results in lower accuracy (accuracy/specificity trade-off with sample size)

## Evidence

- [discussion] Fragmentation and BLASTn search: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] Flank retrieval and filtering: "a 2-kbp region on each side of the BLAST hit using the ‘blastcmddb’ command"
- [discussion] Hit exclusion rule: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [discussion] Central-region generation: "Each per-species reference is fragmented into a collection of 1-kbp central regions, which are binned and stored together."
- [methods] CLI parameter defaults: "--identity [blast_identity] Minimal blast identity (optional, default=97)"
- [readme] BLAST database construction script: "Run the script `syntracker_makeDB.py` to create a directory containing the uniquely renamed target genomes and the blast database created from them"
- [discussion] Downstream region-count dependency: "Pairs of samples or genomes with fewer than n regions are excluded from downstream analysis."
