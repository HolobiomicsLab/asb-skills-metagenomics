---
name: classification-accuracy-comparison
description: Use when comparing two read-classification or aligner tools on the same
  read set and you must verify that a faster method (e.g. PML-based streaming) does
  not degrade classification accuracy relative to a batch baseline.
when_to_use_negative:
- When the goal is base-level alignment or variant calling rather than read-level
  classification — SPUMONI/MONI produce matching statistics, not alignments, and SPUMONI
  has similar accuracy to minimap2 only for classification tasks
- When reads are raw nanopore electrical signal — SPUMONI operates on base-called
  sequences and requires a separate base caller upstream, so signal-level tools like
  UNCALLED are the right comparison
- When the reference contains substantial sequence similarity between depletion-target
  and non-target reads (e.g. conserved genes between species) — the null-index hypothesis-testing
  framework may be insufficient for accurate classification
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Online PML-based read classifier being benchmarked; run with -P for pseudomatching
    lengths and -c to write classification reports
  repo: https://github.com/oma219/spumoni
- name: MONI
  role: Batch MS-finding baseline whose matching statistics and wall-clock time serve
    as the accuracy/speed reference
  repo: https://github.com/maxrossi91/moni
- name: Docker
  role: Container runtime for pulling the pre-built SPUMONI image (oma219/spumoni:latest)
    used to run the comparison
- name: Singularity
  role: Alternative container runtime for pulling docker://oma219/spumoni:latest on
    HPC systems
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:c158bc96ba548e793e8948c60d163323c1d2abd04f878ecddc297e7827b6292e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - Docker
  - MONI
  - SPUMONI
  - Singularity
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# classification-accuracy-comparison

## Summary

Side-by-side benchmarking of classification accuracy (and speed) between two read-classification tools — here SPUMONI's online PML-finding versus MONI's batch MS-finding — run on identical read batches so that speed gains can be checked against accuracy preservation.

## When to use

Use when comparing two read-classification or aligner tools on the same read set and you must verify that a faster method (e.g. PML-based streaming) does not degrade classification accuracy relative to a batch baseline.

## When NOT to use

- When the goal is base-level alignment or variant calling rather than read-level classification — SPUMONI/MONI produce matching statistics, not alignments, and SPUMONI has similar accuracy to minimap2 only for classification tasks
- When reads are raw nanopore electrical signal — SPUMONI operates on base-called sequences and requires a separate base caller upstream, so signal-level tools like UNCALLED are the right comparison
- When the reference contains substantial sequence similarity between depletion-target and non-target reads (e.g. conserved genes between species) — the null-index hypothesis-testing framework may be insufficient for accurate classification

## Inputs

- Reference genome FASTA(s) for building the pan-genome index (e.g. ZymoMC mock community references or many-strain human genome references)
- Sequencing read batches in FASTA/FASTQ (e.g. ZymoMC nanopore read batches, base-called sequences)
- Pre-built SPUMONI index containing both MS (-M) and PML (-P) components
- MONI batch MS-finding tool output (matching statistics per read)

## Outputs

- Classification report files from SPUMONI (-c) and classification results from MONI
- Wall-clock time and throughput (reads/s and bp/s) per tool per read batch
- Tabulated side-by-side comparison of time, throughput, and classification accuracy (MONI vs SPUMONI)
- Validation verdict on whether the ~3x speedup holds with similar-or-greater accuracy

## How to apply

Build the same pan-genome index with both MS (-M) and PML (-P) structures so both tools can be compared fairly, e.g. './spumoni build -r ref.fa -M -P -m -o /path/to/index'. Run MONI in batch MS-finding mode on each read batch and SPUMONI on the identical batches with './spumoni run -r /path/to/index -p reads.fa -P -c', where -P enables online pseudomatching-length computation and -c writes classifications to a report file. For each tool record total wall-clock time and compute throughput (reads/s and bp/s), then extract classification accuracy from the report files and tabulate time, throughput, and accuracy side by side. Validate that the SPUMONI PML-based throughput is approximately 3x that of MONI (the paper's reported ~3x speedup) while classification accuracy is similar or greater; the paper's broader benchmark showed SPUMONI has similar accuracy to minimap2 while being about 12 times faster with a one-sixteenth-sized index. Because both tools consume the same FASTA/FASTQ read batches against the same index, any accuracy difference is attributable to the PML vs MS statistic rather than input differences.

## Related tools

- **SPUMONI** (Online PML-based read classifier being benchmarked; run with -P for pseudomatching lengths and -c to write classification reports) — https://github.com/oma219/spumoni
- **MONI** (Batch MS-finding baseline whose matching statistics and wall-clock time serve as the accuracy/speed reference) — https://github.com/maxrossi91/moni
- **Docker** (Container runtime for pulling the pre-built SPUMONI image (oma219/spumoni:latest) used to run the comparison)
- **Singularity** (Alternative container runtime for pulling docker://oma219/spumoni:latest on HPC systems)

## Examples

```
./spumoni build -r ref.fa -M -P -m -o /path/to/index && ./spumoni run -r /path/to/index -p reads.fa -P -c
```

## Evaluation signals

- Measured SPUMONI PML throughput is approximately 3x MONI's throughput (reads/s and bp/s) on the identical read batches
- Classification accuracy extracted from SPUMONI report files is similar to or greater than MONI's accuracy on the same reads
- Both tools were run against the same index built with both -M (MS) and -P (PML) components, ensuring a fair comparison
- Wall-clock times and throughput are reported separately for each read batch, not pooled across batches
- As a sanity cross-check, SPUMONI should also show accuracy similar to minimap2 on many-strain indexes (with ~12x speedup) in the mock community depletion scenario

## Limitations

- SPUMONI operates on base-called sequences, not the raw electrical signal batches delivered by the Read Until API, so it cannot yet directly control a nanopore sequencer
- A single KS-stat threshold is used for all experiments; the optimal threshold is expected to be a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species
- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- No specific datasets or accessions are named in the methods, and the SPUMONI version number is not stated

## Evidence

- [intro] This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data: "This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data"
- [intro] SPUMONI has similar accuracy as minimap2 but is about 12 times faster. Moreover, its many-strain index is about one-sixteenth the size of minimap2's: "SPUMONI has similar accuracy as minimap2 but is about 12 times faster. Moreover, its many-strain index is about one-sixteenth the size of minimap2's"
- [discussion] it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [methods] The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use: "The easier way would be to use the Docker/Singularity images which have SPUMONI pre-installed and ready to use"
