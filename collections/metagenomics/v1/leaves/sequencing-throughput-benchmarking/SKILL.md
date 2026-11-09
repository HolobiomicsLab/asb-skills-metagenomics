---
name: sequencing-throughput-benchmarking
description: Use when comparing classification speed and accuracy of a lightweight
  read classifier against an aligner baseline on real nanopore reads, with a gold
  standard derived from the baseline under a MAPQ threshold.
when_to_use_negative:
- The goal is direct interaction with the nanopore Read Until API — SPUMONI operates
  on base-called sequences, not raw electrical signal batches
- The reference contains few genomes (e.g., single human genome), where the throughput
  advantage over minimap2 is limited and only expected to improve as the number of
  genomes in the reference increases
- Input reads are already classified by a trusted method and no speed/accuracy comparison
  is needed
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Read classifier under test; builds MS and PML pan-genome indexes and reports
    per-read classifications with the -c flag
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Aligner baseline for accuracy comparison and source of the gold-standard calls
    under MAPQ >= 30
- name: PBSIM
  role: Long-read simulator (PacBio CLR and Nanopore) usable to generate simulated
    benchmarks alongside real-read tests
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:10918a87e2ffae47e0ad574b999b16453f1a23678dfe3d5864b08fdd27258a0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - PBSIM
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# sequencing-throughput-benchmarking

## Summary

Benchmark per-read classification throughput and accuracy (F1) of a fast read classifier like SPUMONI against an aligner baseline like minimap2 on the same preprocessed read set, using an independently derived gold standard. This skill captures the paper's protocol for showing that SPUMONI is ~11.9-12x faster than minimap2 with comparable or better F1 on pan-genomic indexes.

## When to use

Use when comparing classification speed and accuracy of a lightweight read classifier against an aligner baseline on real nanopore reads, with a gold standard derived from the baseline under a MAPQ threshold.

## When NOT to use

- The goal is direct interaction with the nanopore Read Until API — SPUMONI operates on base-called sequences, not raw electrical signal batches
- The reference contains few genomes (e.g., single human genome), where the throughput advantage over minimap2 is limited and only expected to improve as the number of genomes in the reference increases
- Input reads are already classified by a trusted method and no speed/accuracy comparison is needed

## Inputs

- Real Oxford Nanopore base-called reads (SRA SRX7711546, ZymoBIOMICS mock community ZymoMC set)
- Single-reference and pan-genome FASTA reference files (many strains per species)
- minimap2 alignments (SAM/PAF) against both references for gold-standard derivation

## Outputs

- Per-tool, per-reference F1 scores versus the MAPQ >= 30 gold standard
- Per-read classification throughput measurements
- Benchmark comparison table (table3_zymo_benchmark.csv)

## How to apply

Preprocess reads identically for both tools (e.g., retain ZymoMC ONT reads of at least 4000 bp from SRA SRX7711546 and truncate each read to its first 720 bases). Build the classifier's indexes for both a single-reference and a pan-genome (many strains per species) FASTA using `spumoni build -r <ref>.fa -M -P -m -o <index_prefix>`, which constructs matching-statistics (MS) and pseudo-matching-length (PML) indexes. Run classification with `spumoni run -r <index_prefix> -p reads.fa -P -c` and align the same reads with minimap2 against both references. Derive gold-standard calls from trimmed read suffixes using minimap2 with a MAPQ >= 30 threshold, then score both tools by F1 against that gold standard and measure per-read classification throughput. Judge correctness by checking the expected outcome: SPUMONI ~11.9-12x faster with F1 essentially matching minimap2 (97.94% vs 98.73% on the pan-genome index).

## Related tools

- **SPUMONI** (Read classifier under test; builds MS and PML pan-genome indexes and reports per-read classifications with the -c flag) — https://github.com/oma219/spumoni
- **minimap2** (Aligner baseline for accuracy comparison and source of the gold-standard calls under MAPQ >= 30)
- **PBSIM** (Long-read simulator (PacBio CLR and Nanopore) usable to generate simulated benchmarks alongside real-read tests) — https://github.com/yukiteruono/pbsim2

## Examples

```
spumoni run -r <index_prefix> -p reads.fa -P -c
```

## Evaluation signals

- F1 score against the minimap2-derived gold standard (MAPQ >= 30) should be comparable to the baseline: SPUMONI 97.94% vs minimap2 98.73% on the pan-genomic ZymoMC reference
- Throughput ratio should show SPUMONI roughly 11.9-12x faster than minimap2 per read
- Both tools must be scored on the identical preprocessed read set (reads >= 4000 bp truncated to first 720 bases)
- Pan-genome index results should show SPUMONI matching or beating minimap2 F1, consistent with 'similar accuracy as minimap2 when both are run against an index containing many strains per species'
- Output table (table3_zymo_benchmark.csv) should contain F1 and throughput values for both tools under both single-reference and pan-genome conditions

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold is a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- The throughput advantage over minimap2 is limited for human reads and is expected to improve only as the number of human genomes in the reference increases
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [abstract] However SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [abstract] SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively.: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] we expect our throughput advantage to improve as the number of human genomes in the reference increases: "we expect our throughput advantage to improve as the number of human genomes in the reference increases"
- [readme] git clone https://github.com/oma219/spumoni: "git clone https://github.com/oma219/spumoni"
