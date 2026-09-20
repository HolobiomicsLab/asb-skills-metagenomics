---
name: clustering-tool-command-line-parameterization
description: Use when clustering a protein or nucleotide FASTA set to a target sequence
  identity (e.g. 50%, 70%, 90%) and the tool invocation parameters must be specified,
  tuned, and logged for reproducible runtime or cluster-quality comparisons.
when_to_use_negative:
- The input is not a raw sequence set (e.g. already-clustered representatives or a
  feature/abundance table) and no clustering step is needed
- An exact full sensitivity alignment search is required rather than fast approximate
  redundancy reduction at a fixed identity threshold
- The dataset is small enough that clustering runtime is irrelevant and parameter
  benchmarking adds no value
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Primary clustering tool run via `mmseqs linclust` with --cov-mode 1 -c 0.9
    and --min-seq-id set to the target level (0.9/0.5); its flags define coverage
    and identity semantics
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite containing Linclust; provides the linclust subcommand and the
    clustering workflow
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool parameterized with -T 16 -M 0 and word-size/identity
    pairs (-n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5)
- name: UCLUST
  role: Baseline clustering tool parameterized with --id 0.9 or --id 0.5 for the corresponding
    identity levels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:c467e55eb518ce26ca944107787be13fcdcea397e770671c288d831b9d425af7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# clustering-tool-command-line-parameterization

## Summary

Selects and records the exact command-line parameters (identity thresholds, coverage mode, word sizes, thread/memory flags) needed to run sequence clustering tools — Linclust/MMseqs2, CD-HIT, UCLUST — at defined sequence identity levels so that benchmark or production runs are reproducible.

## When to use

Use when clustering a protein or nucleotide FASTA set to a target sequence identity (e.g. 50%, 70%, 90%) and the tool invocation parameters must be specified, tuned, and logged for reproducible runtime or cluster-quality comparisons.

## When NOT to use

- The input is not a raw sequence set (e.g. already-clustered representatives or a feature/abundance table) and no clustering step is needed
- An exact full sensitivity alignment search is required rather than fast approximate redundancy reduction at a fixed identity threshold
- The dataset is small enough that clustering runtime is irrelevant and parameter benchmarking adds no value

## Inputs

- Protein or nucleotide sequence set in FASTA format (e.g. UniProt 2016_03 release, ~123 million sequences)
- Target clustering identity level (e.g. 0.9, 0.7, 0.5)
- Installed tool versions: Linclust/MMseqs2 (commit 5e21868), CD-HIT 4.6, UCLUST

## Outputs

- Clustering result files (cluster membership per tool) at the specified identity level
- runtime_benchmark.tsv with per-tool runtimes and Linclust speedup factors
- Logged command lines with all flags for reproducibility

## How to apply

Map the target sequence identity to each tool's parameters: for Linclust run `mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9` (or --min-seq-id 0.5 for the 50% level); for CD-HIT 4.6 use `-T 16 -M 0` with matched word-size/identity pairs (-n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5); for UCLUST use `--id 0.9` / `--id 0.5`. Time each run with the Linux 'time' command and write runtimes plus computed speedup factors to a TSV (e.g. runtime_benchmark.tsv). The rationale is that fair comparison across tools requires equivalent identity and coverage semantics — Linclust's coverage mode and CD-HIT's word-size choices encode how much of each sequence must align and how similar a match must be — and that recording the exact commit (e.g. Linclust/MMseqs2 commit 5e21868) and flags makes order-of-magnitude runtime results (Linclust ~2300x faster than UCLUST, ~4600x faster than CD-HIT at 50% identity) reproducible.

## Related tools

- **Linclust** (Primary clustering tool run via `mmseqs linclust` with --cov-mode 1 -c 0.9 and --min-seq-id set to the target level (0.9/0.5); its flags define coverage and identity semantics) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite containing Linclust; provides the linclust subcommand and the clustering workflow) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool parameterized with -T 16 -M 0 and word-size/identity pairs (-n 5 -c 0.9, -n 4 -c 0.7, -n 3 -c 0.5))
- **UCLUST** (Baseline clustering tool parameterized with --id 0.9 or --id 0.5 for the corresponding identity levels)

## Examples

```
mmseqs linclust input.fasta result.tsv tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- Reproduced runtimes yield the paper's order-of-magnitude speedups: Linclust ~2300x faster than UCLUST and ~4600x faster than CD-HIT at 50% identity, ~100x and ~62x respectively at 90% identity
- Every tool invocation records the full flag set, and Linclust runs use commit 5e21868 with --cov-mode 1 -c 0.9 and --min-seq-id matching the target level
- CD-HIT word sizes match the identity level per the paper's mapping (-n 5 for 0.9, -n 4 for 0.7, -n 3 for 0.5)
- All runs are timed with the Linux 'time' command and runtime_benchmark.tsv contains one row per tool per identity threshold

## Limitations

- K-mer prefilter-based clustering (CD-HIT, UCLUST, kclust, MMseqs, Linclust's prefilter) is approximate; if falsely clustered pairs are costly, pairwise links must be verified with the exact similarity measure, which is quadratic in time
- Canopy-style preclustering's exact distance calculation O(N^2/F) is still quadratic in N
- Linclust requires that a set of m keys per item can be defined satisfying shared-key conditions, restricting generalizability
- Linclust chooses the longest sequence as cluster representative via greedy incremental clustering, which may not match other tools' representative-selection rules

## Evidence

- [methods] Linclust/MMseqs2 parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [methods] benchmark scripts availability: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available"
- [discussion] approximation limitation: "If falsely clustered pairs are costly (e.g., for redundancy filtering), pairwise links need to be verified with the exact similarity measure, which still requires quadratic time complexity."
- [discussion] greedy clustering rule: "We perform the clustering in step 5 of Fig. 1 with the greedy incremental clustering, because it always chooses the longest sequence as the cluster representative."
