---
name: sequence-database-subset-sampling
description: Use when you need to characterize or validate the asymptotic runtime
  scaling of a clustering or search tool by measuring wall-clock times on nested random
  subsets of a sequence database.
when_to_use_negative:
- You want clustering quality or sensitivity benchmarks rather than runtime scaling
  — this skill only measures wall-clock time versus input size
- The tool under test cannot complete on the smallest subset within your compute budget,
  since nested-subset fitting requires runtimes at every scale point
- You need exact worst-case complexity analysis; fitted exponents are empirical approximations
  ('very roughly quadratic') valid only near the tested identity threshold and hardware
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0623
tools:
- name: Linclust
  role: Test tool whose linear O(mN) runtime scaling is being verified; run as mmseqs
    linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9 on each subset
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Suite hosting Linclust; the full cluster workflow (mmseqs cluster) is also
    benchmarked at commit 5e21868
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Competitor baseline; run with -T 16 -M 0 -n 5 -c 0.9 on the same subsets to
    expose near-quadratic scaling (fitted N^2.75)
- name: UCLUST
  role: Competitor baseline; run with --id 0.9 and 90% coverage of the shorter sequence,
    fitted exponent N^1.62
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:ae596317de834305a595b4266ba0e58d59bef266b320fc3db4cf176fab06155b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0623
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sequence-database-subset-sampling

## Summary

Empirically measure how sequence-clustering runtime scales with input set size N by clustering nested random subsets of a database (e.g., UniProt 2016_03) and fitting a power law T = a*N^b in log-log space, as done in the Linclust paper to demonstrate linear O(mN) scaling.

## When to use

Use when you need to characterize or validate the asymptotic runtime scaling of a clustering or search tool by measuring wall-clock times on nested random subsets of a sequence database.

## When NOT to use

- You want clustering quality or sensitivity benchmarks rather than runtime scaling — this skill only measures wall-clock time versus input size
- The tool under test cannot complete on the smallest subset within your compute budget, since nested-subset fitting requires runtimes at every scale point
- You need exact worst-case complexity analysis; fitted exponents are empirical approximations ('very roughly quadratic') valid only near the tested identity threshold and hardware

## Inputs

- Full FASTA sequence database (e.g., UniProt 2016_03 release)
- Randomly sampled nested subsets at 1/16, 1/8, 1/4, 1/2 of the full set, plus a reversed-sequence version
- Installed clustering tools (Linclust/MMseqs2 commit 5e21868, CD-HIT 4.6, UCLUST 7.0.1090)

## Outputs

- Wall-clock runtime measurements per tool per subset size (from Linux time)
- runtime_scaling.csv with fitted power-law exponent b and prefactor a per tool
- Scaling comparison: Linclust N^0.94, Linclust/MMseqs2 N^1.01, UCLUST N^1.62, CD-HIT N^2.75 at 90% identity

## How to apply

Randomly sample nested subsets of the target database (1/16, 1/8, 1/4, and 1/2 of the UniProt 2016_03 release, plus a version with reversed sequences) so runtimes can be fit against increasing N. Cluster every subset and the full set with each tool under comparison using identical thresholds — for Linclust/MMseqs2 (commit 5e21868): mmseqs linclust and mmseqs cluster with --cov-mode 1 -c 0.9 --min-seq-id 0.9; for CD-HIT 4.6: -T 16 -M 0 -n 5 -c 0.9; for UCLUST 7.0.1090: --id 0.9 with 90% minimum coverage of the shorter sequence to enforce global similarity. Measure each run's wall-clock time with the Linux time command. Fit T = a*N^b per tool by linear regression on the log-transformed runtimes (numpy/scipy), writing the fitted exponent b and prefactor a to a results table such as runtime_scaling.csv. Accept the result when the Linclust exponent is near 1 (linear, consistent with the O(mN) claim) while competitors approach quadratic scaling.

## Related tools

- **Linclust** (Test tool whose linear O(mN) runtime scaling is being verified; run as mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9 on each subset) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Suite hosting Linclust; the full cluster workflow (mmseqs cluster) is also benchmarked at commit 5e21868) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Competitor baseline; run with -T 16 -M 0 -n 5 -c 0.9 on the same subsets to expose near-quadratic scaling (fitted N^2.75))
- **UCLUST** (Competitor baseline; run with --id 0.9 and 90% coverage of the shorter sequence, fitted exponent N^1.62)

## Examples

```
mmseqs linclust subset.fasta subsetClu --cov-mode 1 -c 0.9 --min-seq-id 0.9 && /usr/bin/time -v cd-hit -i subset.fasta -o out -c 0.9 -n 5 -T 16 -M 0
```

## Evaluation signals

- runtime_scaling.csv exists and contains exactly one fitted exponent per tool
- Fitted exponent for Linclust/MMseqs2 is near 1 (paper values: N^0.94 and N^1.01), confirming linear scaling consistent with the O(mN) claim
- CD-HIT and UCLUST exponents are substantially larger (paper: N^2.75 and N^1.62), approaching quadratic behavior
- Every tool was run with matched thresholds (90% identity, 90% coverage of shorter sequence) on identical nested subsets, so runtimes are comparable
- Log-log regression residuals are small and the fit is monotonic in N across the 1/16-to-full-set range

## Limitations

- Fitted exponents are empirical and hardware/parameter dependent ('scale very roughly quadratically' at 90% identity only); they do not prove asymptotic complexity in general
- Exact-distance verification in approximate prefiltering schemes still requires quadratic time O(N2/F) if exact pairwise links are needed
- Traditional approaches become impracticably slow when both N and K are in the tens of millions, capping the largest measurable subset for slow tools

## Evidence

- [results] Fitted power-law exponents at 90% identity show UCLUST (N^1.62) and CD-HIT (N^2.75) scale very roughly quadratically, whereas Linclust/MMseqs2 (N^0.94) and Linclust (N^1.01) scale linearly: [evidence span withheld — anchor status: not_found]
- [methods] Linclust commands: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [discussion] Linear complexity claim: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
- [methods] Benchmark availability: "All scripts and benchmark data including command-line parameters necessary to reproduce the benchmark and analysis results presented here are available at"
