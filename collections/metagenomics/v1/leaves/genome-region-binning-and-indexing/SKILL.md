---
name: genome-region-binning-and-indexing
description: Use when a pipeline needs homologous genomic regions extracted per sample
  from metagenomic assemblies relative to a reference genome, prior to pairwise synteny
  or strain-comparison scoring.
when_to_use_negative:
- Input is raw sequencing reads rather than assembled metagenomic contigs/genomes
  — SynTracker requires per-sample assembly files
- The analysis goal is SNP-based strain tracking on a fixed reference (e.g. MIDAS,
  StrainPhlAn, inStrain popANI) rather than synteny-based structural comparison
- Target sequences are expected to diverge beyond 97% identity to the reference —
  the BLASTn step will drop such homologs entirely
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Pipeline performing reference fragmentation, BLASTn search with 97% identity
    / 70% coverage thresholds, flank retrieval, and region-bin writing
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: Provides BLAST database construction (makeblastdb via syntracker_makeDB.py),
    the BLASTn search of central regions, and blastcmddb for retrieving 2-kbp flanks
    around hits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:16284b900c62842ac9f4445484306ffea6c719d96f4b6bbad35ab76758bcf2b3
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
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# genome-region-binning-and-indexing

## Summary

Generates and organizes homologous genomic regions from a reference genome and per-sample metagenomic assemblies by BLASTn-searching 1-kbp reference central regions, retrieving each hit with 2-kbp flanks, and writing hit-plus-flank sequences into region-specific bins with a TSV sample/region index.

## When to use

Use when a pipeline needs homologous genomic regions extracted per sample from metagenomic assemblies relative to a reference genome, prior to pairwise synteny or strain-comparison scoring.

## When NOT to use

- Input is raw sequencing reads rather than assembled metagenomic contigs/genomes — SynTracker requires per-sample assembly files
- The analysis goal is SNP-based strain tracking on a fixed reference (e.g. MIDAS, StrainPhlAn, inStrain popANI) rather than synteny-based structural comparison
- Target sequences are expected to diverge beyond 97% identity to the reference — the BLASTn step will drop such homologs entirely

## Inputs

- Reference genome(s) as complete sequences or a single multi-contig .fasta file (per-species, all contigs in one file)
- Per-sample metagenomic assembly FASTA files (or single-genome FASTA files), all stored in one target directory
- Optional pre-built BLAST database directory created by syntracker_makeDB.py with uniquely renamed target genomes

## Outputs

- Region-specific bin FASTA files per sample (region_bins.fasta) containing ~5-kbp hit-plus-2-kbp-flank sequences
- TSV index mapping each bin to its sample of origin and reference-region assignment
- BLAST database of uniquely renamed target genomes (when syntracker_makeDB.py is used)

## How to apply

First fragment the reference genome into 1-kbp central regions spaced 4 kbp apart using SynTracker's region-generation step. Build a BLAST database from the uniquely renamed target metagenomic assemblies (via syntracker_makeDB.py, or supply the target directory through -target). Run a BLASTn search of each central region against this database with a minimal identity of 97% and a minimal query coverage of 70%. Exclude hits lacking at least 2 kbp of sequence both upstream and downstream, and retrieve a 2-kbp region on each side of every passing hit using the 'blastcmddb' command. Write the extracted hit-plus-flank (~5 kbp) sequences into region-specific bins per sample as region_bins.fasta, accompanied by a TSV index mapping each bin to its sample of origin and reference-region assignment.

## Related tools

- **SynTracker** (Pipeline performing reference fragmentation, BLASTn search with 97% identity / 70% coverage thresholds, flank retrieval, and region-bin writing) — https://github.com/leylabmpi/SynTracker
- **BLAST** (Provides BLAST database construction (makeblastdb via syntracker_makeDB.py), the BLASTn search of central regions, and blastcmddb for retrieving 2-kbp flanks around hits)

## Examples

```
python syntracker_makeDB.py -target Sample_Data/Input_example/Target_genomes/ -out blastDB_output/ ; python syntracker.py -blastDB blastDB_output/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- Every retained bin sequence derives from a BLAST hit meeting the 97% identity and 70% query-coverage thresholds
- Bins are only produced for hits having at least 2 kbp of flanking sequence both upstream and downstream; insufficiently flanked hits are absent from region_bins.fasta
- Each bin is represented in the TSV index with a valid sample-of-origin and reference-region assignment
- Each bin sequence spans the 1-kbp central region plus 2-kbp flanks on both sides (≈5 kbp total, matching the default region_length=5000)
- The BLAST database target names match the uniquely renamed genomes produced by syntracker_makeDB.py

## Limitations

- Hits within <2 kbp both upstream and downstream are excluded from further analysis, so homologs near contig boundaries are silently lost
- Central regions absent from a sample's assemblies (below identity/coverage thresholds) simply yield no bin for that sample; pairs of samples with fewer than n shared regions are excluded downstream
- Rebuilding the BLAST database from many targets is time-consuming; the README recommends syntracker_makeDB.py plus continue_all_genomes mode to avoid repeating it across reference genomes

## Evidence

- [discussion] Fragment the reference genome into a collection of 1-kbp central regions spaced 4 kbp apart using SynTracker's region-generation step: "Each per-species reference is fragmented into a collection of 1-kbp central regions, which are binned and stored together."
- [discussion] BLASTn thresholds and flank retrieval: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] Flank exclusion criterion: "Hits within <2 kbp both upstream and downstream to the hit are excluded from further analysis."
- [discussion] Flank retrieval command: "a 2-kbp region on each side of the BLAST hit using the ‘blastcmddb’ command"
- [readme] README on inputs and makeDB script: "Run the script `syntracker_makeDB.py` to create a directory containing the uniquely renamed target genomes
and the blast database created from them"
