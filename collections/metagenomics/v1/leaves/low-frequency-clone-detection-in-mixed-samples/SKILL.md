---
name: low-frequency-clone-detection-in-mixed-samples
description: Use when quantifying genetic diversity of a mixed NGS sample (e.g. 454
  Titanium reads from an HIV-1 pol amplicon mixture) and you need to recover individual
  haplotypes/clone frequencies as low as 0.1% despite error-prone reads.
when_to_use_negative:
- Input reads have already been error-corrected or come from a low-error platform
  where residual error rate is below the rare-variant frequency of interest — correction
  by majority vote could erase true minor variants
- Coverage is far below ~5000x, since the article reports 0.1% haplotypes detected
  at coverage of 5000; lower coverage will not support rare-clone inference
- The goal is global (full-length) haplotype reconstruction with current ShoRAH2 —
  global mode is disabled and only ShoRAH1 v1.1.3 or V-pipe options support it
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: ShoRAH
  role: Software suite providing error correction, local haplotype reconstruction
    and frequency estimation for mixed samples
  repo: http://www.cbg.ethz.ch/software/shorah
- name: diri_sampler
  role: Gibbs sampling for error correction via Dirichlet process mixture; clusters
    reads within each window so cluster sizes estimate haplotype prevalence
  repo: https://github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: 'Runs the local analysis on shotgun data: cuts windows from the MSA, invokes
    diri_sampler, and calls shorah snv'
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun sequencing .BAM into multiple overlapping windows prior to
    clustering
  repo: https://github.com/cbg-ethz/shorah
- name: shorah.py
  role: Wrapper invoking the four major ShoRAH steps (alignment, error correction,
    global haplotype reconstruction, frequency estimation)
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:e39323d3bf07f87e223f0c7fdad1b4e028a3f8435335481efe4ff48081525df4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - b2w
  - diri_sampler
  - shorah shotgun
  - shorah.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# low-frequency-clone-detection-in-mixed-samples

## Summary

Detect and quantify rare clones in a genetically heterogeneous deep-sequencing sample by using ShoRAH's model-based error correction (Dirichlet process mixture clustering of overlapping read windows), which removes the sequencing noise that otherwise masks variants present at ~0.1% frequency.

## When to use

Use when quantifying genetic diversity of a mixed NGS sample (e.g. 454 Titanium reads from an HIV-1 pol amplicon mixture) and you need to recover individual haplotypes/clone frequencies as low as 0.1% despite error-prone reads.

## When NOT to use

- Input reads have already been error-corrected or come from a low-error platform where residual error rate is below the rare-variant frequency of interest — correction by majority vote could erase true minor variants
- Coverage is far below ~5000x, since the article reports 0.1% haplotypes detected at coverage of 5000; lower coverage will not support rare-clone inference
- The goal is global (full-length) haplotype reconstruction with current ShoRAH2 — global mode is disabled and only ShoRAH1 v1.1.3 or V-pipe options support it

## Inputs

- Sorted BAM/CRAM file of deep-sequencing reads from a mixed sample (e.g. 454 Titanium pyrosequencing reads)
- Reference sequence of the amplified region (e.g. HIV-1 subtype B pol gene)
- ShoRAH configuration: number of Gibbs-sampling iterations and Dirichlet-process hyperparameter alpha, window width (~average read length), coverage of each position three times

## Outputs

- Corrected reads with reduced sequencing error rate (target: 0.25% -> 0.05%, a five-fold decrease)
- Local haplotype reconstructions per window with cluster-associated read counts as frequency estimates
- Inferred clone frequencies, including clones as rare as 0.1%

## How to apply

First align the reads to the reference sequence of the amplified region (e.g. HIV-1 subtype B pol), typically by pairwise alignment to the reference followed by MSA construction via gap padding. Split the alignment into overlapping windows covering each position three times, with window width approximately equal to the average read length (b2w / dec.py). Run diri_sampler on each window: a Gibbs-sampling Dirichlet process mixture that clusters reads overlapping the same genome region; set the number of iterations and the hyperparameter alpha large enough to guarantee proper mixing so that solutions with new clusters are proposed. Read counts per cluster estimate haplotype prevalence, and corrected reads are derived by majority vote from local haplotype reconstructions across overlapping windows. Validate on a mixture of known clones by checking that the residual error rate drops about five-fold (from 0.25% to 0.05%) and that clones at frequencies as low as 0.1% are recovered with correct frequency estimates at coverage around 5000-6000.

## Related tools

- **ShoRAH** (Software suite providing error correction, local haplotype reconstruction and frequency estimation for mixed samples) — http://www.cbg.ethz.ch/software/shorah
- **diri_sampler** (Gibbs sampling for error correction via Dirichlet process mixture; clusters reads within each window so cluster sizes estimate haplotype prevalence) — https://github.com/cbg-ethz/shorah
- **shorah shotgun** (Runs the local analysis on shotgun data: cuts windows from the MSA, invokes diri_sampler, and calls shorah snv) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun sequencing .BAM into multiple overlapping windows prior to clustering) — https://github.com/cbg-ethz/shorah
- **shorah.py** (Wrapper invoking the four major ShoRAH steps (alignment, error correction, global haplotype reconstruction, frequency estimation)) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun -b test_aln.cram -r reference.fasta
```

## Evaluation signals

- Residual sequencing error rate of corrected reads decreases about five-fold, from 0.25% to 0.05%
- Clones with frequencies as low as 0.1% are detected at coverage ~5000 and their frequencies are correctly estimated
- Every alignment position is covered by exactly three overlapping windows of width ~average read length
- Gibbs sampler mixes properly: solutions proposing new clusters appear (iterations and alpha set large enough), not a single-cluster collapse
- Error correction completes in reasonable time (<12 h on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads), indicating non-degenerate runs

## Limitations

- Performance depends on high coverage (~5000-6000x); rare-clone detection at 0.1% was demonstrated only in that regime
- Sequencing errors must be properly treated for deep sequencing to quantify diversity — uncorrected strand or platform biases remain a risk
- ShoRAH2 has global haplotype reconstruction disabled; only ShoRAH1 v1.1.3 (or V-pipe options) provide global mode
- ShoRAH must be compiled and run from the command line; SAMtools and pysam for SAM alignment must be obtained separately and are not included

## Evidence

- [intro] Corrects errors by windowed Dirichlet clustering: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [intro] Cluster read counts estimate prevalence: "The number of reads associated with the cluster estimates the prevalence of the haplotype in the population"
- [methods] Five-fold error-rate decrease on the 454 mixture: "A five-fold decrease of the error rate, from 0.25% to 0.05%, was achieved"
- [methods] 0.1% haplotypes detected at coverage 5000: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] shorah shotgun invokes diri_sampler on windows: "They work by cutting window from the multiple sequence alignment, invoking `diri_sampler` on the windows and calling `shorah snv` for the SNV calling"
- [readme] Global mode disabled in ShoRAH2: "Currently in ShoRAH2 global haplotype reconstruction is disabled."
