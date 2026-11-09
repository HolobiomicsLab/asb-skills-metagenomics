---
name: blast-evalue-threshold-interpretation
description: Use when a set of nucleotide sequences (e.g. a FASTA catalogue of ISs)
  must be classified against a reference database and the goal is a reproducible strict/loose/novel
  homology split based on BLAST e-value thresholds.
when_to_use_negative:
- The query sequences are not the nucleotide IS catalogue entries (e.g. protein sequences
  or non-IS contigs), since the strict/loose e-value bands were calibrated for IS-vs-ISfinder
  nucleotide queries.
- The analysis goal requires richer homology evidence than e-value alone (e.g. coverage,
  percent identity, or transposase-domain annotation), which this binning scheme does
  not capture.
- Only a strict-homology judgement is needed at the 0.01 cutoff, i.e. a single-tier
  hit/miss call, since the two-tier scheme adds overhead without benefit.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Palidis
  role: Pipeline that discovered the 879 ISC insertion sequences that are subsequently
    classified by this e-value binning scheme
  repo: https://github.com/blue-moon22/palidis
- name: ISfinder
  role: Reference database of known transposable elements queried via its online BLAST
    service at an e-value cutoff of 0.01
- name: COBS index
  role: Alternative compact bit-sliced index of 661 405 microbial genomes that can
    be queried to predict IS origins
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:7382313539371a1d6b3c0e33fb19daccbc5a30153742b3869e5fdab94b09ea23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - COBS index
  - ISfinder
  - Palidis
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# blast-evalue-threshold-interpretation

## Summary

Classifies catalogue sequences (here, the 879 insertion sequences of ISC v1.0.0) against a reference database such as ISfinder by binning BLAST e-values into strict hits (e-value <1e-50), loose hits (0.01 > e-value >= 1e-50), and novel (no hit at the 0.01 cutoff), yielding a quantitative homology breakdown.

## When to use

Use when a set of nucleotide sequences (e.g. a FASTA catalogue of ISs) must be classified against a reference database and the goal is a reproducible strict/loose/novel homology split based on BLAST e-value thresholds.

## When NOT to use

- The query sequences are not the nucleotide IS catalogue entries (e.g. protein sequences or non-IS contigs), since the strict/loose e-value bands were calibrated for IS-vs-ISfinder nucleotide queries.
- The analysis goal requires richer homology evidence than e-value alone (e.g. coverage, percent identity, or transposase-domain annotation), which this binning scheme does not capture.
- Only a strict-homology judgement is needed at the 0.01 cutoff, i.e. a single-tier hit/miss call, since the two-tier scheme adds overhead without benefit.

## Inputs

- FASTA file of catalogue insertion sequences (ISC v1.0.0, 879 ISs, from github.com/blue-moon22/ISC)
- ISfinder reference database via its online BLAST service (e-value cutoff 0.01, default parameters)

## Outputs

- Per-sequence classification: strict hit (e-value <1e-50), loose hit (0.01 > e-value >= 1e-50), or novel (no hit at 0.01)
- Summary CSV (isfinder_homology_benchmark.csv) with counts per class

## How to apply

Download the catalogue FASTA (879 ISC insertion sequences from github.com/blue-moon22/ISC) and query each sequence against ISfinder's online BLAST service with an e-value cutoff of 0.01 and default parameters. Assign each sequence to exactly one class: a strict homolog if the best hit has e-value <1e-50, a loose homolog if 0.01 > e-value >= 1e-50, or novel if no hit passes the 0.01 threshold. The rationale for two tiers is that <1e-50 indicates strong evidence the IS is previously characterized (ISfinder), while hits in the 1e-50–0.01 band suggest weaker, partial, or distant homology. Tabulate the counts and write the summary to a CSV (isfinder_homology_benchmark.csv); for ISC v1.0.0 the expected split is 360 (41.0%) ISfinder hits (60 strict + 300 loose) and 519 novel sequences.

## Related tools

- **Palidis** (Pipeline that discovered the 879 ISC insertion sequences that are subsequently classified by this e-value binning scheme) — https://github.com/blue-moon22/palidis
- **ISfinder** (Reference database of known transposable elements queried via its online BLAST service at an e-value cutoff of 0.01)
- **COBS index** (Alternative compact bit-sliced index of 661 405 microbial genomes that can be queried to predict IS origins)

## Evaluation signals

- Every one of the 879 catalogue sequences is assigned to exactly one class; strict + loose + novel counts sum to 879 (60 + 300 + 519).
- Total ISfinder hits equal 360, i.e. 41.0% of the catalogue, matching the published benchmark.
- No sequence classified as strict has an e-value >= 1e-50, and no loose hit has an e-value outside the [1e-50, 0.01) band.
- Sequences with no ISfinder hit at the 0.01 threshold are counted as novel rather than dropped from the denominator.
- The summary table is written to isfinder_homology_benchmark.csv and reproduces the expected 60/300/519 split.

## Limitations

- The ISC only contains common DDE types of ISs with ITRs; ISs lacking inverted terminal repeats cannot enter the catalogue or be classified by this scheme.
- ISC v1.0.0 was built only from metagenomes of the healthy human oral cavity and stool samples, so homology fractions may not generalize to ISs from other sources.
- The catalogue contains ISs with ITRs of 25 or more nucleotides as generated by Palidis, although ITRs can be as short as ten nucleotides, so shorter-ITR ISs are absent from the classification input.
- Only the e-value criterion is used; hits in the loose band may reflect partial or distant homology and require further inspection.

## Evidence

- [intro] Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes: "Palidis can also predict the origins of these ISs by querying them against ISfinder or a compact bit-sliced signature (COBS) index [11] of 661 405 microbial genomes"
- [abstract] Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized.: "Applying Palidis to 264 human metagenomes identifies 879 unique insertion sequences, with 519 being novel and not previously characterized."
- [other] Classify each catalogue sequence as a strict hit (e-value <1e-50), a loose hit (0.01 > e-value >= 1e-50), or novel (no hit at the 0.01 threshold).: [evidence span withheld — anchor status: not_found]
- [readme] This repository contains a catalogue of insertion sequences in FASTA format generated using palidis4: [evidence span withheld — anchor status: not_found]
