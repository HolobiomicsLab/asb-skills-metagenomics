---
name: subsampled-assembly-benchmark-comparison
description: 'Reference for reproducing or extending depth-benchmarking of viral genome
  reconstruction: subsample a large FASTQ cohort to lower read depths, rerun assembly-based
  reconstruction at each depth, and compare segment contiguity/length against a baseline
  method.'
when_to_use_negative:
- The dataset is small or low-depth already — subsampling provides no informative
  gradient and comparison power is lost.
- The question is taxonomic abundance profiling rather than reconstruction contiguity/length
  benchmarking — depth-vs-length comparison is not the right evaluation.
- No baseline/standard-approach reconstruction is run at matched depths — comparing
  VirMAP against itself across depths does not test the method.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0781
tools:
- name: VirMAP
  role: Reconstruction method under benchmarking; builds virus-like super-scaffolds
    from nucleotide and translated alignments, with --useMegahit --useBbnorm enabled
  repo: https://github.com/cmmr/VirMAP
- name: MEGAHIT
  role: De-novo assembler invoked by VirMAP (--useMegahit) during reconstruction at
    each subsampled depth
  repo: https://github.com/voutcn/megahit
- name: BBMap
  role: Used within VirMAP (with --useBbnorm) for read normalization and mapping assembly
    of subsampled reads
  repo: https://github.com/cmmr/VirMAP
- name: DIAMOND
  role: Translated protein alignment backend (virDmnd.dmnd) used to recruit reads
    at each depth
  repo: https://github.com/bbuchfink/diamond
- name: Tadpole
  role: Alternative assembler within VirMAP's multi-kmer de-novo assembly strategy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:8d26eee8968697053bf576285d0fbbdebe146faa0134c6d961917bd1d6316255
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - MEGAHIT
  - Perl
  - Tadpole
  - VirMAP
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# subsampled-assembly-benchmark-comparison

## Summary

Benchmark a reconstruction pipeline against a baseline by downsampling sequence data to graded depths, rerunning reconstruction at each depth, and comparing output contiguity/length. In the VirMAP influenza UMERS dataset (PRJEB7888, 1.75 billion reads), this reproduces the Fig. 4 VirMAP-vs-standard-approach comparison.

## When to use

Reference for reproducing or extending depth-benchmarking of viral genome reconstruction: subsample a large FASTQ cohort to lower read depths, rerun assembly-based reconstruction at each depth, and compare segment contiguity/length against a baseline method.

## When NOT to use

- The dataset is small or low-depth already — subsampling provides no informative gradient and comparison power is lost.
- The question is taxonomic abundance profiling rather than reconstruction contiguity/length benchmarking — depth-vs-length comparison is not the right evaluation.
- No baseline/standard-approach reconstruction is run at matched depths — comparing VirMAP against itself across depths does not test the method.

## Inputs

- Paired-end metagenomic FASTQ files from the influenza UMERS cohort (BioProject PRJEB7888, ~1.75 billion reads across 25 samples)
- Prebuilt VirMAP database files (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd)
- Subsampling depth schedule (target read counts or fractions of the full depth)

## Outputs

- Per-depth reconstruction outputs (VirMAP super-scaffolds and standard-approach assemblies)
- Per-sample table of reconstructed influenza segment lengths and contig counts at each depth
- Fig. 4-style comparison of VirMAP versus standard-approach segment lengths across subsampling depths
- Segment statistics CSV (e.g., influenza_virmap_results.csv extended with depth-wise results)

## How to apply

Start from the full-cohort FASTQ set (here 25 paired-end influenza metagenome samples from BioProject PRJEB7888 totaling ~1.75 billion reads, downloaded with fasterq-dump). Subsample each sample (or the cohort) to a series of lower read depths, then rerun the full reconstruction pipeline at each depth — for VirMAP this means Virmap.pl with --useMegahit --useBbnorm and all available threads, provisioning at least 64 GB RAM per simultaneous instance. Run the baseline 'standard approach' reconstruction under identical subsampled depths so the comparison is depth-matched. For each influenza-positive sample, extract reconstructed influenza segments from the VirMAP super-scaffold output and record segment length and contig count; VirMAP's key property is that each segment is a single contig. Plot or tabulate segment lengths versus subsampling depth for both methods to reproduce the Fig. 4-style comparison showing how each method degrades as coverage drops.

## Related tools

- **VirMAP** (Reconstruction method under benchmarking; builds virus-like super-scaffolds from nucleotide and translated alignments, with --useMegahit --useBbnorm enabled) — https://github.com/cmmr/VirMAP
- **MEGAHIT** (De-novo assembler invoked by VirMAP (--useMegahit) during reconstruction at each subsampled depth) — https://github.com/voutcn/megahit
- **BBMap** (Used within VirMAP (with --useBbnorm) for read normalization and mapping assembly of subsampled reads) — https://github.com/cmmr/VirMAP
- **DIAMOND** (Translated protein alignment backend (virDmnd.dmnd) used to recruit reads at each depth) — https://github.com/bbuchfink/diamond
- **Tadpole** (Alternative assembler within VirMAP's multi-kmer de-novo assembly strategy)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- At full depth, VirMAP reconstruction of the 20 influenza-positive UMERS samples averages 9.05 influenza segments per sample — the benchmark run should reproduce this before subsampling.
- Every reconstructed influenza segment consists of exactly one contig (contiguous reconstruction) across depths; appearance of multi-contig segments flags a pipeline or parameter problem.
- Segment-length-versus-depth curves for VirMAP and the standard approach are depth-matched (same subsampled read counts per depth) and comparable per sample.
- Total recruited influenza reads at full depth match the reported 509,039 (193% more than the standard approach), validating the read-recruitment step feeding the comparison.
- Per-sample segment tables are complete: 20 influenza-positive samples each with 8 influenza genome segments tallied at every subsampling depth.

## Limitations

- No software versions stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole, and no Genbank database snapshot/version or download date specified, so exact reproduction of depth-comparison curves depends on tooling versions.
- VirMAP requires at least 64 GB RAM per simultaneous instance and the Genbank database grows over time (minimum SSD/RAM requirements expand), constraining how many subsampled depths can be run in parallel.
- Some tunable parameters are only documented externally (Supplementary Data 14 sensitivity/specificity tradeoff options; VirMAP_program_usage_and_options_table.xlsx on GitHub), so benchmark results can vary with unreported option settings.

## Evidence

- [other] Subsample the 1.75 billion reads to lower depths, rerun reconstruction, and compare VirMAP versus standard-approach segment lengths to reproduce the Fig. 4 comparison.: [evidence span withheld — anchor status: not_found]
- [discussion] VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs.: "VirMAP employs both de-novo and mapping assembly strategies to create a working set of contigs."
- [discussion] VirMAP is implemented in Perl.: "The VirMAP software is open-source and available online at https://github.com/cmmr/VirMAP. VirMAP is implemented in Perl."
- [methods] Virmap.pl command with useMegahit and useBbnorm flags.: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [discussion] Influenza dataset BioProject PRJEB7888 is the benchmark dataset.: "Influenza dataset, BioProject ID PRJEB7888"
