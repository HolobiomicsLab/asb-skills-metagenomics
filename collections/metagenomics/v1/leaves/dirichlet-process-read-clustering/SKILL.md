---
name: dirichlet-process-read-clustering
description: Use when you have an alignment of short NGS reads (BAM/CRAM or FASTA
  MSA) from a genetically heterogeneous sample and need error correction plus local
  haplotype reconstruction before global assembly or frequency estimation.
when_to_use_negative:
- You already need global (full-length) haplotype reconstruction rather than local
  windowed reconstruction — ShoRAH2 disables global mode; use v1.1.3 or V-pipe global
  options instead
- The input is unassembled raw reads not yet aligned to a reference — alignment is
  the prerequisite step 1 of the ShoRAH pipeline
- You only need single nucleotide variant detection with strand-bias accounting —
  invoke shorah snv directly rather than the full clustering stage
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: ShoRAH
  role: Package providing the error-correction/local-haplotype-reconstruction workflow
    this skill implements
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Core Gibbs sampler that clusters reads per window via Dirichlet process mixture
    and infers local haplotypes with frequencies
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun sequencing BAM into multiple overlapping windows before sampling
  repo: https://github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: Wrapper invoking windowing plus diri_sampler for shotgun sequencing analysis
  repo: https://github.com/cbg-ethz/shorah
- name: shorah amplicon
  role: Wrapper for the same windowed clustering in amplicon-based analysis
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:e597caeb760fab96faab6845469aa797e6c31e4be44e7affad4addb082dd5dfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - ShoRAH
  - b2w
  - diri_sampler
  - shorah amplicon
  - shorah shotgun
  - shorah.py
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# dirichlet-process-read-clustering

## Summary

Corrects sequencing errors and reconstructs local haplotypes from deep-sequencing reads of a mixed viral sample by clustering reads that overlap the same genomic window with a Dirichlet-process mixture Gibbs sampler (ShoRAH's diri_sampler). Each cluster consensus represents the original haplotype, and the read count per cluster estimates that haplotype's prevalence.

## When to use

Use when you have an alignment of short NGS reads (BAM/CRAM or FASTA MSA) from a genetically heterogeneous sample and need error correction plus local haplotype reconstruction before global assembly or frequency estimation.

## When NOT to use

- You already need global (full-length) haplotype reconstruction rather than local windowed reconstruction — ShoRAH2 disables global mode; use v1.1.3 or V-pipe global options instead
- The input is unassembled raw reads not yet aligned to a reference — alignment is the prerequisite step 1 of the ShoRAH pipeline
- You only need single nucleotide variant detection with strand-bias accounting — invoke shorah snv directly rather than the full clustering stage

## Inputs

- Sorted BAM/CRAM alignment of shotgun or amplicon reads (e.g. test_aln.cram), or a multiple sequence alignment of reads in FASTA format
- Reference sequence of the amplified region for read alignment
- Config parameters: window width (~average read length), number of Gibbs sampler iterations, Dirichlet process hyperparameter alpha

## Outputs

- corrected_reads.fa — error-corrected reads in FASTA format
- Per-window local haplotype reconstructions with inferred cluster frequencies
- Support/SNV output files consumed by downstream shorah snv calling (e.g. SNVs_0.010000.txt)

## How to apply

Load the input read alignment and use b2w (via shorah shotgun or shorah amplicon) to cut the multiple sequence alignment into overlapping windows so that each alignment position is covered multiple times, with window width approximately equal to the average read length. Invoke diri_sampler on each window: it runs a Dirichlet process mixture Gibbs sampler that clusters unique reads, corrects sequencing errors, and infers local haplotypes with frequencies in a Bayesian fashion; set the number of iterations and the hyperparameter alpha large enough to guarantee proper mixing of the sampler. Apply each cluster consensus back to the original reads, correcting them by majority vote across the overlapping window reconstructions, and write the corrected reads to corrected_reads.fa along with per-window local haplotype reconstructions. In the ShoRAH workflow this is step 2 of four (alignment; error correction/local haplotype reconstruction; global haplotype reconstruction; frequency estimation) and is orchestrated by the shorah.py wrapper. Rationale: deep sequencing can quantify genetic diversity in a mixed sample only if sequencing errors are properly treated; at high coverage the reported correction achieves a five-fold error-rate decrease from 0.25% to 0.05%.

## Related tools

- **ShoRAH** (Package providing the error-correction/local-haplotype-reconstruction workflow this skill implements) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Core Gibbs sampler that clusters reads per window via Dirichlet process mixture and infers local haplotypes with frequencies) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun sequencing BAM into multiple overlapping windows before sampling) — https://github.com/cbg-ethz/shorah
- **shorah shotgun** (Wrapper invoking windowing plus diri_sampler for shotgun sequencing analysis) — https://github.com/cbg-ethz/shorah
- **shorah amplicon** (Wrapper for the same windowed clustering in amplicon-based analysis) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun -b test_aln.cram -f ref.fasta
```

## Evaluation signals

- corrected_reads.fa exists and parses as valid FASTA
- Error rate of corrected reads is substantially reduced relative to raw reads — reference benchmark is a five-fold decrease from 0.25% to 0.05%
- Every alignment position is covered by multiple overlapping windows (window width ≈ average read length)
- Number of reads per cluster is a plausible estimate of that haplotype's prevalence; haplotypes at ≥0.1% frequency should be detectable at coverage 5000
- Sampler converged: iterations and alpha were set large enough for proper mixing (benchmark runtime: <12 h on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads)

## Limitations

- Error correction is compute-intensive — less than 12 hours using 6 CPUs at 2.8 GHz was needed for 40,000 pyrosequencing reads
- Gibbs sampler requires enough iterations and a sufficiently large alpha hyperparameter, otherwise mixing is inadequate
- ShoRAH2 has global haplotype reconstruction disabled; only the local windowed analysis is available in current versions (last global-capable release is v1.1.3)
- Requires external dependencies (HTSlib, zlib, Boost, Biopython, NumPy) and must be compiled or installed via bioconda

## Evidence

- [intro] This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [intro] The number of reads associated with the cluster estimates the prevalence of the haplotype in the population: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] They work by cutting window from the multiple sequence alignment, invoking diri_sampler on the windows and calling shorah snv for the SNV calling.: "They work by cutting window from the multiple sequence alignment, invoking diri_sampler on the windows and calling shorah snv for the SNV calling."
