---
name: reference-genome-fragmentation
description: Use when comparing conspecific microbial strains across metagenomic assemblies
  or genomes with SynTracker and you need homologous genomic regions binned per sample
  from a single reference genome.
when_to_use_negative:
- The goal is to detect strain differences driven mainly by point mutations — SynTracker
  has low sensitivity to SNPs, so an SNP-based tool (e.g., MIDAS, StrainPhlAn, inStrain)
  is preferable
- Input is raw sequencing reads rather than per-sample metagenomic assemblies or assembled
  genomes
- Only short homologous stretches exist — hits without at least 2 kbp both upstream
  and downstream are excluded and yield no bins
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SynTracker
  role: Pipeline performing the reference-genome fragmentation into central regions,
    BLASTn search, flank retrieval, and binning of homologous regions
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Provides the BLAST database and BLASTn search used to find homologs of each
    1-kbp central region in the target assemblies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:bfb2a4fcf7d0fdc26b886c139a812ec8e0e106df23f234647942a635387e6250
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# reference-genome-fragmentation

## Summary

Fragment a per-species reference genome into 1-kbp central regions (spaced 4 kbp apart) and use BLASTn to retrieve homologous hit-plus-2-kbp-flank regions (~5 kbp) from per-sample metagenomic assemblies, organizing them into region-specific bins that feed SynTracker's synteny-based strain tracking.

## When to use

Use when comparing conspecific microbial strains across metagenomic assemblies or genomes with SynTracker and you need homologous genomic regions binned per sample from a single reference genome.

## When NOT to use

- The goal is to detect strain differences driven mainly by point mutations — SynTracker has low sensitivity to SNPs, so an SNP-based tool (e.g., MIDAS, StrainPhlAn, inStrain) is preferable
- Input is raw sequencing reads rather than per-sample metagenomic assemblies or assembled genomes
- Only short homologous stretches exist — hits without at least 2 kbp both upstream and downstream are excluded and yield no bins

## Inputs

- Reference genome(s) as a single .fasta file per species (complete genome or collection of contigs in one file)
- Per-sample metagenomic assemblies or assembled genomes, one .fasta file per sample, stored in a target directory

## Outputs

- region_bins.fasta files containing homologous ~5-kbp hit-plus-2-kbp-flank sequences per region, per sample
- TSV index mapping each region bin to its sample of origin and reference-region assignment
- BLAST database of uniquely renamed target genomes (created by syntracker_makeDB.py)

## How to apply

Fragment each reference genome into 1-kbp central regions spaced 4 kbp apart using SynTracker's region-generation step (region length controlled by -length, default 5000). Build a BLAST database from the uniquely renamed target metagenomic assemblies using syntracker_makeDB.py (or pass the target directory via -target). Run a BLASTn search of each central region against the BLAST database with a minimal identity of 97% (default, --identity) and minimal query coverage of 70% (default, --coverage). Exclude hits lacking at least 2 kbp of sequence both upstream and downstream, then retrieve a 2-kbp region on each side of every passing hit using the 'blastcmddb' command. Write each hit-plus-flank sequence (~5 kbp) into region-specific bins per sample as region_bins.fasta, accompanied by a TSV index mapping each bin to its sample of origin and reference-region assignment.

## Related tools

- **SynTracker** (Pipeline performing the reference-genome fragmentation into central regions, BLASTn search, flank retrieval, and binning of homologous regions) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Provides the BLAST database and BLASTn search used to find homologs of each 1-kbp central region in the target assemblies)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Every retained bin sequence derives from a BLAST hit meeting the 97% minimal identity and 70% minimal query coverage thresholds
- All retained hits possess at least 2 kbp of flanking sequence on both sides; hits with <2 kbp upstream or downstream are absent from the bins
- Each bin sequence is ~5 kbp total: the hit plus 2-kbp flanks on each side of the 1-kbp central region query
- A TSV index exists alongside each region_bins.fasta, mapping every bin to its sample of origin and reference-region assignment
- Central regions are spaced 4 kbp apart along each reference genome, so region bins are non-overlapping in reference coordinates

## Limitations

- Hits within <2 kbp of a contig end (upstream or downstream) are excluded from further analysis, biasing bins toward interior genomic regions
- Default thresholds (region length 5000, identity 97, coverage 70) are configurable but changing them alters which homologous regions are recovered
- By default a seed=1 is set for downstream subsampling of n regions per pairwise comparison; use --no_seed to allow run-to-run variation
- Central-region binning is per reference genome, so multi-species runs require one reference genome file per species in the reference directory

## Evidence

- [readme] Fragmentation of the reference genomes and execution of BLASTn search against the target genomes/metagenomes.: "A. Fragmentation of the reference genomes and execution of BLASTn search against the target genomes/metagenomes."
- [discussion] Each per-species reference is fragmented into a collection of 1-kbp central regions, which are binned and stored together.: "Each per-species reference is fragmented into a collection of 1-kbp central regions, which are binned and stored together."
- [discussion] BLAST search with 97% minimal identity and 70% minimal query coverage.: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] Retrieve 2-kbp flanks with blastcmddb; exclude hits lacking flanks.: "a 2-kbp region on each side of the BLAST hit using the ‘blastcmddb’ command"
- [discussion] Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis.: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [readme] Minimal blast identity parameter default of 97 in syntracker.py.: "Minimal blast identity (optional, default=97)"
