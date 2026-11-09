---
name: sequencing-depth-sensitivity-analysis
description: Use when you need to quantify how a viral metagenomic classifier/reconstruction
  pipeline (e.g. VirMAP on VMC reads) degrades as sequencing depth is reduced, or
  to compare retention of a reconstruction pipeline against a standard read-classification
  baseline at shallow depth.
when_to_use_negative:
- The question is a single full-depth taxonomic profile of a virome sample — run VirMAP
  once rather than a subsampling sensitivity sweep
- The goal is quantitative abundance estimation — random subsampling to 0.1% depth
  destroys read-count proportionality and only supports detection/retention robustness
  claims
- The dataset is not viral-focused (e.g. host-genome or bacterial mock communities)
  — VirMAP filters contigs to the viral superkingdom, so non-viral sensitivity is
  out of scope
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: VirMAP
  role: Pipeline under test; Virmap.pl is invoked on each subsample with --useMegahit
    --useBbnorm to reconstruct viral bases and classify mock constituents
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: De-novo assembler enabled via the --useMegahit flag for the multi-kmer assembly
    stage on shallow subsamples
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Used through --useBbnorm for digital normalization of subsampled reads and
    for nucleotide mapping to viral references
- name: DIAMOND
  role: Protein aligner backing the --virDmnd translated-alignment step used to classify
    viral protein pseudo-scaffolds in each subsample run
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:4748796af1cdec3c3f41ae69bc61d38ab909f6c6347965b4be74ab10e4f213cd
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sequencing-depth-sensitivity-analysis

## Summary

A robustness benchmark that randomly subsamples a viral mock community (VMC) FASTQ dataset to 10%, 1%, and 0.1% depth (ten trials each), reruns VirMAP on every subsample, and reports detection rates of the seven mock constituents plus viral-base retention relative to the full-depth run.

## When to use

Use when you need to quantify how a viral metagenomic classifier/reconstruction pipeline (e.g. VirMAP on VMC reads) degrades as sequencing depth is reduced, or to compare retention of a reconstruction pipeline against a standard read-classification baseline at shallow depth.

## When NOT to use

- The question is a single full-depth taxonomic profile of a virome sample — run VirMAP once rather than a subsampling sensitivity sweep
- The goal is quantitative abundance estimation — random subsampling to 0.1% depth destroys read-count proportionality and only supports detection/retention robustness claims
- The dataset is not viral-focused (e.g. host-genome or bacterial mock communities) — VirMAP filters contigs to the viral superkingdom, so non-viral sensitivity is out of scope

## Inputs

- Paired-end FASTQ reads of a viral mock community (SRR9875293, BioProject PRJNA431646)
- Subsampled paired FASTQ files at 10%, 1%, and 0.1% depth, ten trials per depth (30 datasets)
- Prebuilt VirMAP database files (--taxaJson, --virDmnd, --virBbmap, --gbBlastn, --gbBlastx) built from Genbank viral (gbvrl) and phage (gbphg) divisions

## Outputs

- Per-trial detection calls for each of the seven VMC constituents
- Fraction of trials per depth in which six of seven viruses were detected
- Viral-base retention percentages per depth for VirMAP and the standard approach
- VirmapRun.err stderr logs per run
- Aggregated results table vmc_subsampling_detection_retention.csv

## How to apply

Retrieve the paired-end VMC reads (SRR9875293, BioProject PRJNA431646) with fasterq-dump using scratch temp (-t /dev/shm -e 4), then randomly subsample the paired FASTQs to 10%, 1%, and 0.1% of original depth, generating ten independent trials per depth (30 datasets total). Run Virmap.pl on each subsample with --threads $(nproc), --useMegahit, and --useBbnorm, pointing --taxaJson, --virDmnd, --virBbmap, --gbBlastn, and --gbBlastx at the prebuilt database (e.g. /scratch/VirmapDb) and redirecting stderr to a VirmapRun.err log. For each of the 30 runs, score detection of the seven mock community members and compute the fraction of trials per depth in which six of seven viruses are detected. Quantify viral-base retention as the percentage of viral bases recovered in each reconstruction relative to the full-depth analysis, computing the same metric for the standard read-classification approach as a baseline. Aggregate into vmc_subsampling_detection_retention.csv and compare the retention curve against the published values (VirMAP 55.55/25.99/10.0%; standard 24.76/5.36/0.11% at 10/1/0.1%).

## Related tools

- **VirMAP** (Pipeline under test; Virmap.pl is invoked on each subsample with --useMegahit --useBbnorm to reconstruct viral bases and classify mock constituents) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (De-novo assembler enabled via the --useMegahit flag for the multi-kmer assembly stage on shallow subsamples) — https://github.com/voutcn/megahit
- **BBMap** (Used through --useBbnorm for digital normalization of subsampled reads and for nucleotide mapping to viral references)
- **DIAMOND** (Protein aligner backing the --virDmnd translated-alignment step used to classify viral protein pseudo-scaffolds in each subsample run) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- At 10% and 1% depth, six of the seven VMC constituents are reported in all ten trials per depth, with rotavirus consistently absent (seven-of-seven never reached)
- Viral-base retention for VirMAP falls to approximately 55.55% at 10%, 25.99% at 1%, and 10.0% at 0.1% subsampling, while the standard read-classification approach retains approximately 24.76%, 5.36%, and 0.11%
- Exactly 30 subsampled datasets (10 trials x 3 depths) are generated and each produces a completed Virmap.pl run with a VirmapRun.err stderr log
- Retention values and detection rates are correctly aggregated into vmc_subsampling_detection_retention.csv keyed by depth and trial
- The 0.1%-depth retention values should remain strictly positive (VirMAP ~10.0%) but far below full-depth recovery, confirming monotonic degradation with depth

## Limitations

- Rotavirus was never reported among VMC constituents even at 10% and 1% depth, capping detection at six of seven viruses
- VirMAP viral-base retention degrades steeply below 1% depth (only 10.0% retained at 0.1%), so conclusions at ultra-shallow depth are limited
- No software versions for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole and no Genbank database snapshot/date are stated, complicating exact reproduction
- Supplementary Data 14 (parameters tuning the sensitivity/specificity tradeoff) is referenced but not provided, limiting parameter-level re-tuning

## Evidence

- [methods] VMC retrieval command for the subsampling benchmark: "`fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293`"
- [methods] Virmap.pl invocation flags used on each subsample: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [discussion] Jaccard-distance filtering that governs which genomes survive at shallow depth: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed"
- [discussion] Open-source availability of the pipeline being stress-tested: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
- [readme] Mock community validation context for the subsampled VMC experiment: "We validate VirMAP using published data sets and viral mock communities containing RNA and DNA viruses and bacteriophages"
