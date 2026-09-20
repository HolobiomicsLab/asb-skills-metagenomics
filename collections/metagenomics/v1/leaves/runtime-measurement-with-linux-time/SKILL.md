---
name: runtime-measurement-with-linux-time
description: Use when comparing the computational scalability or speed of clustering/search
  tools on large sequence sets, or when reproducing reported runtime scaling exponents
  and speedup factors.
when_to_use_negative:
- When comparing clustering quality (e.g. functional consistency via GO annotations
  or Pfam) rather than runtime — quality metrics, not timing, are the relevant outputs.
- When the tools cannot be run on identical hardware with identical thread counts
  (e.g. CD-HIT -T 16 vs single-threaded baselines), since runtime ratios are then
  confounded.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Clustering tool under benchmark; expected to show linear N^~1 scaling via
    its O(mN) algorithm
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Suite hosting Linclust; run as mmseqs cluster / mmseqs linclust at fixed identity
    and coverage parameters
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool timed for comparison (fitted exponent N^2.75 at 90%
    identity)
- name: UCLUST
  role: Baseline clustering tool timed for comparison (fitted exponent N^1.62 at 90%
    identity)
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:4eaecdebbf829cf78b83947d7925998d5b4cc90fd4ed52fba8aaad0acfd3eaa9
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# runtime-measurement-with-linux-time

## Summary

A benchmarking method for measuring wall-clock runtimes of sequence-clustering tools (Linclust, MMseqs2, CD-HIT, UCLUST) with the Linux 'time' command and fitting power laws T = a*N^b to quantify how runtime scales with input set size N.

## When to use

Use when comparing the computational scalability or speed of clustering/search tools on large sequence sets, or when reproducing reported runtime scaling exponents and speedup factors.

## When NOT to use

- When comparing clustering quality (e.g. functional consistency via GO annotations or Pfam) rather than runtime — quality metrics, not timing, are the relevant outputs.
- When the tools cannot be run on identical hardware with identical thread counts (e.g. CD-HIT -T 16 vs single-threaded baselines), since runtime ratios are then confounded.

## Inputs

- FASTA-formatted protein sequence sets of varying sizes (subsampled and full UniProt 2016_03, UniProt with reversed sequences)
- Tool executables with fixed parameters: MMseqs2 commit 5e21868 (mmseqs linclust / mmseqs cluster), CD-HIT 4.6, UCLUST 7.0.1090

## Outputs

- runtime_scaling.csv with fitted power-law exponent b (and prefactor a) per tool
- runtime_benchmark.tsv with wall-clock runtimes and speedup factors per tool at 50% and 90% identity
- log-log regression fit of runtime vs input set size N per tool

## How to apply

Prepare input sets spanning a range of sizes (e.g. 1/16, 1/8, 1/4, and 1/2 subsets of UniProt 2016_03 plus the full ~123-million-sequence UniProt+reversed set) so that runtimes can be fitted to a power law. Cluster each set with each tool using fixed, documented parameters — e.g. mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9, CD-HIT 4.6 with -T 16 -M 0 -n 5 -c 0.9, UCLUST with --id 0.9 — keeping hardware and thread counts constant. Measure each run's wall-clock runtime with the Linux time command rather than relying on tool-internal timers, so all tools are timed identically. Fit T = a*N^b per tool by linear regression in log-log space (numpy/scipy) to obtain the scaling exponent b, and compute speedup factors as runtime ratios between tools at matched identity thresholds (50% and 90%). Accept results when the fitted Linclust exponent is near 1 (linear, consistent with its O(mN) claim) and speedups reproduce the reported orders of magnitude.

## Related tools

- **Linclust** (Clustering tool under benchmark; expected to show linear N^~1 scaling via its O(mN) algorithm) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Suite hosting Linclust; run as mmseqs cluster / mmseqs linclust at fixed identity and coverage parameters) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool timed for comparison (fitted exponent N^2.75 at 90% identity))
- **UCLUST** (Baseline clustering tool timed for comparison (fitted exponent N^1.62 at 90% identity))

## Examples

```
time mmseqs linclust uniprot.db clusters.tmp --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- runtime_scaling.csv exists with one fitted exponent per tool, and the Linclust exponent is near 1 (linear scaling, consistent with the O(mN) claim).
- Fitted exponents for competing tools are substantially larger, approaching quadratic behavior (UCLUST ~N^1.62, CD-HIT ~N^2.75 at 90% identity).
- Computed speedup factors reproduce the reported orders of magnitude: ~2300x vs UCLUST and ~4600x vs CD-HIT at 50% identity; ~100x and ~62x at 90% identity.
- All tools are timed with the same method (Linux time) on the same sequence subsets at matched identity thresholds (0.5 and 0.9) with fixed parameters.
- Log-log regression residuals are small and the power law fits the measured (N, T) points across all subset sizes.

## Limitations

- Runtime measurements are hardware- and configuration-dependent (e.g. -T 16 -M 0 for CD-HIT); speedup factors are only comparable on identical setups.
- Fitted exponents are empirical over the sampled size range and 'very roughly quadratic'; they do not prove asymptotic complexity.
- Linclust's linear-time guarantee depends on the k-mer prefilter being approximate; exact-distance verification in canopy-style alternatives remains quadratic O(N^2/F).

## Evidence

- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] In contrast, Linclust's linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure.: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [results] Fitted power-law exponents at 90% identity show UCLUST (N^1.62) and CD-HIT (N^2.75) scale very roughly quadratically: [evidence span withheld — anchor status: not_found]
- [methods] The source code and binaries for Linclust can be download at https://github.com/soedinglab/mmseqs2.: "The source code and binaries for Linclust can be download at https://github.com/soedinglab/mmseqs2."
