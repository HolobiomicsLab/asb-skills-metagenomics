---
name: ks-statistic-null-distribution-testing
description: Use when classifying streaming sequencing reads as target vs. random-chance
  hits using matching statistics, requiring a principled null-model test rather than
  an alignment score cutoff.
when_to_use_negative:
- When depletion-target and non-target reads share substantial sequence similarity
  (e.g. conserved genes between species), the reverse-sequence null index may not
  discriminate.
- When raw nanopore electrical signal must be analyzed directly — SPUMONI operates
  on base-called sequences and requires a separate base caller upstream.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Implements positive and null index construction, streaming MS/PML computation,
    KS-statistic thresholding, and majority-of-regions read classification
  repo: github.com/oma219/spumoni
- name: MONI
  role: Underlying batch matching-statistics algorithm that SPUMONI's classification
    approach extends with a null index and hypothesis testing
  repo: github.com/maxrossi91/moni
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:70dc4a2e6d280c855b976665b2ce0117d21cbfb6b5a59ebad994f39dec3002ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MONI
  - SPUMONI
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ks-statistic-null-distribution-testing

## Summary

Hypothesis-testing classification method from SPUMONI that decides whether a streaming read belongs to a target set by comparing its matching statistics (or PMLs) against a positive index and a null index (built from reversed sequences) using a Kolmogorov-Smirnov statistic threshold.

## When to use

Use when classifying streaming sequencing reads as target vs. random-chance hits using matching statistics, requiring a principled null-model test rather than an alignment score cutoff.

## When NOT to use

- When depletion-target and non-target reads share substantial sequence similarity (e.g. conserved genes between species), the reverse-sequence null index may not discriminate.
- When raw nanopore electrical signal must be analyzed directly — SPUMONI operates on base-called sequences and requires a separate base caller upstream.

## Inputs

- FASTA reference of target sequences for the positive index
- FASTQ/base-called nanopore reads to classify
- Null index derived from the reverse of the positive-index sequences

## Outputs

- Per-read classification report (target/nontarget) written with the -c option
- Per-region KS statistics comparing observed versus null MS/PML distributions

## How to apply

Build two indexes with 'spumoni build -r <reference.fa> -M -P -m -o <prefix>': a positive index over the target reference sequences including their reverse complements, and a null index constructed from the reverse (not reverse-complement) of those sequences, so null statistics reflect random-chance MS/PML behavior. Stream query reads through 'spumoni run', computing either MSs (with -M) or PMLs (with -P) over 90 bp nonoverlapping regions of each read against both indexes. For each region, subtract the null-index mean statistic from the observed statistic and compute the Kolmogorov-Smirnov statistic comparing the observed versus null distributions. Classify each read as target if the majority of its 90 bp regions exceed the KS-stat threshold — 0.25 for MS and 0.10 for PML — and write classifications to a report with the -c option. Rationale: longer MSs indicate a better approximate match to the index, so a null index plus hypothesis testing makes principled ejection choices when observed lengths exceed what random chance would produce.

## Related tools

- **SPUMONI** (Implements positive and null index construction, streaming MS/PML computation, KS-statistic thresholding, and majority-of-regions read classification) — github.com/oma219/spumoni
- **MONI** (Underlying batch matching-statistics algorithm that SPUMONI's classification approach extends with a null index and hypothesis testing) — github.com/maxrossi91/moni

## Examples

```
spumoni build -r reference.fa -M -P -m -o prefix; spumoni run -i reads.fq -o prefix -c classifications.txt
```

## Evaluation signals

- On simulated E. coli reads where positive and null statistic distributions are known to differ, the report separates target from nontarget reads by majority-of-regions voting.
- The majority of a read's 90 bp regions exceed the KS-stat threshold (0.25 for MS, 0.10 for PML) for target calls.
- Null-index statistics computed from reversed sequences behave like random-chance matches, giving observed-minus-null mean differences near zero for nontarget reads.
- Classification accuracy should approximate minimap2's in mock-community depletion scenarios while using an index about one-sixteenth the size.

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold is a function of read accuracy and the reference used, and no per-experiment thresholding method is provided.
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads.
- Users must run a separate base caller upstream of SPUMONI; it cannot yet directly interact with the Read Until API as control software.

## Evidence

- [intro] SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by"
- [intro] The first insight of SPUMONI is that these statistics can be used to classify the query sequence; longer MSs indicate a better approximate match to the index: "The first insight of SPUMONI is that these statistics can be used to classify the query sequence; longer MSs indicate a better approximate match to the index"
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [discussion] it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
