---
name: viral-base-retention-quantification
description: Use when measuring how much viral sequence information a reconstruction
  pipeline retains as sequencing depth decreases — e.g., subsampled paired-end FASTQ
  from a viral mock community at 10%, 1%, and 0.1% depth with replicate trials per
  depth.
when_to_use_negative:
- Input is an already-assembled viral genome or contigs rather than subsampled read
  data — retention must be measured from reconstructions of the same sample at varying
  depth
- The goal is general taxonomic classification of a virome without a full-depth reference
  run to define the 100% retention baseline
- The dataset lacks a defined mock community composition (e.g., seven known constituents),
  making detection-rate scoring against known members impossible
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0781
tools:
- name: VirMAP
  role: Reconstructs viral bases from each subsample via mapping assembly merged with
    de-novo assembly; Virmap.pl is the driver script
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: De-novo multi-kmer assembler invoked with --useMegahit on each subsample
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Used with --useBbnorm for digital normalization and nucleotide alignment to
    viral databases
- name: DIAMOND
  role: Protein/translated alignment (virDmnd database) supporting classification
    of reconstructions
  repo: https://github.com/bbuchfink/diamond
- name: GNU parallel
  role: Parallelizes the 30 subsample runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:b1b417ae16921f2e42ece323b4bbc3fd477086293a0a047cc0c4753979ca8e31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - GNU parallel
  - MEGAHIT
  - VirMAP
  - Virmap.pl
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# viral-base-retention-quantification

## Summary

Quantifies the percentage of viral bases recovered by VirMAP reconstructions relative to a full-depth analysis of the Viral Mock Community (BioProject PRJNA431646), benchmarking retention across 10%, 1%, and 0.1% subsampling depths (ten trials each) against a standard read-classification approach.

## When to use

Use when measuring how much viral sequence information a reconstruction pipeline retains as sequencing depth decreases — e.g., subsampled paired-end FASTQ from a viral mock community at 10%, 1%, and 0.1% depth with replicate trials per depth.

## When NOT to use

- Input is an already-assembled viral genome or contigs rather than subsampled read data — retention must be measured from reconstructions of the same sample at varying depth
- The goal is general taxonomic classification of a virome without a full-depth reference run to define the 100% retention baseline
- The dataset lacks a defined mock community composition (e.g., seven known constituents), making detection-rate scoring against known members impossible

## Inputs

- Paired-end FASTQ reads of the Viral Mock Community (SRR9875293, BioProject PRJNA431646) retrieved via fasterq-dump
- Randomly subsampled paired FASTQ datasets: 10%, 1%, and 0.1% of original read depth, ten independent trials per depth (30 datasets total)
- Prebuilt VirMAP Genbank viral/phage reference databases (taxaJson, virDmnd, virBbmap, gbBlastn, gbBlastx) in /scratch/VirmapDb
- Full-depth VirMAP analysis output as the 100% retention reference

## Outputs

- Per-run viral-base retention percentages relative to the full-depth analysis for each of the 30 subsampled runs
- Six-of-seven mock community constituent detection rates per depth
- Standard read-classification retention values (24.76%, 5.36%, 0.11% at 10%, 1%, 0.1%) as comparison baseline
- vmc_subsampling_detection_retention.csv aggregating detection rates and retention values

## How to apply

Retrieve the viral mock community paired-end reads with fasterq-dump (SRR9875293, BioProject PRJNA431646), then randomly subsample the paired FASTQ files to 10%, 1%, and 0.1% of original read depth, generating ten independent trials per depth (30 datasets total). Run Virmap.pl on each subsample with --threads $(nproc), --useMegahit, and --useBbnorm, pointing --taxaJson, --virDmnd, --virBbmap, --gbBlastn, and --gbBlastx to the prebuilt database in /scratch/VirmapDb and logging stderr to a VirmapRun.err file. For each of the 30 runs, score detection of the seven mock community members and compute the fraction of trials per depth in which six of seven viruses were detected (rotavirus is never reported). Quantify viral-base retention as the percentage of viral bases recovered in each VirMAP reconstruction relative to the full-depth analysis, and compute the same metric for the standard read-classification approach as a baseline. Aggregate the six-of-seven detection rates and retention values into vmc_subsampling_detection_retention.csv, expecting VirMAP retention of 55.55%, 25.99%, and 10.0% versus 24.76%, 5.36%, and 0.11% for the standard approach at 10%, 1%, and 0.1% depth respectively.

## Related tools

- **VirMAP** (Reconstructs viral bases from each subsample via mapping assembly merged with de-novo assembly; Virmap.pl is the driver script) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (De-novo multi-kmer assembler invoked with --useMegahit on each subsample) — https://github.com/voutcn/megahit
- **BBMap** (Used with --useBbnorm for digital normalization and nucleotide alignment to viral databases)
- **DIAMOND** (Protein/translated alignment (virDmnd database) supporting classification of reconstructions) — https://github.com/bbuchfink/diamond
- **GNU parallel** (Parallelizes the 30 subsample runs)

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293; Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --taxaJson /scratch/VirmapDb --virDmnd /scratch/VirmapDb 2>VirmapRun.err
```

## Evaluation signals

- VirMAP retention at 10%, 1%, and 0.1% subsampling reproduces the published 55.55%, 25.99%, and 10.0% values
- Standard read-classification retention matches the published 24.76%, 5.36%, and 0.11% at the three depths
- Six of seven mock community constituents (rotavirus never reported) are detected in all trials at 10% and 1% depth
- All 30 subsample runs (ten trials x three depths) complete without errors, with stderr captured in VirmapRun.err logs
- Retention is computed relative to the same full-depth analysis for both VirMAP and the standard approach, ensuring a fair comparison

## Limitations

- Rotavirus is never reported even at full detection tier — retention and detection metrics exclude it by design of the mock community results
- No software versions are stated for VirMAP, BBMap, DIAMOND, or MEGAHIT, and the Genbank database snapshot/version is not specified, complicating exact reproduction
- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the text, affecting sensitivity/specificity tuning
- Supplementary Data 14 (sensitivity/specificity tuning parameters) is referenced but not provided in the document

## Evidence

- [results] VirMAP reported six of seven VMC constituents in all trials at 10% and 1% (rotavirus never reported): [evidence span withheld — anchor status: not_found]
- [results] Retention percentages published: [evidence span withheld — anchor status: not_found]
- [methods] Virmap.pl invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq`"
- [discussion] Read assignment and Jaccard filtering: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome. Genomic read sets are then filtered by Jaccard distance"
- [discussion] Open-source availability: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
