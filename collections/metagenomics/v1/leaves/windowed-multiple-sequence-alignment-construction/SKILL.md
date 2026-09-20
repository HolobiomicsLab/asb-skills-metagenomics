---
name: windowed-multiple-sequence-alignment-construction
description: Use when deep-sequencing reads of a genetically heterogeneous viral sample
  must be aligned to a reference and split into overlapping windows to support local
  haplotype reconstruction and error correction.
when_to_use_negative:
- You only need single nucleotide variant calls without local haplotype context —
  shorah snv can be run for SNV calling directly, taking strand bias into account
- You require global (full-length) haplotype reconstruction from ShoRAH2, which has
  global mode disabled; use ShoRAH v1.1.3 or V-pipe's global haplotype options instead
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ShoRAH
  role: Package providing the alignment step, MSA construction, and window splitting
    that precedes local haplotype reconstruction
  repo: https://github.com/cbg-ethz/shorah
- name: shorah shotgun
  role: 'Invoked to run the local analysis: cuts windows from the multiple sequence
    alignment and invokes diri_sampler on the windows'
  repo: https://github.com/cbg-ethz/shorah
- name: b2w
  role: Splits shotgun sequencing .BAM files into multiple overlapping windows
  repo: https://github.com/cbg-ethz/shorah
- name: diri_sampler
  role: Gibbs sampler (Dirichlet process mixture) run on each window for error correction
    / local haplotype reconstruction
  repo: https://github.com/cbg-ethz/shorah
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:d89f6ee3b10a2a8f997c0483a78802fb8bf9f82664ee90a01cdb681bdc40e7be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/shorah
  tools:
  - Python
  - ShoRAH
  - b2w
  - diri_sampler
  - shorah shotgun
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# windowed-multiple-sequence-alignment-construction

## Summary

Construct and window a multiple sequence alignment (MSA) of reads aligned to a reference (e.g., HIV-1 subtype B pol) so that each genome position is covered by multiple overlapping windows of approximately read length, enabling local haplotype reconstruction with ShoRAH's diri_sampler.

## When to use

Use when deep-sequencing reads of a genetically heterogeneous viral sample must be aligned to a reference and split into overlapping windows to support local haplotype reconstruction and error correction.

## When NOT to use

- You only need single nucleotide variant calls without local haplotype context — shorah snv can be run for SNV calling directly, taking strand bias into account
- You require global (full-length) haplotype reconstruction from ShoRAH2, which has global mode disabled; use ShoRAH v1.1.3 or V-pipe's global haplotype options instead

## Inputs

- Sorted BAM/CRAM file of next-generation sequencing reads (e.g., test_aln.cram)
- Reference sequence of the amplified region (e.g., HIV-1 subtype B pol gene)
- Window width parameter (approximately the average read length)

## Outputs

- Multiple sequence alignment (MSA) of aligned reads
- Overlapping windowed sub-alignments covering each genome position three times
- Local haplotype reconstructions with cluster-based frequency estimates per window (input to SNV calling / global reconstruction)

## How to apply

Align the reads to the reference sequence of the amplified region (e.g., the HIV-1 subtype B pol gene) using ShoRAH's alignment step and build a multiple sequence alignment. Split the MSA into overlapping windows such that every alignment position is covered three times, with the window width approximately equal to the average read length. For shotgun data this windowing can be done from a sorted .BAM/.CRAM file (ShoRAH's b2w tool performs the window splitting). The resulting windows feed diri_sampler, a Gibbs sampler with a Dirichlet process mixture model, so tune the number of iterations and hyper-parameter alpha to be large enough for proper mixing; the number of reads associated with each cluster then estimates haplotype prevalence in the population.

## Related tools

- **ShoRAH** (Package providing the alignment step, MSA construction, and window splitting that precedes local haplotype reconstruction) — https://github.com/cbg-ethz/shorah
- **shorah shotgun** (Invoked to run the local analysis: cuts windows from the multiple sequence alignment and invokes diri_sampler on the windows) — https://github.com/cbg-ethz/shorah
- **b2w** (Splits shotgun sequencing .BAM files into multiple overlapping windows) — https://github.com/cbg-ethz/shorah
- **diri_sampler** (Gibbs sampler (Dirichlet process mixture) run on each window for error correction / local haplotype reconstruction) — https://github.com/cbg-ethz/shorah

## Examples

```
shorah shotgun
```

## Evaluation signals

- Every alignment position in the MSA is covered by three overlapping windows
- Window width is approximately equal to the average read length
- Local haplotype reconstruction detects and correctly estimates frequencies of rare variants — e.g., haplotypes at 0.1% frequency at a coverage of 5000 in the simulated 10-variant HIV-1 pol mixture
- Inferred haplotype frequencies from cluster read counts match the known ground-truth mixture proportions of simulated variants
- Error correction achieves the reported five-fold error-rate reduction (from 0.25% to 0.05%)

## Limitations

- ShoRAH2 has global haplotype reconstruction disabled; the last version of ShoRAH1 with global mode is v1.1.3, or V-pipe's global haplotype options can be considered
- The software must be compiled and run from the command line (Linux/Mac OS X tested; Windows via WSL or Cygwin); SAMtools and pysam-style dependencies are not all included in the package
- Error correction is computationally intensive — less than 12 hours on 6 CPUs at 2.8 GHz for 40,000 pyrosequencing reads
- diri_sampler requires tuned iteration counts and a sufficiently large hyper-parameter alpha for proper mixing of the sampling

## Evidence

- [intro] This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length: "This software corrects sequencing errors by clustering all reads that overlap the same region of the genome of length approximately equal to the read length"
- [methods] Haplotypes as low as 0.1% detected: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] shorah shotgun invokes local analysis: "The local analysis alone can be run invoking `shorah shotgun` or `shorah amplicon`"
- [readme] b2w splits BAM into windows: "`b2w`          | splitting shotgun sequencing .BAM into multiple overlapping windows"
- [methods] Four major analysis steps: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
