---
name: binary-classification-f1-scoring
description: Use when comparing read classifiers on nanopore reads where a gold standard
  must be derived from minimap2 alignments (MAPQ >= 30) and classification accuracy
  must be reported as F1 on target vs non-target calls.
when_to_use_negative:
- When the input is raw nanopore electrical signal rather than basecalled sequences
  — SPUMONI operates on sequences of bases and requires a separate base caller upstream
- When evaluating on human reads with few reference genomes, where SPUMONI's throughput
  advantage over minimap2 is limited and expected to improve only as more human genomes
  are added to the reference
- When reads share substantial sequence similarity between target and non-target organisms
  (e.g. conserved genes between species), where the reverse-sequence null index may
  be insufficient
tools:
- name: SPUMONI
  role: Classifies basecalled reads as target/non-target using matching statistics
    (MS) and pseudo-matching-length (PML) indexes built with `spumoni build -r -M
    -P -m`; run with `-P -c` to emit per-read classification reports.
  repo: github.com/oma219/spumoni
- name: minimap2
  role: Aligns the same trimmed reads to both references to derive the gold-standard
    classification calls (MAPQ >= 30) and to serve as the accuracy and throughput
    comparison baseline.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:8aa0f7d640472907528ae36c393f3eba02e6f29e8505b266977733ac13d2ee4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPUMONI
  - minimap2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binary-classification-f1-scoring

## Summary

Benchmark per-read binary target/non-target classification calls (e.g. SPUMONI vs minimap2) by scoring F1 against an alignment-derived gold standard, as done for the ZymoBIOMICS nanopore mock-community benchmark (Table 3).

## When to use

Use when comparing read classifiers on nanopore reads where a gold standard must be derived from minimap2 alignments (MAPQ >= 30) and classification accuracy must be reported as F1 on target vs non-target calls.

## When NOT to use

- When the input is raw nanopore electrical signal rather than basecalled sequences — SPUMONI operates on sequences of bases and requires a separate base caller upstream
- When evaluating on human reads with few reference genomes, where SPUMONI's throughput advantage over minimap2 is limited and expected to improve only as more human genomes are added to the reference
- When reads share substantial sequence similarity between target and non-target organisms (e.g. conserved genes between species), where the reverse-sequence null index may be insufficient

## Inputs

- Oxford Nanopore basecalled reads (SRA SRX7711546, ZymoMC), filtered to reads >= 4000 bp and truncated to first 720 bases (FASTA)
- Single-reference and pan-genome reference FASTA files
- minap2 alignments of the same reads with MAPQ values (gold standard source)
- Per-read classification report files from SPUMONI (via -c flag)

## Outputs

- Per-read binary target/non-target calls for each classifier and reference
- Precision, recall and F1 score per classifier per reference (e.g. table3_zymo_benchmark.csv)
- Per-read classification throughput measurements

## How to apply

Derive gold-standard labels from minimap2 alignments of the same trimmed reads against the same references, keeping only alignments with MAPQ >= 30 from trimmed read suffixes. Run SPUMONI classification with the `-c` flag so read classifications are written to a report file for each reference (single-reference and pan-genome indexes built with `spumoni build -r <ref>.fa -M -P -m -o <index_prefix>`, producing both MS and PML indexes). Convert each tool's per-read output into binary target/non-target calls, compare against the gold standard, and compute precision, recall and F1 per reference; also measure per-read classification throughput. Expected outcome on real ZymoMC reads: SPUMONI F1 ~97.94% vs minimap2 ~98.73% on the pan-genome index, with SPUMONI roughly 11.9x faster.

## Related tools

- **SPUMONI** (Classifies basecalled reads as target/non-target using matching statistics (MS) and pseudo-matching-length (PML) indexes built with `spumoni build -r -M -P -m`; run with `-P -c` to emit per-read classification reports.) — github.com/oma219/spumoni
- **minimap2** (Aligns the same trimmed reads to both references to derive the gold-standard classification calls (MAPQ >= 30) and to serve as the accuracy and throughput comparison baseline.)

## Examples

```
spumoni run -r <index_prefix> -p reads.fa -P -c
```

## Evaluation signals

- Gold standard is restricted to minimap2 alignments with MAPQ >= 30 derived from trimmed read suffixes; verify no lower-MAPQ alignments leak into the labels
- On the pan-genome index with real ZymoMC reads, SPUMONI's F1 should land near 97.94%, essentially matching minimap2's 98.73%
- SPUMONI's per-read throughput should be roughly 11.9x faster than minimap2 on the ZymoMC benchmark (paper claims ~12x)
- Reads were preprocessed to >= 4000 bp then truncated to their first 720 bases; confirm the same preprocessed reads.fa is fed to both tools
- The F1 table (table3_zymo_benchmark.csv) must contain both F1 and throughput values per classifier per reference (single-reference and pan-genome)

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold depends on read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species
- Throughput advantage over minimap2 is limited for human reads; improvement is expected only as the number of human genomes in the reference increases
- SPUMONI version number is not stated in the methods (docker pull oma219/spumoni:latest)

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [abstract] SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [discussion] null index insufficiency caveat: "if there is substantial sequence similarity between depletion-target reads and reads that should not be targeted for depletion"
- [discussion] SPUMONI operates on basecalled sequences: "it operates on sequences of bases opposed to the batches of signal that are delivered by the Read Until API"
- [methods] SPUMONI build step: "git clone https://github.com/oma219/spumoni"
