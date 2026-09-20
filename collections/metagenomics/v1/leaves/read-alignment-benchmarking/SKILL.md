---
name: read-alignment-benchmarking
description: Use when comparing read-classification accuracy (F1, precision, recall)
  of a fast classifier like SPUMONI against an aligner like minimap2 across reference
  index compositions (target strain present vs excluded) on simulated or real nanopore
  reads.
when_to_use_negative:
- Reads are raw nanopore electrical signal rather than base-called sequences — SPUMONI
  operates on base-called sequences and cannot interface with the Read Until API directly
- Substantial sequence similarity exists between depletion-target and non-target reads
  (e.g. conserved genes between species), where the null-index hypothesis test may
  be insufficient
- The task requires per-experiment threshold optimization rather than a single KS-stat
  threshold tuned on real nanopore data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SPUMONI
  role: Classifies reads as target/non-target using matching statistics (MS) and pseudo-matching
    lengths (PML) via pan-genome indexes built with spumoni build
  repo: github.com/oma219/spumoni
- name: minimap2
  role: Reference aligner producing comparator target/non-target calls against each
    reference strategy for accuracy benchmarking
- name: PBSIM2
  role: Simulates nanopore reads (e.g. 90% accuracy ZymoMC reads) used as the benchmark
    query set
  repo: yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:aa4e6c0238582c3021e2ff368d939660c52e2d3b3a0dfbd1c45be1720be0939a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PBSIM2
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# read-alignment-benchmarking

## Summary

Benchmark read-classification accuracy of SPUMONI (PML and MS modes) against minimap2 across reference index strategies, computing precision, recall, and F1 scores on simulated ZymoMC nanopore reads to reproduce the paper's Table 1 comparison of one-genome versus pan-genome indexes.

## When to use

Use when comparing read-classification accuracy (F1, precision, recall) of a fast classifier like SPUMONI against an aligner like minimap2 across reference index compositions (target strain present vs excluded) on simulated or real nanopore reads.

## When NOT to use

- Reads are raw nanopore electrical signal rather than base-called sequences — SPUMONI operates on base-called sequences and cannot interface with the Read Until API directly
- Substantial sequence similarity exists between depletion-target and non-target reads (e.g. conserved genes between species), where the null-index hypothesis test may be insufficient
- The task requires per-experiment threshold optimization rather than a single KS-stat threshold tuned on real nanopore data

## Inputs

- PBSIM2-simulated nanopore reads in FASTA from ZymoMC references at 90% base-call accuracy
- Reference FASTA files for One Genome and Pan-genome strategies, each with and without Zymo mock references
- SPUMONI index prefixes (MS and PML indexes) built per reference strategy

## Outputs

- SPUMONI classification calls (target/non-target) per read for PML and MS modes across four reference strategies
- minimap2 alignment-derived target/non-target calls per read for each reference strategy
- table1_f1_scores.csv with precision, recall, and F1 scores for each tool x reference-strategy combination

## How to apply

Simulate long nanopore reads (e.g. PBSIM2 at 90% accuracy from ZymoMC references), then build four SPUMONI indexes with 'spumoni build -r <ref>.fa -M -P -m -o <prefix>' representing One Genome and Pan-genome strategies, each with and without the Zymo mock references; this constructs both matching-statistics (MS) and pseudo-matching-lengths (PML) indexes. Classify reads with 'spumoni run' in PML mode (-P) and MS mode (-M), labeling each read as target or non-target using the null-index KS-statistic hypothesis-testing threshold. Align the same reads with minimap2 against each reference strategy to obtain comparator target/non-target calls. Compute precision, recall, and F1 for each tool x reference-strategy combination and assemble into table1_f1_scores.csv; expect pan-genome indexes excluding the target strain to yield F1 near 99.7% for both tools, comparable to indexes containing the exact target strain.

## Related tools

- **SPUMONI** (Classifies reads as target/non-target using matching statistics (MS) and pseudo-matching lengths (PML) via pan-genome indexes built with spumoni build) — github.com/oma219/spumoni
- **minimap2** (Reference aligner producing comparator target/non-target calls against each reference strategy for accuracy benchmarking)
- **PBSIM2** (Simulates nanopore reads (e.g. 90% accuracy ZymoMC reads) used as the benchmark query set) — yukiteruono/pbsim2

## Examples

```
spumoni build -r pangenome.fa -M -P -m -o pangenome_idx && spumoni run -r pangenome_idx -p reads.fa -P -c
```

## Evaluation signals

- Pan-genome index excluding the depletion-target strain yields F1 of ~99.7% for both SPUMONI and minimap2 (e.g. 99.74 for SPUMONI vs 99.74 for minimap2 on Pan-genome without mock refs)
- One-genome strategy without mock references reproduces reported F1 values of 93.13 (SPUMONI) and 95.00 (minimap2)
- Classification labels (target/non-target) are derivable for every read via the null-index KS-statistic hypothesis-testing threshold
- Output CSV (table1_f1_scores.csv) contains precision, recall, and F1 for the full 3 tools (SPUMONI-PML, SPUMONI-MS, minimap2) x 4 reference-strategy grid
- All four indexes are built with both MS and PML structures, as spumoni build with -M and -P constructs both index types

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold depends on read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may fail when there is substantial sequence similarity between depletion-target and non-target reads, such as conserved genes between species
- Throughput advantage over minimap2 is limited for human reads; it is expected to improve only as the number of human genomes in the reference increases
- SPUMONI version number is not stated in the methods

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [methods] spumoni build constructs both index types: [evidence span withheld — anchor status: not_found]
- [discussion] null-index KS threshold caveat: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [discussion] null index similarity failure mode: "if there is substantial sequence similarity between depletion-target reads and reads that should not be targeted for depletion"
