---
name: blast-hit-flanking-sequence-extraction
description: Use when homologous genomic regions must be recovered from per-sample
  assemblies via BLASTn against a fragmented reference, and each hit needs flanking
  context for synteny block detection.
when_to_use_negative:
- The goal is SNP-based strain comparison rather than synteny — tools like MIDAS,
  StrainPhlAn, or inStrain are more appropriate, and SNP-based methods are insensitive
  to the structural changes SynTracker targets
- Target sequences do not meet the 97% identity / 70% coverage thresholds or lack
  ≥2 kbp flanking sequence on both sides — such hits are excluded and cannot support
  region binning
- Working with phages/plasmids or low-data contexts requiring no database — while
  SynTracker suits these, this flanking-extraction step specifically presumes a reference-fragmented
  BLAST workflow with per-sample assemblies
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: SynTracker
  role: Pipeline orchestrating reference fragmentation, BLASTn search (97% identity,
    70% coverage), hit filtering, and region-binned output generation
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Provides BLASTn search against the target database and the 'blastcmddb' command
    used to retrieve 2-kbp flanks on each side of hits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:229ba7744e05be768c5c89d4050231ae10be9489456c8dbf60882043df7cc481
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# blast-hit-flanking-sequence-extraction

## Summary

Extracts BLASTn hits plus fixed 2-kbp flanking sequences on both sides from metagenomic assemblies, binning hit-plus-flank sequences by reference region for downstream synteny scoring in SynTracker.

## When to use

Use when homologous genomic regions must be recovered from per-sample assemblies via BLASTn against a fragmented reference, and each hit needs flanking context for synteny block detection.

## When NOT to use

- The goal is SNP-based strain comparison rather than synteny — tools like MIDAS, StrainPhlAn, or inStrain are more appropriate, and SNP-based methods are insensitive to the structural changes SynTracker targets
- Target sequences do not meet the 97% identity / 70% coverage thresholds or lack ≥2 kbp flanking sequence on both sides — such hits are excluded and cannot support region binning
- Working with phages/plasmids or low-data contexts requiring no database — while SynTracker suits these, this flanking-extraction step specifically presumes a reference-fragmented BLAST workflow with per-sample assemblies

## Inputs

- Reference genome (complete or multi-contig .fasta, all sequences in one file)
- Per-sample metagenomic assemblies or target genomes (one .fasta per sample) in a target directory
- BLAST database built by syntracker_makeDB.py from uniquely renamed target genomes

## Outputs

- region_bins.fasta per sample: extracted hit-plus-2-kbp-flank sequences organized into region-specific bins
- TSV index mapping each bin to its sample of origin and reference-region assignment

## How to apply

Fragment the reference genome into 1-kbp central regions spaced 4 kbp apart, build a BLAST database from the uniquely renamed target metagenomic assemblies (via syntracker_makeDB.py or -target), and run a BLASTn search per central region with --identity 97 (minimal identity) and --coverage 70 (minimal query coverage). Exclude hits lacking at least 2 kbp both upstream and downstream of the hit, then retrieve a 2-kbp region on each side of each passing BLAST hit using the 'blastcmddb' command. Write each extracted hit-plus-flank sequence (~5 kbp total) into region-specific bins per sample as region_bins.fasta, with a TSV index mapping each bin to its sample of origin and reference-region assignment. Flanking context matters because the later FindSynteny step requires alignments with a minimal overlap of 4,800 bp to detect syntenic blocks.

## Related tools

- **SynTracker** (Pipeline orchestrating reference fragmentation, BLASTn search (97% identity, 70% coverage), hit filtering, and region-binned output generation) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Provides BLASTn search against the target database and the 'blastcmddb' command used to retrieve 2-kbp flanks on each side of hits)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Every retained bin sequence must derive from a BLAST hit meeting the 97% minimal identity and 70% minimal query coverage thresholds
- Every retained bin must possess ≥2 kbp of flanking sequence both upstream and downstream of the hit; bins from insufficiently flanked hits must be excluded
- Each extracted bin sequence should be ~5 kbp (1-kbp central region plus 2×2-kbp flanks)
- A TSV index must accompany region_bins.fasta mapping each bin to its sample of origin and reference-region assignment
- Downstream FindSynteny pairwise comparisons should consider only alignments with a minimal overlap of 4,800 bp, confirming flank retrieval was sufficient

## Limitations

- Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis, which drops contig-edge hits and short contigs
- The default BLAST thresholds (97% identity, 70% coverage) may miss more divergent homologs, limiting detection of distantly related strains
- Extraction is per reference region; pairs of samples with fewer than n regions are excluded from downstream analysis, so insufficient flanked hits reduce the number of comparable regions

## Evidence

- [discussion] performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis.: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [discussion] a 2-kbp region on each side of the BLAST hit using the 'blastcmddb' command: [evidence span withheld — anchor status: not_found]
- [discussion] only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis: "only pairwise comparisons with a minimal overlap of 4,800 bp are considered for downstream analysis"
- [readme] Run the script syntracker_makeDB.py to create a directory containing the uniquely renamed target genomes and the blast database created from them: "Run the script `syntracker_makeDB.py` to create a directory containing the uniquely renamed target genomes
and the blast database created from them"
