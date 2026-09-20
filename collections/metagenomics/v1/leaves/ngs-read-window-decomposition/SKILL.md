---
name: ngs-read-window-decomposition
description: Use when you have a sorted BAM/CRAM (or FASTA MSA) of reads from a genetically
  heterogeneous viral sample and need error-corrected reads plus local haplotype reconstruction
  before global haplotype assembly or SNV calling.
when_to_use_negative:
- You need full-length global haplotype reconstruction in ShoRAH2 — global mode is
  disabled there; use ShoRAH v1.1.3 or V-pipe's global haplotype options instead
- The sample is not genetically heterogeneous (single-clone data) or coverage is too
  low to support window clustering — clustering-based error correction needs deep
  coverage (e.g. ~5000x to detect 0.1%-frequency haplotypes)
- You only need variant frequencies from an existing curated variant list rather than
  de novo error correction of raw reads
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0797
tools:
- name: ShoRAH
  role: Package providing the windowing, Dirichlet-process error correction, and local
    haplotype reconstruction pipeline
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah shotgun
  role: Entry point that runs the local (window-based) analysis for shotgun sequencing
    data
  repo: https://github.com/cbg-ethz/shorah
- name: shorah amplicon
  role: Entry point for the window-based local analysis on amplicon sequencing data
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler for error correction via Dirichlet process mixture, run on each
    genomic window
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun sequencing .BAM input into multiple overlapping windows
  repo: https://github.com/cbg-ethz/shorah
- name: shorah snv
  role: Called after windowing/sampling to detect SNVs, taking strand bias into account
  repo: https://github.com/cbg-ethz/shorah
- name: shorah.py
  role: Wrapper invoking the four major steps (alignment, error correction, global
    reconstruction, frequency estimation)
  repo: https://github.com/cbg-ethz/shorah
- name: V-pipe
  role: Recommended workflow for global haplotype reconstruction when ShoRAH2 global
    mode is unavailable
  repo: https://github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:2398e4fa9afd6e8d85663878acd8c8cb3e904e77f21602d94a86c5b6012b014d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ShoRAH
  - V-pipe
  - b2w
  - diri_sampler
  - shorah amplicon
  - shorah shotgun
  - shorah snv
  - shorah.py
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# ngs-read-window-decomposition

## Summary

Decompose a deep-sequencing read alignment into overlapping genomic windows (~one read length wide, each position covered ~3 times) and run a Dirichlet-process Gibbs sampler (diri_sampler) per window to correct sequencing errors and reconstruct local haplotypes with frequencies in a mixed sample.

## When to use

Use when you have a sorted BAM/CRAM (or FASTA MSA) of reads from a genetically heterogeneous viral sample and need error-corrected reads plus local haplotype reconstruction before global haplotype assembly or SNV calling.

## When NOT to use

- You need full-length global haplotype reconstruction in ShoRAH2 — global mode is disabled there; use ShoRAH v1.1.3 or V-pipe's global haplotype options instead
- The sample is not genetically heterogeneous (single-clone data) or coverage is too low to support window clustering — clustering-based error correction needs deep coverage (e.g. ~5000x to detect 0.1%-frequency haplotypes)
- You only need variant frequencies from an existing curated variant list rather than de novo error correction of raw reads

## Inputs

- Sorted BAM or CRAM file of reads aligned to the reference (e.g. test_aln.cram)
- Multiple sequence alignment of reads in FASTA format (alternative input)
- Reference sequence of the amplified/genomic region
- Window/sampler parameters: window width (~average read length), number of Gibbs-sampler iterations, Dirichlet-process hyperparameter alpha

## Outputs

- corrected_reads.fa — error-corrected reads in FASTA format
- Per-window local haplotype reconstructions with inferred cluster frequencies
- SNV calls (SNVs_0.010000.txt / SNV.txt) when shorah snv is invoked
- Support files (e.g. .w.log, .freqs, .reads files) per window

## How to apply

Load the input read alignment (BAM/CRAM or multiple sequence alignment in FASTA) and use b2w/shorah.py to cut overlapping windows on the MSA with width approximately the average read length, such that each alignment position is covered three times. Invoke diri_sampler on each window; it uses a Dirichlet process mixture Gibbs sampler to cluster unique reads, correct sequencing errors, and infer local haplotypes with frequencies in a Bayesian fashion — set the number of iterations and the hyperparameter alpha large enough to guarantee proper mixing of the sampler. Apply each cluster consensus back to the original reads by majority vote across the overlapping window reconstructions, writing corrected reads to corrected_reads.fa and per-window local haplotype reconstructions to disk. The local analysis alone can be run invoking `shorah shotgun` (shotgun data) or `shorah amplicon` (amplicon data), which also calls `shorah snv` for SNV calling with strand-bias correction. Validation: at high coverage the paper reports a five-fold decrease of the error rate, from 0.25% to 0.05%.

## Related tools

- **ShoRAH** (Package providing the windowing, Dirichlet-process error correction, and local haplotype reconstruction pipeline) — http://www.cbg.ethz.ch/software/shorah
- **shorah shotgun** (Entry point that runs the local (window-based) analysis for shotgun sequencing data) — https://github.com/cbg-ethz/shorah
- **shorah amplicon** (Entry point for the window-based local analysis on amplicon sequencing data) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler for error correction via Dirichlet process mixture, run on each genomic window) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun sequencing .BAM input into multiple overlapping windows) — https://github.com/cbg-ethz/shorah
- **shorah snv** (Called after windowing/sampling to detect SNVs, taking strand bias into account) — https://github.com/cbg-ethz/shorah
- **shorah.py** (Wrapper invoking the four major steps (alignment, error correction, global reconstruction, frequency estimation)) — https://github.com/cbg-ethz/shorah
- **V-pipe** (Recommended workflow for global haplotype reconstruction when ShoRAH2 global mode is unavailable) — https://github.com/cbg-ethz/V-pipe

## Examples

```
shorah shotgun
```

## Evaluation signals

- corrected_reads.fa exists and is valid FASTA containing the corrected reads
- Error rate of corrected reads is substantially reduced relative to raw reads (paper benchmark: five-fold decrease from 0.25% to 0.05% at high coverage)
- Each alignment position is covered by three overlapping windows of width ~average read length
- The Dirichlet-process Gibbs sampler shows proper mixing (iterations and alpha set large enough; check window .w.log outputs)
- Number of reads assigned to each cluster yields haplotype frequencies consistent with known mixtures on simulated HIV-1 pol data

## Limitations

- ShoRAH2 has global haplotype reconstruction disabled; the last version with global mode is v1.1.3, otherwise use V-pipe's global haplotype options
- Not packaged for Windows — tested on Linux and Mac OS X; Windows users need WSL or Cygwin
- SAMtools and pysam are required but not included in the package and must be obtained separately
- Error correction is computationally heavy: less than 12 hours using 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads
- No pre-set version stated for the main description and dependency versions (Biopython, GSL, SAMtools, pysam) not specified

## Evidence

- [intro] This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [intro] The number of reads associated with the cluster estimates the prevalence of the haplotype in the population: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [readme] they work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling: "They work by cutting window from the multiple sequence
alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the
SNV calling."
- [readme] The last version of ShoRAH1 with global mode is v1.1.3: "Currently in ShoRAH2 global haplotype reconstruction is disabled.
The last version of ShoRAH1 with global mode is
[v1.1.3]"
