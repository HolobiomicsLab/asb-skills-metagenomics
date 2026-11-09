---
name: f1-score-computation
description: 'Use when benchmarking read classifiers in a mock-community depletion
  setting: you have per-read target/non-target calls from SPUMONI or minimap2 over
  multiple index strategies and need F1 scores comparable to the paper''s Table 1.'
when_to_use_negative:
- The task is throughput, index-size, or memory benchmarking rather than classification
  accuracy — the paper measures speed and RSS separately from F1.
- The index contains only a single reference genome and reads have very low base-call
  accuracy; the paper shows one-genome indexes drop to ~93-95 F1 on 90%-accuracy reads,
  so F1 comparisons against pan-genome results would be confounded.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SPUMONI
  role: Produces target/non-target read classifications in PML (-P) and MS (-M) modes
    against pan-genome and one-genome indexes; its calls feed the precision/recall/F1
    computation.
  repo: github.com/oma219/spumoni
- name: minimap2
  role: Baseline aligner whose target/non-target calls against each reference strategy
    are compared to SPUMONI's classifications via the same precision/recall/F1 computation.
- name: PBSIM2
  role: Simulates the 90%-accuracy ZymoMC nanopore reads used as the classified input
    for the F1 benchmark.
  repo: github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:dc1de6c55e940f4798f5462db2159ef09bb64e91abc1aa7825634999de4ed594
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PBSIM2
  - SPUMONI
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# f1-score-computation

## Summary

Compute precision, recall, and F1 score for read-classification tools (SPUMONI-PML, SPUMONI-MS, minimap2) across reference-strategy combinations, to quantify how well a pan-genome index lacking the exact depletion-target strain preserves targeted-nanopore read classification accuracy compared to an index containing the target strain.

## When to use

Use when benchmarking read classifiers in a mock-community depletion setting: you have per-read target/non-target calls from SPUMONI or minimap2 over multiple index strategies and need F1 scores comparable to the paper's Table 1.

## When NOT to use

- The task is throughput, index-size, or memory benchmarking rather than classification accuracy — the paper measures speed and RSS separately from F1.
- The index contains only a single reference genome and reads have very low base-call accuracy; the paper shows one-genome indexes drop to ~93-95 F1 on 90%-accuracy reads, so F1 comparisons against pan-genome results would be confounded.

## Inputs

- PBSIM2-simulated ZymoMC nanopore reads at 90% base-call accuracy (FASTA/FASTQ)
- SPUMONI classification outputs (PML mode via -P and MS mode via -M) for each of four index strategies
- minimap2 alignment calls of the same reads against each reference strategy
- Null-index KS-statistic hypothesis-testing threshold used to bin reads as target or non-target

## Outputs

- Per-read target/non-target classification labels for SPUMONI-PML, SPUMONI-MS, and minimap2 under each reference strategy
- Precision, recall, and F1 score per tool x reference-strategy combination
- table1_f1_scores.csv with F1 scores comparable to the paper's Table 1

## How to apply

Build a SPUMONI index for each reference strategy (One Genome and Pan-genome, each with and without the Zymo mock references) using 'spumoni build -r <ref>.fa -M -P -m -o <prefix>', which constructs both a matching-statistics (MS) index and a pseudo-matching-lengths (PML) index. Run 'spumoni run -r <index_prefix> -p reads.fa -P -c' for PML mode and repeat with -M for MS mode on PBSIM2-simulated 90%-accuracy ZymoMC reads. Bin each read as target or non-target from the SPUMONI statistics using the null-index KS-statistic hypothesis-testing threshold, and derive minimap2 target/non-target calls by aligning the same reads against each reference strategy. From the resulting calls, compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 = 2*precision*recall/(precision+recall) for each tool x reference-strategy combination. Assemble the F1 scores into table1_f1_scores.csv and compare against the reported values, e.g. SPUMONI 99.74 vs minimap2 99.74 on the pan-genome index without mock references, and 93.13/95.00 on the one-genome index without mock references.

## Related tools

- **SPUMONI** (Produces target/non-target read classifications in PML (-P) and MS (-M) modes against pan-genome and one-genome indexes; its calls feed the precision/recall/F1 computation.) — github.com/oma219/spumoni
- **minimap2** (Baseline aligner whose target/non-target calls against each reference strategy are compared to SPUMONI's classifications via the same precision/recall/F1 computation.)
- **PBSIM2** (Simulates the 90%-accuracy ZymoMC nanopore reads used as the classified input for the F1 benchmark.) — github.com/yukiteruono/pbsim2

## Examples

```
spumoni run -r <index_prefix> -p reads.fa -P -c
```

## Evaluation signals

- F1 for SPUMONI and minimap2 on the pan-genome index without mock references should both reach ~99.74.
- F1 on the one-genome index without mock references should fall to ~93.13 (one tool) and ~95.00 (the other), reproducing the paper's reported values.
- table1_f1_scores.csv exists and contains one precision/recall/F1 entry per tool (SPUMONI-PML, SPUMONI-MS, minimap2) x reference-strategy (4) combination.
- Target/non-target binning is consistent with a single KS-stat threshold from the null-index hypothesis test, as the paper uses the same threshold across experiments.

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold is a function of read accuracy and the reference used, and no per-experiment thresholding method is provided.
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads, e.g. conserved genes between species.
- SPUMONI version number is not stated in the methods, complicating exact reproduction.

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [methods] ## Step 2: Running Classification: "## Step 2: Running Classification"
- [methods] ## Step 1: Building an Index: "## Step 1: Building an Index"
- [discussion] it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy.: "SPUMONI is well positioned for these trends because it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy."
