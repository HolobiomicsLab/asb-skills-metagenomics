---
name: multiple-sequence-alignment-construction-from-ngs-reads
description: Use when deep-sequencing reads from a mixed sample (shotgun or amplicon,
  e.g. 454 Titanium pyrosequencing or Illumina viral data) must be placed in a common
  reference coordinate frame before windowed error correction or haplotype reconstruction.
when_to_use_negative:
- The reads are already assembled in a gap-padded MSA against the reference, so re-aligning
  would only duplicate ShoRAH's first step.
- The goal is global (full-length) haplotype reconstruction with current ShoRAH2,
  where global mode is disabled — instead use ShoRAH1 v1.1.3 or the V-pipe global
  haplotype options.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ShoRAH
  role: Package whose first of four major steps (alignment) builds the MSA of reads
    against the reference before error correction and frequency estimation
  repo: github.com/cbg-ethz/shorah
- name: shorah.py
  role: Wrapper called to perform the full four-step analysis, including the alignment
    step that constructs the MSA
  repo: github.com/cbg-ethz/shorah
- name: b2w
  role: Splits the shotgun-sequencing .BAM-derived MSA into the multiple overlapping
    windows consumed by diri_sampler
  repo: github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: Entry point that runs the local (windowed) analysis on shotgun data after
    alignment
  repo: github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:9c6c84c2bb0ebabff1aac5024e6913aebfdad096fbe12b2963a3a5729bcdd09b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - b2w
  - shorah shotgun
  - shorah.py
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# multiple-sequence-alignment-construction-from-ngs-reads

## Summary

Builds a multiple sequence alignment (MSA) of error-prone NGS reads from a genetically heterogeneous mixed sample (e.g., 454 Titanium pyrosequencing reads of HIV-1 subtype B pol) against a reference sequence, putting all reads in a common coordinate frame as the first step of ShoRAH's local haplotype reconstruction pipeline.

## When to use

Use when deep-sequencing reads from a mixed sample (shotgun or amplicon, e.g. 454 Titanium pyrosequencing or Illumina viral data) must be placed in a common reference coordinate frame before windowed error correction or haplotype reconstruction.

## When NOT to use

- The reads are already assembled in a gap-padded MSA against the reference, so re-aligning would only duplicate ShoRAH's first step.
- The goal is global (full-length) haplotype reconstruction with current ShoRAH2, where global mode is disabled — instead use ShoRAH1 v1.1.3 or the V-pipe global haplotype options.

## Inputs

- sorted BAM (or CRAM/SAM) file of NGS reads mapped to a reference (e.g., test_aln.cram)
- reference sequence of the amplified/sequenced region (e.g., HIV-1 subtype B pol gene)
- shorah.py config specifying bamfile and snvsfile paths

## Outputs

- multiple sequence alignment of reads in reference coordinates (built by pairwise alignment plus gap padding)
- overlapping windows of the MSA, each covering every alignment position three times, with width approximately equal to the average read length

## How to apply

Align the reads pairwise to the reference sequence of the amplified or sequenced region (in the ShoRAH workflow, the HIV-1 subtype B pol reference), using the alignment step invoked through the shorah.py wrapper as the first of the four major steps (alignment, error correction, global haplotype reconstruction, frequency estimation). Provide the input as a sorted BAM file; ShoRAH's alignment dependencies handle BAM/CRAM/SAM access via HTSlib (with SAMtools/pysam historically required externally). Construct the MSA by gap padding so that all reads share reference-genome coordinates. Then split the MSA into overlapping windows with b2w so that each alignment position is covered three times, choosing a window width approximately equal to the average read length (e.g. the 454 Titanium read length), because downstream diri_sampler clustering operates per window. Run the local analysis with `shorah shotgun` or `shorah amplicon` depending on whether the protocol was shotgun or amplicon based.

## Related tools

- **ShoRAH** (Package whose first of four major steps (alignment) builds the MSA of reads against the reference before error correction and frequency estimation) — github.com/cbg-ethz/shorah
- **shorah.py** (Wrapper called to perform the full four-step analysis, including the alignment step that constructs the MSA) — github.com/cbg-ethz/shorah
- **b2w** (Splits the shotgun-sequencing .BAM-derived MSA into the multiple overlapping windows consumed by diri_sampler) — github.com/cbg-ethz/shorah
- **shorah shotgun** (Entry point that runs the local (windowed) analysis on shotgun data after alignment) — github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- The MSA covers each alignment position three times via overlapping windows, with window width approximately equal to the average read length (e.g., 454 Titanium read length).
- All reads sit in a common reference coordinate frame after gap padding, so reads aligned to the HIV-1 subtype B pol reference share positions consistently.
- The pipeline was given a valid sorted BAM/CRAM input (e.g., test_aln.cram) and `shorah shotgun` completed, producing window files for diri_sampler.
- Downstream error correction on this alignment achieves the expected five-fold error-rate decrease from 0.25% to 0.05% at coverage around 6000.
- Haplotypes with frequencies as low as 0.1% are detected at coverage ~5000 with correct frequency estimates, indicating the alignment supports sensitive local reconstruction.

## Limitations

- Only tested on Linux and Mac OS X platforms; Windows users must rely on WSL or Cygwin.
- The git tree contains only bootstrapping files — versioned tarballs are recommended, and bootstrapping requires Autoconf 2.69, Automake 1.15, and m4.
- Some dependencies (historically SAMtools and pysam; currently HTSlib, zlib, Boost, Biopython, NumPy) are not bundled and must be obtained separately.
- Sensitivity of downstream clustering depends on coverage: 0.1% clones required coverage around 5000-6000 reads.

## Evidence

- [methods] four major steps of the ShoRAH pipeline: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [intro] alignment to the amplified-region reference: "The reads can be aligned to the reference sequence of the amplified region"
- [readme] BAM splitting into windows: "b2w | splitting shotgun sequencing .BAM into multiple overlapping windows"
- [readme] sorted BAM input requirement: "The input is a sorted bam file. Analysis can be performed in local mode."
- [methods] shorah.py wrapper for the full analysis: "Full analysis with ShoRAH includes four major steps that are performed by calling the wrapper shorah.py"
