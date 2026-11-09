---
name: sequence-identity-threshold-configuration
description: Use when clustering protein or nucleotide sequence sets at a chosen sequence
  identity level (e.g. 90%, 70%, or 50%) and the tool-specific parameters — --min-seq-id,
  -c/-n for CD-HIT, --id for UCLUST — must be matched to that threshold.
when_to_use_negative:
- The task is pairwise sequence search or alignment scoring rather than redundancy
  clustering — identity thresholds there (e.g. BLAST-style criteria) are configured
  differently
- The identity cutoff falls between the validated word-size regimes (e.g. needing
  a cutoff between 40% and 50% where CD-HIT -n 3 is no longer matched)
- Exact similarity verification of all candidate pairs is required, since k-mer-prefiltered
  clustering at low identity is only approximate (Linclust's prefilter is approximate)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Sets the identity threshold directly via --min-seq-id together with --cov-mode
    1 -c 0.9; integrated into MMseqs2
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool whose identity threshold -c must be paired with the
    matching word size -n (5 for 90%, 4 for 70%, 3 for 50%)
- name: UCLUST
  role: Baseline clustering tool configured at 90% and 50% levels via --id 0.9 and
    --id 0.5
- name: MMseqs2
  role: Software suite hosting the Linclust command used to apply the configured thresholds
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:226cb790a346687ad680078dd8e5040aa890c0f48387680354a9ab834b76e35d
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# sequence-identity-threshold-configuration

## Summary

Configures and validates sequence identity thresholds (e.g. 90%, 70%, 50%) when running clustering tools like Linclust/MMseqs2, CD-HIT, and UCLUST, mapping each threshold to the correct word-size/-n and identity-parameter combinations. Correct threshold-to-parameter pairing is essential because runtime and clustering quality diverge sharply between tools as the identity level drops.

## When to use

Use when clustering protein or nucleotide sequence sets at a chosen sequence identity level (e.g. 90%, 70%, or 50%) and the tool-specific parameters — --min-seq-id, -c/-n for CD-HIT, --id for UCLUST — must be matched to that threshold.

## When NOT to use

- The task is pairwise sequence search or alignment scoring rather than redundancy clustering — identity thresholds there (e.g. BLAST-style criteria) are configured differently
- The identity cutoff falls between the validated word-size regimes (e.g. needing a cutoff between 40% and 50% where CD-HIT -n 3 is no longer matched)
- Exact similarity verification of all candidate pairs is required, since k-mer-prefiltered clustering at low identity is only approximate (Linclust's prefilter is approximate)

## Inputs

- Protein sequence set in FASTA format (e.g. UniProt 2016_03 release, or the ~123-million-sequence UniProt+reversed benchmark set)
- Target sequence identity threshold (90%, 70%, or 50%) and desired coverage mode

## Outputs

- Cluster assignment of input sequences at the configured identity threshold
- Runtime measurements (per tool, per threshold) suitable for a runtime_benchmark.tsv comparison
- Speedup factors of Linclust vs. baseline tools at each identity level

## How to apply

First fix the target identity level (the paper evaluates 90%, 70% and 50% sequence identity) and the coverage mode: Linclust was run with --cov-mode 1 -c 0.9 to enforce coverage. For Linclust via MMseqs2, set the identity directly with --min-seq-id (0.9 for the 90% level, 0.5 for the 50% level). For CD-HIT 4.6, pair the identity cutoff -c with the matching word size: -n 5 for 90%, -n 4 for 70%, -n 3 for 50%, while fixing resources with -T 16 -M 0. For UCLUST, set the threshold with --id 0.9 or --id 0.5. The decision point is the word-size-to-identity mapping: using an incorrect -n for a given -c in CD-HIT, or omitting the coverage mode in Linclust, changes which pairs can be clustered and invalidates cross-tool runtime comparisons.

## Related tools

- **Linclust** (Sets the identity threshold directly via --min-seq-id together with --cov-mode 1 -c 0.9; integrated into MMseqs2) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool whose identity threshold -c must be paired with the matching word size -n (5 for 90%, 4 for 70%, 3 for 50%))
- **UCLUST** (Baseline clustering tool configured at 90% and 50% levels via --id 0.9 and --id 0.5)
- **MMseqs2** (Software suite hosting the Linclust command used to apply the configured thresholds) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust input.fasta resultCluster tmp --cov-mode 1 -c 0.9 --min-seq-id 0.5
```

## Evaluation signals

- CD-HIT word size matches the identity cutoff: -n 5 ↔ c=0.9, -n 4 ↔ c=0.7, -n 3 ↔ c=0.5
- Linclust runs include --cov-mode 1 -c 0.9 alongside the intended --min-seq-id value
- Reproduced speedups fall in the expected orders of magnitude: ~2300x vs UCLUST and ~4600x vs CD-HIT at 50% identity; ~100x and ~62x at 90%
- Runtimes at each threshold are recorded (e.g. via the Linux 'time' command) and written to a consistent output such as runtime_benchmark.tsv
- Cluster count and cluster membership change sensibly as the threshold is lowered from 90% to 50% (lower identity ⇒ larger clusters)

## Limitations

- K-mer prefilter-based clustering (as in CD-HIT, UCLUST, kclust, MMseqs) is only approximate; if falsely clustered pairs are costly, exact-similarity verification of pairwise links is needed and is quadratic in N
- CD-HIT 4.6 with -M 0 was memory-unbounded in the benchmark; other resource settings change measured runtimes
- At 50% identity, exhaustive baselines like UCLUST and CD-HIT become impracticably slow on very large sets (tens of millions of sequences), making full runtime comparison infeasible

## Evidence

- [methods] Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [methods] The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated.: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [readme] MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets.: "MMseqs2 (Many-against-Many sequence searching) is a software suite to search and cluster huge protein and nucleotide sequence sets."
- [full_text] At 50% identity, Linclust clusters 123 million sequences 2300 times faster than UCLUST, 720 times faster than MMseqs, and 4600 times faster than CD-HIT: [evidence span withheld — anchor status: not_found]
