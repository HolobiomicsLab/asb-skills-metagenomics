---
name: pseudomatching-length-online-querying
description: Use when reads must be classified against a large, repetitive multi-strain
  reference in streaming/online fashion, and batch MS-finding (MONI) is too slow or
  unsuited to streaming input.
when_to_use_negative:
- Raw nanopore electrical signal must be classified directly via the Read Until API
  — SPUMONI operates on base-called sequences, so a separate base caller is required
  upstream
- The task requires alignment-level output or strain-level assembly rather than read
  classification by matching statistics (minimap2 may be more appropriate)
- Substantial sequence similarity exists between depletion-target reads and non-target
  reads (e.g., conserved genes between species) — the reversed-sequence null index
  may be insufficient
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SPUMONI
  role: Online PML-finding and read classification against the pan-genome index; run
    with 'spumoni run ... -P -c'
  repo: https://github.com/oma219/spumoni
- name: MONI
  role: Batch MS-finding baseline against which the ~3x PML speedup is measured
  repo: https://github.com/maxrossi91/moni
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:5f4390d808c3af2c6b3a392d37a3cf018288ce325126531141e52696f778d57e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - Docker
  - MONI
  - SPUMONI
  - Singularity
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pseudomatching-length-online-querying

## Summary

Compute pseudomatching lengths (PMLs) online against a pan-genome r-index with SPUMONI, replacing MONI's batch matching-statistics (MS) computation with a faster streaming procedure (~3x faster) for read classification in targeted nanopore sequencing.

## When to use

Use when reads must be classified against a large, repetitive multi-strain reference in streaming/online fashion, and batch MS-finding (MONI) is too slow or unsuited to streaming input.

## When NOT to use

- Raw nanopore electrical signal must be classified directly via the Read Until API — SPUMONI operates on base-called sequences, so a separate base caller is required upstream
- The task requires alignment-level output or strain-level assembly rather than read classification by matching statistics (minimap2 may be more appropriate)
- Substantial sequence similarity exists between depletion-target reads and non-target reads (e.g., conserved genes between species) — the reversed-sequence null index may be insufficient

## Inputs

- Reference FASTA(s) for building the pan-genome index
- Base-called sequencing reads (e.g., ZymoMC mock-community nanopore reads) in FASTA/FASTQ
- Pre-built SPUMONI index directory containing both MS (-M) and PML (-P) components

## Outputs

- Per-read classification report file (via -c)
- Pseudomatching lengths (PMLs) per query read computed online
- Wall-clock time and throughput (reads/s, bp/s) measurements for benchmarking

## How to apply

Build a pan-genome index with both MS and PML components using './spumoni build -r ref.fa -M -P -m -o /path/to/index', where -M adds the matching-statistics index and -P adds the PML index. Then run classification online with './spumoni run -r /path/to/index -p reads.fa -P -c', which replaces MONI's batch MS-finding with the optimized online PML algorithm and writes classifications to a report file via -c. Record wall-clock time and throughput (reads/s, bp/s) and compare against MONI's batch MS-finding on identical read batches. Rationale: longer matching statistics indicate a better approximate match to the index, and SPUMONI augments this with a null index plus hypothesis testing (KS-stat threshold) for principled ejection decisions; the reported result is ~3x speedup over MONI with similar or greater accuracy.

## Related tools

- **SPUMONI** (Online PML-finding and read classification against the pan-genome index; run with 'spumoni run ... -P -c') — https://github.com/oma219/spumoni
- **MONI** (Batch MS-finding baseline against which the ~3x PML speedup is measured) — https://github.com/maxrossi91/moni

## Examples

```
./spumoni run -r /path/to/index -p reads.fa -P -c
```

## Evaluation signals

- Measured SPUMONI PML-based throughput should be approximately 3x that of MONI's batch MS-finding on identical read batches
- Classification accuracy from SPUMONI's report files should be similar to or greater than MONI's
- Index build must include both -M (MS) and -P (PML) components; confirm both index files exist before running
- Classification accuracy should be comparable to minimap2 against an index containing many strains per species (~12x faster, index ~1/16 the size)
- Report file (via -c) must contain per-read classifications consistent with the KS-stat hypothesis-testing threshold

## Limitations

- SPUMONI operates on base-called sequences, not the batches of signal delivered by the Read Until API, and cannot yet directly interact with the Read Until API as control software
- A single KS-stat threshold is used for all experiments; the optimal threshold is expected to be a function of read accuracy and the reference used, with no per-experiment thresholding method provided
- The null index (reverse of positive-index sequences) may be insufficient when depletion-target reads share substantial similarity with non-target reads, e.g., conserved genes between species
- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases

## Evidence

- [intro] This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data: "This optimized PML-finding procedure makes SPUMONI about 3 times faster than MONI, while achieving similar (often greater) accuracy and allowing it to operate on streaming data"
- [intro] SPUMONI replaces MONI's batch MS-finding with a faster online algorithm: [evidence span withheld — anchor status: not_found]
- [abstract] SPUMONI uses a compressed index to rapidly generate exact or approximate matching statistics in a streaming fashion: "SPUMONI uses a compressed index to rapidly generate exact or approximate matching statistics in a streaming fashion."
- [intro] SPUMONI null index and hypothesis testing for ejection choices: [evidence span withheld — anchor status: not_found]
- [methods] docker pull oma219/spumoni:latest: "docker pull oma219/spumoni:latest"
- [readme] SPUMONI performs rapid read classifications using matching statistics or pseudo-matching lengths: "SPUMONI is a software tool for **performing rapid read classifications on sequencing reads using a read's matching statistics (or a related quantity called pseudo-matching lengths).**"
