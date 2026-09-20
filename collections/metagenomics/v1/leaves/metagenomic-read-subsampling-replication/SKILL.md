---
name: metagenomic-read-subsampling-replication
description: Use when a user wants to quantify how classification sensitivity and
  reconstruction completeness degrade with sequencing depth, e.g. testing a pipeline
  on SRA mock-community reads (FASTQ) at 10/1/0.1% subsampling across replicate trials.
when_to_use_negative:
- The dataset has no ground-truth mock community composition, so detection rates of
  known constituents cannot be scored.
- Only a single sequencing depth is available and the goal is one-shot classification
  rather than depth-robustness benchmarking.
- Input is already a taxonomic abundance table rather than raw FASTQ reads, so subsampling
  of original reads is impossible.
edam_operation: http://edamontology.org/operation_3230
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: VirMAP
  role: Pipeline under test; reconstructs and taxonomically classifies viral genomes
    from subsampled reads
  repo: https://github.com/cmmr/VirMAP
- name: Virmap.pl
  role: Command-line Perl entry point run on each subsample with --useMegahit and
    --useBbnorm
  repo: https://raw.githubusercontent.com/cmmr/virmap/master/virmapInstall.pl
- name: MEGAHIT
  role: De-novo assembler invoked via --useMegahit to build contigs from subsampled
    reads
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Read normalization (--useBbnorm) and nucleotide alignment to the viral database
- name: DIAMOND
  role: Translated protein alignment to the viral protein database (--virDmnd)
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:b2119fcb07f3b0d4faed6380ea199fb7b289a7eedca762e9e5778cdfb26e84d7
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
  edam_operation: http://edamontology.org/operation_3230
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# metagenomic-read-subsampling-replication

## Summary

Benchmark a viral metagenomic classifier's robustness by randomly subsampling paired FASTQ reads to fixed depth fractions (10%, 1%, 0.1%) across ten independent trials per depth, then scoring taxonomic detection and viral-base retention against full-depth results. It reproduces the VirMAP paper's viral mock community degradation analysis.

## When to use

Use when a user wants to quantify how classification sensitivity and reconstruction completeness degrade with sequencing depth, e.g. testing a pipeline on SRA mock-community reads (FASTQ) at 10/1/0.1% subsampling across replicate trials.

## When NOT to use

- The dataset has no ground-truth mock community composition, so detection rates of known constituents cannot be scored.
- Only a single sequencing depth is available and the goal is one-shot classification rather than depth-robustness benchmarking.
- Input is already a taxonomic abundance table rather than raw FASTQ reads, so subsampling of original reads is impossible.

## Inputs

- Paired-end FASTQ reads from SRA (SRR9875293, viral mock community, BioProject PRJNA431646)
- Prebuilt VirMAP database (taxaJson, virDmnd, virBbmap, gbBlastn, gbBlastx files in /scratch/VirmapDb)
- List of seven viral mock community constituents expected for detection scoring
- Full-depth reference run for computing relative viral-base retention

## Outputs

- 30 subsampled paired FASTQ datasets (ten trials each at 10%, 1%, 0.1% depth)
- Per-run VirMAP taxonomic classifications and viral reconstructions plus VirmapRun.err logs
- Aggregated detection-rate and viral-base-retention table (vmc_subsampling_detection_retention.csv)

## How to apply

Retrieve paired-end reads with fasterq-dump (e.g. SRR9875293 from BioProject PRJNA431646 into /home/$USER/VirmapTest), then randomly subsample the paired FASTQ files to 10%, 1%, and 0.1% of original depth, generating ten independent trials per depth (30 datasets total). Run Virmap.pl on each subsample with --threads $(nproc), --useMegahit, and --useBbnorm, pointing --taxaJson, --virDmnd, --virBbmap, --gbBlastn, and --gbBlastx to the prebuilt Genbank viral/phage database, and capture stderr in a VirmapRun.err log. Score each run for detection of the seven viral mock community constituents (VirMAP reported six of seven in all trials at 10% and 1%, with rotavirus never reported) and compute viral-base retention relative to the full-depth analysis. Aggregate the six-of-seven detection rates and retention values (VirMAP 55.55/25.99/10.0%; standard read-classification 24.76/5.36/0.11% at 10/1/0.1%) into a CSV such as vmc_subsampling_detection_retention.csv. Rationale: subsampling with replicate trials exposes how assembly-based reconstruction recovers proportionally more viral information at low coverage than per-read classification.

## Related tools

- **VirMAP** (Pipeline under test; reconstructs and taxonomically classifies viral genomes from subsampled reads) — https://github.com/cmmr/VirMAP
- **Virmap.pl** (Command-line Perl entry point run on each subsample with --useMegahit and --useBbnorm) — https://raw.githubusercontent.com/cmmr/virmap/master/virmapInstall.pl
- **MEGAHIT** (De-novo assembler invoked via --useMegahit to build contigs from subsampled reads) — https://github.com/voutcn/megahit
- **BBMap** (Read normalization (--useBbnorm) and nucleotide alignment to the viral database)
- **DIAMOND** (Translated protein alignment to the viral protein database (--virDmnd)) — https://github.com/bbuchfink/diamond

## Examples

```
fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293; Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Six of seven VMC constituents detected in 10/10 trials at both 10% and 1% subsampling, with rotavirus never reported at any depth.
- VirMAP viral-base retention reproduces approximately 55.55%, 25.99%, and 10.0% at 10%, 1%, and 0.1% subsampling respectively.
- Standard read-classification retention reproduces approximately 24.76%, 5.36%, and 0.11% at the same three depths.
- Exactly 30 independent subsample datasets generated (10 trials x 3 depths) and all 30 runs produce a scored result.
- stderr logs (VirmapRun.err) are free of fatal pipeline errors for each trial.

## Limitations

- Rotavirus was never detected in the VMC even at full depth, so detection-rate scores cap at six of seven constituents.
- Viral-base retention degrades sharply at low depth; at 0.1% subsampling VirMAP retains only 10.0% of viral bases.
- No software versions for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole are stated, and the Genbank database snapshot/date is unspecified, complicating exact reproduction.
- Supplementary Data 14 with sensitivity/specificity tuning parameters is referenced but not provided in the main document.

## Evidence

- [methods] SRA retrieval command: "fasterq-dump -t /dev/shm -e 4 -O /home/$USER/VirmapTest SRR9875293"
- [methods] Virmap.pl invocation options: "--useMegahit --useBbnorm --sampleName SRR9875293"
- [discussion] Mock community dataset: "Viral Mock Community, BioProject ID PRJNA431646"
- [discussion] VirMAP repository availability: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
- [discussion] Assembly strategy context: "VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs."
