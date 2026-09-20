---
name: sequence-read-deduplication
description: Use when error-corrected, window-based reads from a ShoRAH run must be
  collapsed into a minimal non-redundant set before running global haplotype reconstruction
  (mm.py) and frequency estimation (freqEst).
when_to_use_negative:
- Only local analysis is needed — invoking shorah shotgun or shorah amplicon handles
  windows and SNV calling without the contain/deduplication step
- Running ShoRAH2, where global haplotype reconstruction is disabled and deduplication
  for global assembly is not available (use ShoRAH1 v1.1.3 or V-pipe's global haplotype
  options instead)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ShoRAH
  role: Pipeline whose error-corrected read output is the input to the deduplication
    step; global mode required
  repo: http://www.cbg.ethz.ch/software/shorah
- name: shorah.py
  role: Wrapper orchestrating the four major steps (alignment, error correction, global
    reconstruction, frequency estimation) that include this deduplication step
  repo: http://www.cbg.ethz.ch/software/shorah
- name: contain
  role: Program that extracts the set of unique reads maximal with respect to the
    substring relation from error-corrected reads
  repo: http://www.cbg.ethz.ch/software/shorah
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/1471-2105-12-119
    title: 'ShoRAH: estimating the genetic diversity of a mixed sample from next-generation
      sequencing data'
  source_content_hash: sha256:d8336d5e4caed7782d2c92f6c058a8d860e11eaeabba3cfb11f67c51c52be502
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ShoRAH
  - contain
  - freqEst
  - mm.py
  - shorah.py
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sequence-read-deduplication

## Summary

Extract the set of unique error-corrected reads that are maximal with respect to the substring relation using ShoRAH's contain program, so that redundant reads are removed before global haplotype reconstruction. This deduplication step is the bridge between local error correction and parsimony-based global assembly.

## When to use

Use when error-corrected, window-based reads from a ShoRAH run must be collapsed into a minimal non-redundant set before running global haplotype reconstruction (mm.py) and frequency estimation (freqEst).

## When NOT to use

- Only local analysis is needed — invoking shorah shotgun or shorah amplicon handles windows and SNV calling without the contain/deduplication step
- Running ShoRAH2, where global haplotype reconstruction is disabled and deduplication for global assembly is not available (use ShoRAH1 v1.1.3 or V-pipe's global haplotype options instead)

## Inputs

- Error-corrected reads (output of the ShoRAH local error-correction stage, from window-based Dirichlet process clustering via diri_sampler)
- Multiple sequence alignment of reads against the reference of the amplified region

## Outputs

- Set of unique reads maximal with respect to the substring relation (non-redundant read set)
- Downstream: reconstructed global haplotype sequences and their estimated frequencies (haplotype_frequencies.tsv)

## How to apply

Load the error-corrected reads produced by the local error-correction stage of the ShoRAH pipeline (the wrapper shorah.py performs alignment, error correction, global haplotype reconstruction, and frequency estimation in that order). Run the contain program to extract the set of unique reads that are maximal with respect to the substring relation, i.e., drop any read that is a substring of another read since it carries no additional haplotype information. Feed the resulting non-redundant read set into mm.py, which solves a maximum weight matching problem under a parsimony principle (worst-case cubic time in the number of unique reads) to compute a minimal set of haplotypes explaining all reads. Note that global reconstruction including contain is only available in ShoRAH1 (last version with global mode is v1.1.3); in ShoRAH2 global haplotype reconstruction is disabled.

## Related tools

- **ShoRAH** (Pipeline whose error-corrected read output is the input to the deduplication step; global mode required) — http://www.cbg.ethz.ch/software/shorah
- **shorah.py** (Wrapper orchestrating the four major steps (alignment, error correction, global reconstruction, frequency estimation) that include this deduplication step) — http://www.cbg.ethz.ch/software/shorah
- **contain** (Program that extracts the set of unique reads maximal with respect to the substring relation from error-corrected reads) — http://www.cbg.ethz.ch/software/shorah

## Examples

```
conda install shorah=1.1.3  # last ShoRAH1 version with global mode (contain -> mm.py -> freqEst)
```

## Evaluation signals

- The output read set contains no pair where one read is a substring of another (maximality invariant)
- All unique error-corrected input reads are explained by some read in the deduplicated set (no information loss)
- The subsequent mm.py maximum weight matching run terminates within its cubic worst-case time bound and produces a minimal haplotype set explaining all reads
- Estimated frequencies from freqEst resolve low-frequency haplotypes, consistent with the paper's detection of haplotypes at frequencies as low as 0.1% at coverage 5000

## Limitations

- Global haplotype reconstruction (and thus this deduplication step) is disabled in ShoRAH2; the last version of ShoRAH1 with global mode is v1.1.3
- mm.py has worst-case time complexity cubic in the number of unique reads, so very large deduplicated read sets are computationally costly
- The paper's global reconstruction was demonstrated on simulated HIV-1 subtype B pol gene data and 454 pyrosequencing reads; the software was tested on Linux and Mac OS X platforms

## Evidence

- [other] run the program contain to extract the set of unique reads that are maximal with respect to the substring relation: [evidence span withheld — anchor status: not_found]
- [methods] four major pipeline steps: "These are: 1) alignment; 2) error correction (local haplotype reconstruction); 3) global haplotype reconstruction and 4) frequency estimation"
- [other] mm.py parsimony and complexity: [evidence span withheld — anchor status: not_found]
- [readme] ShoRAH2 global mode disabled: "Currently in ShoRAH2 global haplotype reconstruction is disabled."
- [methods] low-frequency detection: "Haplotypes with frequencies as low as 0.1% were detected at a coverage of 5000 and their frequency correctly estimated"
