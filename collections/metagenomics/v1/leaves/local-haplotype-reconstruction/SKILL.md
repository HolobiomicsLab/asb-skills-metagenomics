---
name: local-haplotype-reconstruction
description: Use when deep-sequencing reads (sorted BAM/CRAM or a read MSA) from a
  genetically heterogeneous mixed sample need sequencing-error correction and local
  (window-level) haplotype reconstruction before global assembly or frequency estimation.
when_to_use_negative:
- You need full-length (global) haplotype reconstruction with ShoRAH2 — global mode
  is disabled there; use ShoRAH v1.1.3 or the V-pipe global haplotype options instead.
- You only want single nucleotide variant detection with strand bias — invoke 'shorah
  snv' directly rather than the full windowed reconstruction.
- Input is a low-coverage sample where clustering cannot resolve rare variants — sensitivity
  of 0.1% frequency detection was demonstrated at coverage 5000.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: shorah shotgun
  role: 'wrapper that runs the local analysis: cuts windows from the MSA, invokes
    diri_sampler, and calls shorah snv'
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler for error correction via Dirichlet process mixture, clustering
    unique reads per window and inferring local haplotypes with frequencies
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: splits shotgun sequencing .BAM files into multiple overlapping windows
  repo: https://github.com/cbg-ethz/shorah
- name: shorah amplicon
  role: amplicon-based variant of the local analysis workflow
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:fa2eaed366871cd8da82c75bb15c87c879114bd4fef87795dcb72a7901e06a7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - Python
  - ShoRAH
  - b2w
  - diri_sampler
  - freqEst
  - mm.py
  - shorah amplicon
  - shorah shotgun
  - shorah.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# local-haplotype-reconstruction

## Summary

ShoRAH's error-correction/local-haplotype stage: overlapping windows cut from the read MSA are clustered by a Dirichlet-process Gibbs sampler (diri_sampler) so that each cluster consensus recovers a local haplotype and its prevalence from reads drawn from a genetically heterogeneous (e.g., HIV-1) mixed sample.

## When to use

Use when deep-sequencing reads (sorted BAM/CRAM or a read MSA) from a genetically heterogeneous mixed sample need sequencing-error correction and local (window-level) haplotype reconstruction before global assembly or frequency estimation.

## When NOT to use

- You need full-length (global) haplotype reconstruction with ShoRAH2 — global mode is disabled there; use ShoRAH v1.1.3 or the V-pipe global haplotype options instead.
- You only want single nucleotide variant detection with strand bias — invoke 'shorah snv' directly rather than the full windowed reconstruction.
- Input is a low-coverage sample where clustering cannot resolve rare variants — sensitivity of 0.1% frequency detection was demonstrated at coverage 5000.

## Inputs

- sorted BAM or CRAM alignment of shotgun reads (e.g., test_aln.cram)
- multiple sequence alignment (MSA) of reads (FASTA)
- reference sequence (e.g., HIV-1 subtype B pol gene)

## Outputs

- corrected_reads.fa (error-corrected reads in FASTA format)
- per-window local haplotype reconstructions with inferred frequencies
- SNV calls from shorah snv (e.g., SNVs_0.010000.txt)

## How to apply

Load a sorted input alignment (BAM/CRAM, e.g. test_aln.cram) or a multiple sequence alignment of reads; use b2w/dec.py to cut overlapping windows of width approximately the average read length such that each alignment position is covered three times. Run diri_sampler on each window: a Gibbs sampler with a Dirichlet process mixture model clusters unique reads, corrects errors and infers local haplotypes with frequencies in a Bayesian fashion — set the number of iterations and hyperparameter alpha large enough to guarantee proper mixing of the sampler. Apply the cluster consensus to the original reads by majority vote across the overlapping window reconstructions, writing corrected reads to corrected_reads.fa and per-window local haplotypes to disk. Invoke the whole local analysis via the 'shorah shotgun' (shotgun data) or 'shorah amplicon' (amplicon data) wrapper, which also calls shorah snv for SNV calling. On simulated Illumina data (10 HIV-1 subtype B pol variants), this detected haplotypes down to 0.1% frequency at coverage 5000.

## Related tools

- **shorah shotgun** (wrapper that runs the local analysis: cuts windows from the MSA, invokes diri_sampler, and calls shorah snv) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler for error correction via Dirichlet process mixture, clustering unique reads per window and inferring local haplotypes with frequencies) — https://github.com/cbg-ethz/shorah
- **b2w** (splits shotgun sequencing .BAM files into multiple overlapping windows) — https://github.com/cbg-ethz/shorah
- **shorah amplicon** (amplicon-based variant of the local analysis workflow) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- corrected_reads.fa exists and is valid FASTA after the run
- Error rate of corrected reads is substantially reduced relative to raw reads — expected about five-fold (0.25% to 0.05%)
- Each alignment position is covered three times by the window construction, with window width approximately the average read length
- Haplotypes at frequencies as low as 0.1% are detected and their frequencies correctly estimated at coverage 5000 (simulated 10-variant HIV-1 pol mixture)
- diri_sampler mixing is adequate: number of iterations and hyperparameter alpha were set large enough for proper mixing of the Gibbs sampler

## Limitations

- Error correction of 40,000 pyrosequencing reads took less than 12 hours on 6 CPUs at 2.8 GHz — compute cost scales with read count and sampler iterations.
- ShoRAH2 has global haplotype reconstruction disabled; only the local mode described here (or ShoRAH v1.1.3 / V-pipe) supports the global steps.
- Tested on Linux and Mac OS X only; Windows requires WSL or Cygwin, and dependencies (HTSlib, zlib, Boost, Biopython, NumPy, and for the git version Autoconf/Automake) must be resolved — bioconda install is recommended.
- Requires per-window runtime tuning of iterations and alpha; too-small values risk improper mixing of the Dirichlet-process sampler.

## Evidence

- [intro] corrects sequencing errors by clustering reads that overlap the same genome window: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [methods] 0.1% frequency detection at coverage 5000: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [methods] five-fold error-rate decrease: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] runtime cost of error correction: "the error correction step was completed in less than 12 hours using 6 CPUs at 2.8 GHz for a large sample consisting of 40,000 pyrosequencing reads"
- [readme] README on how shorah shotgun/amplicon run the local analysis: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling."
- [readme] global mode disabled in ShoRAH2: "Currently in ShoRAH2 global haplotype reconstruction is disabled. The last version of ShoRAH1 with global mode is [v1.1.3]"
