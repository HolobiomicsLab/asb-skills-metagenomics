---
name: speedup-factor-calculation
description: Use when benchmarking clustering software runtime on large protein sequence
  sets and you need to express performance as a multiplicative speedup over baseline
  tools at fixed identity thresholds (e.g., 50% and 90%).
when_to_use_negative:
- The goal is to compare clustering quality (e.g., GO/Pfam cluster consistency) rather
  than runtime; speedup factors say nothing about clustering accuracy
- The dataset is small enough that all tools finish in comparable time, making multiplicative
  speedup uninformative
- Sequence identity thresholds or coverage settings differ between the new tool and
  the baselines, so runtimes are not comparable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Test tool whose runtime is measured; invoked via 'mmseqs linclust' with --cov-mode
    1 -c 0.9 and --min-seq-id 0.9 or 0.5
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool run with -T 16 -M 0 and -n 5 -c 0.9 / -n 3 -c 0.5
    to obtain comparison runtimes
- name: UCLUST
  role: Baseline clustering tool run with --id 0.9 and --id 0.5 to obtain comparison
    runtimes
- name: MMseqs2
  role: Software suite embedding Linclust, also serving as a baseline ('Linclust/MMseqs2'
    timings)
  repo: https://github.com/soedinglab/mmseqs2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:45a2fffaff4e09c4b767f05f625bdaa26ed718d709b26dfc255fb30da95d2b27
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# speedup-factor-calculation

## Summary

Compute runtime speedup factors of a linear-time clustering algorithm (Linclust) against baseline tools (UCLUST, CD-HIT, MMseqs) at fixed sequence identity thresholds, quantifying how many times faster the new method clusters huge protein sequence sets such as the 123-million-sequence UniProt+reversed benchmark.

## When to use

Use when benchmarking clustering software runtime on large protein sequence sets and you need to express performance as a multiplicative speedup over baseline tools at fixed identity thresholds (e.g., 50% and 90%).

## When NOT to use

- The goal is to compare clustering quality (e.g., GO/Pfam cluster consistency) rather than runtime; speedup factors say nothing about clustering accuracy
- The dataset is small enough that all tools finish in comparable time, making multiplicative speedup uninformative
- Sequence identity thresholds or coverage settings differ between the new tool and the baselines, so runtimes are not comparable

## Inputs

- UniProt 2016_03 release protein sequence set / UniProt+reversed benchmark set (~123 million sequences)
- Installed clustering tools: Linclust/MMseqs2 (commit 5e21868), CD-HIT 4.6, UCLUST
- Identity thresholds (50%, 90%) and coverage parameter (-c 0.9, --cov-mode 1)

## Outputs

- Per-tool wall-clock runtimes recorded with the Linux 'time' command
- Speedup factors of Linclust over each baseline at each identity threshold
- runtime_benchmark.tsv containing runtimes and speedups

## How to apply

Obtain the UniProt 2016_03 release and prepare the ~123-million-sequence UniProt+reversed test set. Run Linclust (inside MMseqs2, commit 5e21868) with 'mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9' for the 90% level and '--min-seq-id 0.5' for the 50% level, timing each run with the Linux 'time' command. Run baselines with matched thresholds: CD-HIT 4.6 with -T 16 -M 0 and word-size/identity pairs -n 5 -c 0.9 (90%) and -n 3 -c 0.5 (50%); UCLUST with --id 0.9 and --id 0.5. Divide each baseline runtime by the Linclust runtime to obtain per-tool, per-threshold speedup factors, and write runtimes and speedups to runtime_benchmark.tsv.

## Related tools

- **Linclust** (Test tool whose runtime is measured; invoked via 'mmseqs linclust' with --cov-mode 1 -c 0.9 and --min-seq-id 0.9 or 0.5) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool run with -T 16 -M 0 and -n 5 -c 0.9 / -n 3 -c 0.5 to obtain comparison runtimes)
- **UCLUST** (Baseline clustering tool run with --id 0.9 and --id 0.5 to obtain comparison runtimes)
- **MMseqs2** (Software suite embedding Linclust, also serving as a baseline ('Linclust/MMseqs2' timings)) — https://github.com/soedinglab/mmseqs2

## Examples

```
mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- At 50% identity, Linclust should be ~2300x faster than UCLUST, ~720x faster than MMseqs, and ~4600x faster than CD-HIT on the 123M-sequence set
- At 90% identity, Linclust should remain ~100x faster than UCLUST, ~62x faster than CD-HIT, and ~570x faster than MMseqs
- All tools must be run with matched identity thresholds and coverage settings (e.g., -c 0.9 with --cov-mode 1) before dividing runtimes
- runtime_benchmark.tsv must contain per-tool runtimes and the computed speedup ratios at both 50% and 90% identity
- Reproduced speedups should match the paper's orders of magnitude (thousands-fold at 50%, tens- to hundreds-fold at 90%)

## Limitations

- Baseline tools like CD-HIT and UCLUST scale nearly quadratically (N x K) on huge sets, so some runs are impractically slow or infeasible when both N and K are in the tens of millions
- Speedup factors are hardware- and commit-dependent: Linclust timings were obtained with MMseqs2 commit 5e21868 on a specific server configuration
- Runtime speedup does not guarantee cluster-quality equivalence; quality must be assessed separately (e.g., via GO and Pfam annotation consistency)

## Evidence

- [methods] Linclust/MMseqs2 command: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [methods] Benchmark reproducibility: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
- [methods] UniProt dataset: "The UniProt 2016_03 release was clustered by each tool at 90%, 70% and 50% sequence identity level and then evaluated."
- [abstract] Order-of-magnitude speedup: "We cluster 1.6 billion metagenomic sequence fragments in 10 h on a single server to 50% sequence identity, >1000 times faster than has been possible before."
