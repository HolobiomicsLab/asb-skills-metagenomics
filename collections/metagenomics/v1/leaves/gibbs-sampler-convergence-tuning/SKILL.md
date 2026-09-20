---
name: gibbs-sampler-convergence-tuning
description: Use when running diri_sampler-based error correction on overlapping windows
  of a read alignment (BAM/CRAM or MSA) and you must set sampler iterations and the
  Dirichlet hyperparameter alpha to achieve proper mixing for haplotype reconstruction.
when_to_use_negative:
- Input is raw unaligned FASTQ reads — reads must first be aligned to the reference
  sequence of the amplified region before windowing
- You need global haplotype reconstruction from ShoRAH2 — global mode is disabled
  there; use ShoRAH v1.1.3 or the global haplotype options in V-pipe instead
- The task is strand-bias-aware SNV calling only and you will not tune or inspect
  sampler convergence — invoke shorah snv directly rather than re-tuning diri_sampler
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: diri_sampler
  role: Dirichlet process mixture Gibbs sampler that clusters unique reads per window,
    corrects sequencing errors, and infers local haplotypes with frequencies
  repo: github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: Wrapper that cuts windows from the multiple sequence alignment, invokes diri_sampler
    on the windows, and calls shorah snv for SNV calling
  repo: github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun sequencing BAM input into multiple overlapping windows fed
    to the sampler
  repo: github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:67bab5aab2f182887c0f8e35f0b41770c28b929ff779be1eb568ed49de3cfa94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ShoRAH
  - b2w
  - diri_sampler
  - shorah shotgun
  - shorah.py
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# gibbs-sampler-convergence-tuning

## Summary

Tuning skill for the Dirichlet-process mixture Gibbs sampler (diri_sampler) in ShoRAH, which clusters reads overlapping the same genomic window to correct sequencing errors and infer local haplotypes. Proper setting of the number of iterations and the hyperparameter alpha is required so the sampler mixes and cluster frequencies are reliable.

## When to use

Use when running diri_sampler-based error correction on overlapping windows of a read alignment (BAM/CRAM or MSA) and you must set sampler iterations and the Dirichlet hyperparameter alpha to achieve proper mixing for haplotype reconstruction.

## When NOT to use

- Input is raw unaligned FASTQ reads — reads must first be aligned to the reference sequence of the amplified region before windowing
- You need global haplotype reconstruction from ShoRAH2 — global mode is disabled there; use ShoRAH v1.1.3 or the global haplotype options in V-pipe instead
- The task is strand-bias-aware SNV calling only and you will not tune or inspect sampler convergence — invoke shorah snv directly rather than re-tuning diri_sampler

## Inputs

- Sorted BAM/CRAM alignment of reads against a reference (e.g. test_aln.cram)
- Multiple sequence alignment of reads (used to cut overlapping windows)

## Outputs

- corrected_reads.fa — error-corrected reads in FASTA format
- Per-window local haplotype reconstructions with inferred frequencies
- Window files produced by b2w splitting of the alignment

## How to apply

Construct overlapping windows on the multiple sequence alignment such that each alignment position is covered three times, with window width approximately the average read length. On each window, run diri_sampler, which uses a Dirichlet process mixture Gibbs sampler to cluster unique reads, correct sequencing errors, and infer local haplotypes with frequencies in a Bayesian fashion. Set the number of iterations and the hyperparameter alpha large enough to guarantee proper mixing of the sampler — this is the key convergence decision point. The consensus sequence of each read cluster represents the original haplotype, and the number of reads associated with the cluster estimates that haplotype's prevalence in the population. Apply the cluster consensus to the original reads by majority vote across the overlapping window reconstructions to produce corrected reads.

## Related tools

- **diri_sampler** (Dirichlet process mixture Gibbs sampler that clusters unique reads per window, corrects sequencing errors, and infers local haplotypes with frequencies) — github.com/cbg-ethz/shorah
- **shorah shotgun** (Wrapper that cuts windows from the multiple sequence alignment, invokes diri_sampler on the windows, and calls shorah snv for SNV calling) — github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun sequencing BAM input into multiple overlapping windows fed to the sampler) — github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun -b test_aln.cram -f reference.fa
```

## Evaluation signals

- corrected_reads.fa exists and is valid FASTA after the run
- Error rate of corrected reads is substantially reduced relative to raw reads — the paper reports a five-fold decrease from 0.25% to 0.05% at high coverage
- Every alignment position is covered by three windows, with window width approximately the average read length
- Sampler shows proper mixing: cluster assignments and inferred haplotype frequencies stabilize across Gibbs iterations when iterations and alpha are set large enough
- Number of reads per cluster yields prevalence estimates that reproduce known haplotype frequencies (e.g. haplotypes as rare as 0.1% detected at coverage 5000)

## Limitations

- Convergence depends on the number of iterations and hyperparameter alpha; setting them too small risks improper mixing of the Dirichlet process sampler
- Error correction is computationally heavy: less than 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads
- Global haplotype reconstruction is disabled in ShoRAH2; the last version with global mode is v1.1.3
- Software has been tested on Linux and Mac OS X platforms only (Windows users need WSL or Cygwin)

## Evidence

- [intro] Correct sequencing errors by clustering reads that overlap the same genome window: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [readme] diri_sampler performs Gibbs sampling for error correction via Dirichlet process mixture: "`diri_sampler` | Gibbs sampling for error correction via Dirichlet process mixture"
- [readme] Local analysis cuts windows and invokes diri_sampler: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling."
- [intro] Cluster size estimates haplotype prevalence: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] Five-fold error rate decrease achieved: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
