---
name: matching-statistics-hypothesis-testing
description: Use when classifying basecalled nanopore reads as target vs non-target
  against a repetitive pan-genomic reference for targeted sequencing/depletion, especially
  when alignment tools like minimap2 are too slow or memory-heavy.
when_to_use_negative:
- When working directly with raw nanopore electrical signal via the Read Until API
  — SPUMONI operates on sequences of bases, so a separate base caller must run upstream
- When the depletion target and non-targets share substantial sequence similarity
  (e.g. conserved genes between species), where the reverse-sequence null index may
  be insufficient
- When indexing a single, non-repetitive genome where alignment-based classifiers
  are already fast enough and the pan-genome matching-statistics advantage does not
  apply
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Builds the MS/PML pan-genome indexes and runs streaming read classification
    with null-index KS-statistic hypothesis testing
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Alignment-based classifier used as the accuracy baseline for target/non-target
    calls against the same reference strategies
- name: MONI
  role: Predecessor batch algorithm computing matching statistics; SPUMONI's online
    PML algorithm is ~3x faster than MONI with similar or greater accuracy
  repo: https://github.com/maxrossi91/moni
- name: PBSIM2
  role: Simulator used to generate 90%-accuracy nanopore ZymoMC reads for benchmarking
    classification accuracy
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:098f688eabfa91109bc23a0fc1049cc626b50efa126b388e5efd88755c2bc967
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - MONI
  - PBSIM2
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# matching-statistics-hypothesis-testing

## Summary

A read-classification method (SPUMONI) that uses compressed pan-genome indexes to compute matching statistics (MS) or pseudo-matching lengths (PML), then applies a null-index KS-statistic hypothesis test to decide whether a read is a target for nanopore read-until depletion, achieving minimap2-comparable accuracy with far less index size and memory.

## When to use

Use when classifying basecalled nanopore reads as target vs non-target against a repetitive pan-genomic reference for targeted sequencing/depletion, especially when alignment tools like minimap2 are too slow or memory-heavy.

## When NOT to use

- When working directly with raw nanopore electrical signal via the Read Until API — SPUMONI operates on sequences of bases, so a separate base caller must run upstream
- When the depletion target and non-targets share substantial sequence similarity (e.g. conserved genes between species), where the reverse-sequence null index may be insufficient
- When indexing a single, non-repetitive genome where alignment-based classifiers are already fast enough and the pan-genome matching-statistics advantage does not apply

## Inputs

- Basecalled sequencing reads in FASTA/FASTQ (e.g. PBSIM2-simulated 90%-accuracy ZymoMC nanopore reads)
- Reference FASTA files for one or more strains (pan-genome index inputs, with or without the mock-community reference strains)

## Outputs

- Per-read target/non-target classification calls derived from SPUMONI MS or PML statistics via the null-index KS-statistic threshold
- Precision, recall, and F1 scores per tool (SPUMONI-PML, SPUMONI-MS, minimap2) x reference-strategy combination (e.g. table1_f1_scores.csv)

## How to apply

Build a SPUMONI index with 'spumoni build -r <ref>.fa -M -P -m -o <prefix>', which constructs both a matching-statistics index and a pseudo-matching-lengths index (alternatively via Docker/Singularity images or 'git clone https://github.com/oma219/spumoni' followed by cmake build). Run classification with 'spumoni run -r <index_prefix> -p reads.fa -P -c' for PML mode or substituting -M for MS mode. For the null-index hypothesis test, SPUMONI adds a null index (reverse of the positive-index sequences) and rejects the null when observed MS/PML lengths are longer than expected by chance, using a single KS-statistic threshold optimized for real nanopore data. Benchmark against minimap2 alignments by computing precision, recall, and F1 per tool x reference-strategy combination; in the ZymoMC mock-community depletion scenario (PBSIM2-simulated 90%-accuracy reads), pan-genome indexes yield F1 ~99.7% for both SPUMONI and minimap2, and SPUMONI is about 12x faster with an index ~1/16 the size and peak memory under 1/4 of minimap2's.

## Related tools

- **SPUMONI** (Builds the MS/PML pan-genome indexes and runs streaming read classification with null-index KS-statistic hypothesis testing) — https://github.com/oma219/spumoni
- **minimap2** (Alignment-based classifier used as the accuracy baseline for target/non-target calls against the same reference strategies)
- **MONI** (Predecessor batch algorithm computing matching statistics; SPUMONI's online PML algorithm is ~3x faster than MONI with similar or greater accuracy) — https://github.com/maxrossi91/moni
- **PBSIM2** (Simulator used to generate 90%-accuracy nanopore ZymoMC reads for benchmarking classification accuracy) — https://github.com/yukiteruono/pbsim2

## Examples

```
spumoni build -r pangenome.fa -M -P -m -o pangenome_idx; spumoni run -r pangenome_idx -p reads.fa -P -c
```

## Evaluation signals

- F1 scores reproduce Table 1 values: SPUMONI 99.74 vs minimap2 99.74 on Pan-genome without mock refs; 93.13/95.00 on One genome without mock refs
- Pan-genome index excluding the specific depletion-target strain still yields F1 ~99.7% for both SPUMONI and minimap2 on simulated 90%-accuracy ZymoMC reads
- Classification calls in MS mode (spumoni run -M) and PML mode (spumoni run -P) are each computable and comparable via precision/recall/F1
- Index and peak memory footprint check: SPUMONI's index ~1/16 and peak RSS ~1/4 or less of minimap2's on the many-strain reference
- Hypothesis testing invariants: reads ejected only when observed MS/PML lengths exceed the null-index expectation at the chosen KS-stat threshold

## Limitations

- SPUMONI operates on base-called sequences, not raw electrical signal, and cannot yet directly interact with the Read Until API as control software; a separate base caller must run upstream
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species
- A single KS-stat threshold is used for all experiments; the optimal threshold depends on read accuracy and reference used, and no per-experiment thresholding method is provided
- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- SPUMONI's peak RSS for three human genomes is about 1.86X larger than minimap2's

## Evidence

- [intro] SPUMONI adds a 'null index' together with a hypothesis testing framework to make principled ejection choices: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices depending on whether the observed MS lengths are longer than what would be expected by"
- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [abstract] SPUMONI is 12 times faster than minimap2: "However SPUMONI is 12 times faster than minimap2."
- [methods] README install instructions for SPUMONI from source: "git clone https://github.com/oma219/spumoni
cd spumoni
mkdir build && cd build
cmake ..
make install"
- [readme] README describes classification mode flags: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
- [discussion] Null index caveat for similar sequences: "if there is substantial sequence similarity between depletion-target reads and reads that should not be targeted for depletion"
- [discussion] Single KS threshold caveat: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
