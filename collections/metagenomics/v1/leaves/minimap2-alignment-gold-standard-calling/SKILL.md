---
name: minimap2-alignment-gold-standard-calling
description: 'Use when benchmarking a nanopore read classifier (e.g. SPUMONI) and
  you need a per-read truth label set: align the same read set with minimap2 against
  the same references and take confident alignments (MAPQ >= 30) as ground truth.'
when_to_use_negative:
- The task is de novo alignment or variant calling rather than generating truth labels
  for a read-classification benchmark
- The reference contains substantial sequence similarity between target and non-target
  sequences (e.g. conserved genes between species), making MAPQ-based confidence unreliable
- Reads are too short or low-quality for minimap2 to produce confident MAPQ >= 30
  alignments, leaving most reads without gold-standard labels
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Read aligner used to generate the gold-standard per-read classifications (MAPQ
    >= 30) against single-reference and pan-genome references
- name: SPUMONI
  role: Classifier under test, scored by F1 against the minimap2-derived gold standard
    using MS/PML indexes
  repo: github.com/oma219/spumoni
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:b8109e8380c65c317baa03b7fe2f192e24682367d3e17d63f383b98768466f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# minimap2-alignment-gold-standard-calling

## Summary

Derives a trusted per-read classification gold standard by aligning base-called nanopore reads with minimap2 against both single-reference and pan-genome indexes, using a MAPQ >= 30 threshold on trimmed read suffixes, so that faster classifiers such as SPUMONI can be scored by F1 against it.

## When to use

Use when benchmarking a nanopore read classifier (e.g. SPUMONI) and you need a per-read truth label set: align the same read set with minimap2 against the same references and take confident alignments (MAPQ >= 30) as ground truth.

## When NOT to use

- The task is de novo alignment or variant calling rather than generating truth labels for a read-classification benchmark
- The reference contains substantial sequence similarity between target and non-target sequences (e.g. conserved genes between species), making MAPQ-based confidence unreliable
- Reads are too short or low-quality for minimap2 to produce confident MAPQ >= 30 alignments, leaving most reads without gold-standard labels

## Inputs

- Base-called Oxford Nanopore reads (FASTA/FASTQ), filtered to >= 4000 bp and truncated to first 720 bases
- Single-reference genome FASTA (e.g. ZymoBIOMICS mock community reference genomes)
- Pan-genome FASTA containing many strains per species
- minimap2 alignments (SAM/PAF) of the same reads against each reference

## Outputs

- Per-read gold-standard classification labels per reference (derived from MAPQ >= 30 alignments on trimmed suffixes)
- F1 scores for each classifier/reference combination (e.g. table3_zymo_benchmark.csv)
- Per-read classification throughput measurements

## How to apply

Take the same preprocessed read set used for the classifier under test (for the ZymoMC benchmark: ONT reads from SRX7711546, filtered to >= 4000 bp and truncated to their first 720 bases). Align these reads with minimap2 against both references under evaluation — the single-reference FASTA and the multi-strain pan-genome FASTA — so gold-standard labels are reference-specific. Call the gold-standard classification from the trimmed read suffixes, accepting only alignments with MAPQ >= 30; unaligned or low-MAPQ reads are treated as unclassified. Score the classifier under test (e.g. SPUMONI run with `spumoni run -r <index_prefix> -p reads.fa -P -c`) against these labels using F1 score, and record per-read classification throughput for the speed comparison. In the source study this showed SPUMONI reaching 97.94% F1 on the pan-genomic reference versus minimap2's own 98.73% F1, while SPUMONI ran about 11.9x–12x faster.

## Related tools

- **minimap2** (Read aligner used to generate the gold-standard per-read classifications (MAPQ >= 30) against single-reference and pan-genome references)
- **SPUMONI** (Classifier under test, scored by F1 against the minimap2-derived gold standard using MS/PML indexes) — github.com/oma219/spumoni

## Evaluation signals

- Every gold-standard label is backed by a minimap2 alignment with MAPQ >= 30 on the trimmed (first-720-base) read suffix
- F1 scores are reference-specific: the pan-genome gold standard yields SPUMONI ~97.94% vs minimap2 ~98.73% on real ZymoMC reads
- Throughput comparison shows SPUMONI roughly 11.9x–12x faster than minimap2 on the mock-community reads
- The identical preprocessed read set is used for both the classifier under test and the minimap2 gold standard, so label sets are directly comparable per read
- Classifier calls and gold-standard calls are aggregated into the same table format (F1 and throughput columns per reference)

## Limitations

- The gold standard is itself minimap2-derived, so any systematic minimap2 error is inherited as truth (circularity for the minimap2 arm of the comparison)
- The throughput advantage of SPUMONI over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- A single fixed confidence threshold (MAPQ >= 30) is applied rather than per-experiment tuning; the analogous SPUMONI threshold issue (one KS-stat threshold for all experiments) is noted by the authors

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [intro] the minimap2 read aligner (Li 2018) to align reads to a reference genome in real time: "the minimap2 read aligner (Li 2018) to align reads to a reference genome in real time"
- [intro] Payne et al. described the Readfish system (Payne et al., 2020) which combines an existing base caller with the minimap2 read aligner: "Payne et al. described the Readfish system (Payne et al., 2020) which combines an existing base caller with the minimap2 read aligner"
- [abstract] SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [readme] SPUMONI is a software tool for performing rapid read classifications on sequencing reads using a read's matching statistics: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
