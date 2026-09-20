---
name: viral-quasispecies-assembly
description: Use when deep-sequencing reads from a genetically heterogeneous viral
  sample must be assembled into full-length haplotypes with maximum-likelihood frequency
  estimates after local error correction.
when_to_use_negative:
- Only SNV calling is needed — run shorah snv or the local-only analysis (shorah shotgun
  / shorah amplicon) instead
- Running ShoRAH2 — global haplotype reconstruction is disabled in that version and
  ShoRAH1 v1.1.3 or V-pipe must be used
- Input reads have not yet been error-corrected — global reconstruction assumes the
  local haplotype reconstruction stage has already been completed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3168
tools:
- name: ShoRAH
  role: Suite providing the contain, mm.py and freqEst programs used for global haplotype
    reconstruction and frequency estimation
  repo: http://www.cbg.ethz.ch/software/shorah
- name: mm.py
  role: Python program implementing global reconstruction by maximum weight matching
    under a parsimony principle
  repo: https://github.com/cbg-ethz/shorah
- name: freqEst
  role: Expectation Maximization program estimating haplotype frequencies by maximum
    likelihood (threshold 10^-6, max 5000 iterations)
- name: V-pipe
  role: Snakemake workflow whose options include global haplotype reconstruction for
    viruses
  repo: https://github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:eaa99f99498b767f818ac882cf0bca11f35ac054f44a797df71be749573a0f3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - freqEst
  - mm.py
  - shorah.py
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# viral-quasispecies-assembly

## Summary

Reconstruct full-length (global) viral haplotypes from error-corrected local window haplotypes and estimate their population frequencies by maximum likelihood, using ShoRAH's contain/mm.py/freqEst chain as the final two steps of the four-step ShoRAH pipeline.

## When to use

Use when deep-sequencing reads from a genetically heterogeneous viral sample must be assembled into full-length haplotypes with maximum-likelihood frequency estimates after local error correction.

## When NOT to use

- Only SNV calling is needed — run shorah snv or the local-only analysis (shorah shotgun / shorah amplicon) instead
- Running ShoRAH2 — global haplotype reconstruction is disabled in that version and ShoRAH1 v1.1.3 or V-pipe must be used
- Input reads have not yet been error-corrected — global reconstruction assumes the local haplotype reconstruction stage has already been completed

## Inputs

- Error-corrected reads (output of the local error-correction stage, e.g. from diri_sampler)
- Sorted BAM/CRAM alignment of shotgun or amplicon reads to a reference sequence

## Outputs

- Reconstructed global haplotype sequences
- Haplotype frequency table written to haplotype_frequencies.tsv

## How to apply

After ShoRAH's alignment and local error-correction (local haplotype reconstruction) steps, load the error-corrected reads and run contain to extract the set of unique reads that are maximal with respect to the substring relation. Run mm.py, which implements the global reconstruction via a parsimony principle, solving a maximum weight matching problem (worst-case cubic time in the number of unique reads) to compute a minimal set of haplotypes explaining all reads. Then run freqEst, an Expectation Maximization algorithm estimating haplotype frequencies by maximum likelihood with a convergence threshold of 10^-6 on the log-likelihood difference and a maximum of 5000 iterations. Write the reconstructed haplotype sequences and estimated frequencies to haplotype_frequencies.tsv. Important: use ShoRAH1 v1.1.3 (or the global haplotype options in V-pipe), since global reconstruction is disabled in ShoRAH2.

## Related tools

- **ShoRAH** (Suite providing the contain, mm.py and freqEst programs used for global haplotype reconstruction and frequency estimation) — http://www.cbg.ethz.ch/software/shorah
- **mm.py** (Python program implementing global reconstruction by maximum weight matching under a parsimony principle) — https://github.com/cbg-ethz/shorah
- **freqEst** (Expectation Maximization program estimating haplotype frequencies by maximum likelihood (threshold 10^-6, max 5000 iterations))
- **V-pipe** (Snakemake workflow whose options include global haplotype reconstruction for viruses) — https://github.com/cbg-ethz/V-pipe

## Examples

```
conda install shorah=1.1.3  # bioconda ShoRAH1 with global mode enabled
```

## Evaluation signals

- The freqEst EM run terminates either by convergence (log-likelihood difference below 10^-6) or after the 5000-iteration cap
- The haplotype_frequencies.tsv table resolves low-frequency haplotypes, consistent with detection of haplotypes at frequencies as low as 0.1% at coverage 5000
- The computed haplotype set is minimal (parsimony) and explains all maximal unique reads produced by contain
- Estimated frequencies are maximum-likelihood values and should be checked for consistency (e.g. summing to ~1)

## Limitations

- Global haplotype reconstruction is disabled in ShoRAH2; the last version with global mode is ShoRAH1 v1.1.3
- mm.py has worst-case time complexity cubic in the number of unique reads, which limits scalability to very high haplotype diversity
- Accurate frequency estimation for very rare haplotypes requires very deep coverage (0.1% detection was reported at coverage 5000)

## Evidence

- [intro] Longer haplotypes can be assembled from overlapping local ones: "Longer haplotypes sequences can be reconstructed from adjacent, overlapping reads, and the frequencies of these global haplotypes can then be estimated"
- [methods] Four-step pipeline: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [methods] Rare haplotype detection benchmark: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] Global mode requires ShoRAH1 v1.1.3: "Currently in ShoRAH2 global haplotype reconstruction is disabled. The last version of ShoRAH1 with global mode is [v1.1.3]"
