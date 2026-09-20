---
name: viral-mock-community-benchmarking
description: Use when a taxonomic classifier (read-, contig-, or marker-gene-based)
  must be validated on sequencing data with a known ground-truth viral composition,
  or when robustness to reduced sequencing depth must be quantified.
when_to_use_negative:
- The input dataset has no known ground-truth viral composition — precision/recall/F-score
  against intended constituents cannot be computed and retention cannot be referenced
  to a full-depth baseline.
- The goal is taxonomic classification of a real environmental or clinical virome
  rather than benchmarking a classifier against mock-community truth.
- The input is already a classified taxonomy table or feature table — rerunning the
  pipeline and rescoring would be redundant.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: VirMAP
  role: Pipeline under test; merges nucleotide and protein information to taxonomically
    classify viral reconstructions and reconstruct viral bases from the mock community
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: Multi-kmer de-novo assembler used by Virmap.pl via --useMegahit to generate
    contigs for classification
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Used for digital normalization (--useBbnorm) and nucleotide alignment to viral
    databases during VirMAP classification
- name: DIAMOND
  role: Protein/translated alignment engine (virDmnd.dmnd, gbBlastx.dmnd) supporting
    VirMAP's protein-level taxonomic assignment
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:e03939576311dcbbde5f703784b6fb66a826a285faf661c21574edd520754964
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - GNU parallel
  - MEGAHIT
  - Perl
  - VirMAP
  - Virmap.pl
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# viral-mock-community-benchmarking

## Summary

Benchmark a viral taxonomic classification pipeline against a viral mock community of known composition (e.g. PRJNA431646's seven constituents) by computing precision, recall, and F-score, and by quantifying detection robustness and viral-base retention under depth subsampling.

## When to use

Use when a taxonomic classifier (read-, contig-, or marker-gene-based) must be validated on sequencing data with a known ground-truth viral composition, or when robustness to reduced sequencing depth must be quantified.

## When NOT to use

- The input dataset has no known ground-truth viral composition — precision/recall/F-score against intended constituents cannot be computed and retention cannot be referenced to a full-depth baseline.
- The goal is taxonomic classification of a real environmental or clinical virome rather than benchmarking a classifier against mock-community truth.
- The input is already a classified taxonomy table or feature table — rerunning the pipeline and rescoring would be redundant.

## Inputs

- Paired-end viral mock community FASTQ reads from SRA accession SRR9875293 (BioProject PRJNA431646)
- Ground-truth list of the seven intended viral mock community constituents
- Pre-built VirMAP databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd in /scratch/VirmapDb)
- Subsampled FASTQ datasets at 10%, 1%, and 0.1% depth (ten trials per depth)

## Outputs

- Per-run taxonomic classification outputs in the VirmapRun directory (script_runs and result files)
- Precision, recall, and F-score table comparing the pipeline against VirMAP, the standard MEGAHIT+BLASTn/BLASTx approach, and eight published classifiers
- Aggregated detection-rate and viral-base retention table (vmc_subsampling_detection_retention.csv) with six-of-seven detection rates and retention percentages at each subsampling depth

## How to apply

Run the pipeline under test on the Viral Mock Community paired-end reads (SRA SRR9875293, BioProject PRJNA431646), e.g. Virmap.pl with --threads $(nproc), --useMegahit, and --useBbnorm against pre-built databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd). Score output classifications against the seven intended viral constituents, counting true positives, false positives (e.g. the bosavirus MS-2016a cell-culture contaminant), and misses to compute precision, recall, and F-score. For robustness, randomly subsample the paired FASTQ files to 10%, 1%, and 0.1% depth in ten independent trials per depth (30 datasets), rerun the pipeline, and record which of the seven constituents are detected per trial plus the percentage of viral bases reconstructed relative to the full-depth run (VirMAP reported 55.55%, 25.99%, and 10.0% retention at 10%, 1%, 0.1%; the standard MEGAHIT+BLAST approach 24.76%, 5.36%, and 0.11%). Aggregate metrics into a CSV (e.g. vmc_subsampling_detection_retention.csv) and compare against published values.

## Related tools

- **VirMAP** (Pipeline under test; merges nucleotide and protein information to taxonomically classify viral reconstructions and reconstruct viral bases from the mock community) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (Multi-kmer de-novo assembler used by Virmap.pl via --useMegahit to generate contigs for classification) — https://github.com/voutcn/megahit
- **BBMap** (Used for digital normalization (--useBbnorm) and nucleotide alignment to viral databases during VirMAP classification)
- **DIAMOND** (Protein/translated alignment engine (virDmnd.dmnd, gbBlastx.dmnd) supporting VirMAP's protein-level taxonomic assignment) — https://github.com/bbuchfink/diamond

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293 && Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- VirmapRun output directory exists and contains populated VirMAP taxonomic classification results for the mock community run (file_exists/script_runs checks).
- Full-depth recall of 1.0 — all seven intended VMC constituents are detected; precision near 0.88 with the single false positive being the bosavirus MS-2016a cell-culture contaminant.
- Subsampled detection rate: six of seven constituents reported in all trials at 10% and 1% depth (rotavirus never reported).
- Viral-base retention values reproduce the published curve: 55.55%, 25.99%, 10.0% for VirMAP and 24.76%, 5.36%, 0.11% for the standard approach at 10%, 1%, 0.1% respectively.
- All 30 subsample runs complete with stderr captured in a VirmapRun.err log before aggregation.

## Limitations

- Rotavirus, one of the seven VMC constituents, was never reported by VirMAP under subsampling, so full seven-of-seven detection is not achievable at reduced depth.
- Precision on the full-depth VMC is lowered by a genuine cell-culture contaminant (bosavirus MS-2016a) rather than a pipeline error — ground truth itself can be imperfect.
- VirMAP databases are Genbank-derived (gbvrl, gbphage/gbphg divisions); as Genbank expands, minimum SSD and RAM requirements grow, and no software versions or Genbank snapshot date are stated in the paper.

## Evidence

- [results] VirMAP achieved precision, recall, and F-score values of 0.88, 1.0, and 0.94: [evidence span withheld — anchor status: not_found]
- [methods] fasterq-dump retrieval command: "fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293"
- [methods] Virmap.pl invocation with databases: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [methods] Assembly tooling options: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [discussion] Dereplication and normalization of input reads: "input reads are dereplicated and digitally normalized18 using default parameters"
- [discussion] Viral mock community BioProject identifier: "Viral Mock Community, BioProject ID PRJNA431646"
- [intro] Database source divisions: "clustered nucleotide and translated alignments to the Genbank viral and phage divisions (gbvrl and gbphage)"
- [methods] Database growth caveat: "As Genbank expands, the minimum SSD and RAM requirements will expand as well."
