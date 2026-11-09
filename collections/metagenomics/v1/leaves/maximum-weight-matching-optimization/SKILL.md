---
name: maximum-weight-matching-optimization
description: Use when error-corrected local haplotypes from ShoRAH windows must be
  stitched into full-genome haplotypes and their frequencies estimated from deep sequencing
  of a mixed sample.
when_to_use_negative:
- ShoRAH2 is installed and only SNV calling or local (window-level) analysis is needed
  — global haplotype reconstruction is disabled there
- The goal is single nucleotide variant detection with strand-bias correction — use
  shorah snv instead of global reconstruction
- 'Coverage is low: the article detects haplotypes as rare as 0.1% only at coverage
  5000, so low-coverage data will not support reliable global assembly'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: mm.py
  role: Solves the maximum weight matching problem under a parsimony principle to
    compute a minimal set of haplotypes explaining all unique reads
  repo: github.com/cbg-ethz/shorah
- name: contain
  role: Extracts the set of unique reads that are maximal with respect to the substring
    relation, the input to mm.py
  repo: github.com/cbg-ethz/shorah
- name: freqEst
  role: Expectation Maximization algorithm that estimates maximum-likelihood haplotype
    frequencies (threshold 10^-6, max 5000 iterations)
  repo: github.com/cbg-ethz/shorah
- name: shorah
  role: ShoRAH1 wrapper providing the global pipeline; must be version v1.1.3 since
    global mode is disabled in ShoRAH2
  repo: github.com/cbg-ethz/shorah/tree/v1.1.3
- name: V-pipe
  role: Recommended workflow for accessing global haplotype reconstruction options
  repo: github.com/cbg-ethz/V-pipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:1842bd35552b236a729e6b94f7a87d6e43884719333810084dfdb550193b8ab4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - V-pipe
  - contain
  - freqEst
  - mm.py
  - shorah
  - shorah.py
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# maximum-weight-matching-optimization

## Summary

Reconstruct global (full-length) haplotypes from ShoRAH's error-corrected local window haplotypes by computing a minimal set of haplotypes that explain all unique reads via maximum weight matching, then estimating their population frequencies with an Expectation Maximization algorithm (freqEst).

## When to use

Use when error-corrected local haplotypes from ShoRAH windows must be stitched into full-genome haplotypes and their frequencies estimated from deep sequencing of a mixed sample.

## When NOT to use

- ShoRAH2 is installed and only SNV calling or local (window-level) analysis is needed — global haplotype reconstruction is disabled there
- The goal is single nucleotide variant detection with strand-bias correction — use shorah snv instead of global reconstruction
- Coverage is low: the article detects haplotypes as rare as 0.1% only at coverage 5000, so low-coverage data will not support reliable global assembly

## Inputs

- Error-corrected reads / local window haplotypes (output of ShoRAH local error-correction stage)
- Sorted BAM/CRAM alignment of shotgun or amplicon reads to the reference
- Error-corrected reads output of ShoRAH local error-correction stage

## Outputs

- Set of reconstructed global (full-length) haplotype sequences
- Haplotype population frequencies (maximum-likelihood estimates) in haplotype_frequencies.tsv

## How to apply

Start from the error-corrected reads produced by the local error-correction stage and run the program contain to extract unique reads that are maximal with respect to the substring relation. Then run mm.py, which implements global reconstruction via a parsimony principle by solving a maximum weight matching problem (worst-case time complexity cubic in the number of unique reads) to compute a minimal set of haplotypes explaining all reads. Finally run freqEst, an EM algorithm estimating haplotype frequencies by maximum likelihood, using a convergence threshold of 10^-6 on the log-likelihood difference between iterations and a cap of 5000 iterations. Write the reconstructed haplotype sequences and frequencies to haplotype_frequencies.tsv. Note that this global mode exists only in ShoRAH1 v1.1.3; ShoRAH2 has it disabled, so pin the bioconda v1.1.3 package or use the global haplotype options in V-pipe.

## Related tools

- **mm.py** (Solves the maximum weight matching problem under a parsimony principle to compute a minimal set of haplotypes explaining all unique reads) — github.com/cbg-ethz/shorah
- **contain** (Extracts the set of unique reads that are maximal with respect to the substring relation, the input to mm.py) — github.com/cbg-ethz/shorah
- **freqEst** (Expectation Maximization algorithm that estimates maximum-likelihood haplotype frequencies (threshold 10^-6, max 5000 iterations)) — github.com/cbg-ethz/shorah
- **shorah** (ShoRAH1 wrapper providing the global pipeline; must be version v1.1.3 since global mode is disabled in ShoRAH2) — github.com/cbg-ethz/shorah/tree/v1.1.3
- **V-pipe** (Recommended workflow for accessing global haplotype reconstruction options) — github.com/cbg-ethz/V-pipe

## Examples

```
contain <window_dir>; python mm.py <window_dir>; freqEst <window_dir>  # v1.1.3: parsimony haplotype assembly then EM frequency estimation
```

## Evaluation signals

- The EM run in freqEst terminates either by convergence (log-likelihood difference below 10^-6) or after the 5000-iteration cap
- The computed haplotype set is minimal (parsimony): every unique read is explained as a substring of at least one reconstructed haplotype
- Frequencies in haplotype_frequencies.tsv sum to 1 and resolve low-frequency variants, consistent with detection of haplotypes at 0.1% frequency at coverage 5000
- Runtime of mm.py stays tractable — worst-case cubic in the number of unique reads; a blow-up indicates too many unreduced unique reads entering contain

## Limitations

- Global haplotype reconstruction is disabled in ShoRAH2; the last ShoRAH1 version with global mode is v1.1.3, which must be obtained from bioconda or the V-pipe global options used
- Haplotype detection sensitivity depends on coverage: haplotypes as low as 0.1% frequency were detected only at coverage 5000
- mm.py has worst-case time complexity cubic in the number of unique reads, which can become expensive for highly diverse samples

## Evidence

- [methods] pipeline steps include global reconstruction: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [methods] low-frequency haplotype detection benchmark: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
- [readme] ShoRAH2 global mode disabled: "Currently in ShoRAH2 global haplotype reconstruction is disabled."
- [readme] global reconstruction recommendation: "If you wish to perform global reconstruction, we suggest that you consider the"
