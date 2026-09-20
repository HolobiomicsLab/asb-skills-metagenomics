---
name: nanopore-read-classification
description: Use when base-called nanopore reads must be classified as target vs non-target
  (e.g., adaptive-sampling host/strain depletion) against one-genome or multi-strain
  pan-genome references where minimap2 alignment is too slow or memory-hungry.
when_to_use_negative:
- 'Direct real-time control through the nanopore Read Until API is required: SPUMONI
  operates on base-called sequences, not the raw signal batches, so a separate base
  caller must run upstream (as with Readfish)'
- Depletion-target and non-target reads share substantial sequence similarity (e.g.,
  conserved genes between species) — the reversed null index may fail to discriminate
- Read accuracy or reference differs markedly from the real nanopore data used to
  tune the single shared KS-statistic threshold; no per-experiment thresholding method
  is provided
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: SPUMONI
  role: Builds the MS/PML pan-genome indexes (`spumoni build -r <ref.fa> -M -P -m`)
    and performs the streaming per-read classification with `-c` reports; PML mode
    runs an online one-pass MS-related computation over the RLE BWT in O(m log log
    n) time and O(r) space
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Alignment baseline producing the MAPQ >= 30 gold-standard target/non-target
    calls used to compute SPUMONI's precision, recall, and F1 and to compare throughput
- name: PBSIM2
  role: Simulates nanopore reads (e.g., at 90% base-call accuracy from ZymoMC or CHM13
    references) for controlled classification benchmarks
  repo: https://github.com/yukiteruono/pbsim2
- name: MONI
  role: Batch matching-statistics algorithm whose batch MS-finding SPUMONI replaces
    with a faster online PML procedure (~3x faster than MONI with similar or greater
    accuracy)
  repo: https://github.com/maxrossi91/moni
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:45b48ceacc320b5afb35c714ff7f29fdae3c47c74e9c340a7d4f80f27ea3c697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - Big-BWT
  - MONI
  - PBSIM2
  - SPUMONI
  - minimap2
  - pfp-thresholds
  - sdsl-lite
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# nanopore-read-classification

## Summary

Rapid target/non-target classification of base-called nanopore reads using SPUMONI's matching statistics (MS) or pseudomatching lengths (PML) computed against a compressed one-genome or multi-strain pan-genome index, with a reversed 'null index' and KS-statistic hypothesis testing to make principled ejection decisions. Achieves minimap2-comparable F1 (~97-99%) at ~12x the throughput with a much smaller index.

## When to use

Use when base-called nanopore reads must be classified as target vs non-target (e.g., adaptive-sampling host/strain depletion) against one-genome or multi-strain pan-genome references where minimap2 alignment is too slow or memory-hungry.

## When NOT to use

- Direct real-time control through the nanopore Read Until API is required: SPUMONI operates on base-called sequences, not the raw signal batches, so a separate base caller must run upstream (as with Readfish)
- Depletion-target and non-target reads share substantial sequence similarity (e.g., conserved genes between species) — the reversed null index may fail to discriminate
- Read accuracy or reference differs markedly from the real nanopore data used to tune the single shared KS-statistic threshold; no per-experiment thresholding method is provided

## Inputs

- Base-called nanopore reads (FASTA/FASTQ), e.g., ONT ZymoBIOMICS reads from SRA SRX7711546, filtered to >= 4000 bp and truncated to the first 720 bases
- Reference FASTA: one genome (e.g., CHM13) or a multi-FASTA pan-genome of many strains per species (with or without the depletion-target strain)
- PBSIM2-simulated nanopore reads (e.g., 90% base-call accuracy from ZymoMC references) for controlled benchmarking
- minimap2 alignments (MAPQ >= 30) against the same references, used as gold-standard target/non-target calls

## Outputs

- SPUMONI index files (RLE BWT, MONI thresholds, MS and PML structures) per reference strategy
- Per-read target/non-target classification report from `spumoni run ... -c`
- Precision, recall, and F1 scores versus the minimap2 MAPQ >= 30 gold standard (e.g., table1_f1_scores.csv, table3_zymo_benchmark.csv, table4_spumoni_vs_minimap2.csv)
- Throughput (kbp/s), index size (GB), and peak RSS comparisons between SPUMONI and minimap2

## How to apply

Build an index from a single reference (e.g., CHM13) or a multi-FASTA pan-genome of many strains per species — the depletion-target strain may be excluded — with `spumoni build -r <ref.fa> -M -P -m -o <prefix>`, which constructs both matching-statistics (MS) and pseudomatching-length (PML) indexes over the run-length-encoded BWT and MONI thresholds. Preprocess real ONT reads (e.g., ZymoBIOMICS SRX7711546) by keeping reads >= 4000 bp and truncating each to its first 720 bases, then run `spumoni run -r <index_prefix> -p reads.fa -P -c` in PML mode (or `-M` for MS) so the `-c` flag writes a per-read classification report; SPUMONI can suspend and resume its MS/PML computation as each new batch of data arrives. Classify each read as target or non-target using the null-index (reverse of the positive-index sequences) KS-statistic hypothesis-testing threshold, judging whether observed MS/PML lengths exceed what is expected by chance. Benchmark accuracy against a minimap2-derived gold standard (MAPQ >= 30) using precision/recall/F1 — expected values include 99.7% F1 for SPUMONI-PML/MS and minimap2 on simulated 90%-accuracy ZymoMC reads with a pan-genome index, and 97.94% F1 at ~11.9x throughput on real ZymoMC reads. Install via the Docker/Singularity images or build from source with `git clone https://github.com/oma219/spumoni`, `cmake ..`, `make install`.

## Related tools

- **SPUMONI** (Builds the MS/PML pan-genome indexes (`spumoni build -r <ref.fa> -M -P -m`) and performs the streaming per-read classification with `-c` reports; PML mode runs an online one-pass MS-related computation over the RLE BWT in O(m log log n) time and O(r) space) — https://github.com/oma219/spumoni
- **minimap2** (Alignment baseline producing the MAPQ >= 30 gold-standard target/non-target calls used to compute SPUMONI's precision, recall, and F1 and to compare throughput)
- **PBSIM2** (Simulates nanopore reads (e.g., at 90% base-call accuracy from ZymoMC or CHM13 references) for controlled classification benchmarks) — https://github.com/yukiteruono/pbsim2
- **MONI** (Batch matching-statistics algorithm whose batch MS-finding SPUMONI replaces with a faster online PML procedure (~3x faster than MONI with similar or greater accuracy)) — https://github.com/maxrossi91/moni

## Examples

```
spumoni build -r pangenome.fa -M -P -m -o pgidx; spumoni run -r pgidx -p reads.fa -P -c
```

## Evaluation signals

- F1 versus minimap2 MAPQ >= 30 gold standard on simulated 90%-accuracy ZymoMC reads: SPUMONI 99.74 vs minimap2 99.74 on the pan-genome index without mock refs, and 93.13/95.00 on the one-genome index without mock refs (Table 1)
- Real ZymoMC pan-genome benchmark: SPUMONI F1 ~ 97.94% vs minimap2 98.73%, with ~11.9x faster per-read classification throughput
- Human host-depletion benchmark: F1 96.97 vs 95.17 (one-genome) and 97.08 vs 99.17 (three-genome); throughput 27.0 vs 13.5 kbp/s; index sizes 18 vs 21 GB
- Every input read receives a classification entry in the `-c` report file, with one MS/PML value emitted per position of each read
- On the mock-community scenario, SPUMONI's many-strain index should be about one-sixteenth the size of minimap2's, with peak memory footprint less than one-fourth

## Limitations

- SPUMONI operates on base-called sequences rather than the batches of signal delivered by the Read Until API, so it cannot yet directly interact with the sequencer and requires a separate upstream base caller
- The null index (reverse of the positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g., conserved genes between species
- A single KS-stat threshold (optimized on real nanopore data) is used for all experiments; the optimal threshold is expected to be a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases, and for three human genomes SPUMONI's peak RSS is about 1.86x larger than minimap2's
- The SPUMONI version number is not stated in the methods

## Evidence

- [abstract] We present a novel method called SPUMONI, which enables rapid and accurate targeted sequencing using efficient pan-genome indexes.: "We present a novel method called SPUMONI, which enables rapid and accurate targeted sequencing using efficient pan-genome indexes."
- [abstract] However SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [intro] SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices: "SPUMONI adds a “null index” together with a hypothesis testing framework to make principled ejection choices"
- [intro] SPUMONI replaces MONI's “batch” MS-finding algorithm with a faster online algorithm that calculates a different quantity related to the MS, called the “pseudomatching length (PML)”: [evidence span withheld — anchor status: not_found]
- [methods] The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use: "The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use"
- [discussion] it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
- [readme] SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
