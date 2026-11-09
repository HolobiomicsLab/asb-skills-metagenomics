---
name: power-law-regression-fitting
description: 'Use when comparing algorithm scalability: runtime measurements exist
  for multiple input sizes N (e.g., UniProt subsets) and you need a fitted scaling
  exponent to distinguish linear (b≈1) from quadratic (b≈2) behavior.'
when_to_use_negative:
- You need exact clustering quality metrics (e.g., GO-annotation cluster consistency)
  rather than runtime-vs-size scaling behavior
- The dataset has no natural size axis — a single fixed-size run cannot support a
  power-law fit
- Comparisons across tools would use different coverage or identity thresholds, making
  the fitted exponents incomparable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Linclust
  role: Clustering tool whose linear runtime scaling is the expected fitted result
    (b≈1) at 90% identity with --cov-mode 1 -c 0.9 --min-seq-id 0.9
  repo: https://github.com/soedinglab/mmseqs2
- name: MMseqs2
  role: Software suite providing Linclust; also run as 'Linclust/MMseqs2' (mmseqs
    cluster) to obtain a second scaling reference point
  repo: https://github.com/soedinglab/mmseqs2
- name: CD-HIT 4.6
  role: Baseline clustering tool run with -T 16 -M 0 -n 5 -c 0.9 whose fitted exponent
    (~N^2.75) demonstrates near-quadratic scaling
- name: UCLUST
  role: Baseline clustering tool run with --id 0.9 whose fitted exponent (~N^1.62)
    demonstrates rough quadratic scaling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-04964-5
    title: Clustering huge protein sequence sets in linear time
  source_content_hash: sha256:96f7c434f57debc4b0cae17978e44bd2706d66d4ec8dbbe1d608e90cfb8420d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/mmseqs2
  tools:
  - CD-HIT 4.6
  - Linclust
  - MMseqs2
  - UCLUST
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# power-law-regression-fitting

## Summary

Fits a power law T = a*N^b to wall-clock runtimes measured over exponentially spaced input subset sizes (e.g., 1/16 to full UniProt 2016_03), estimating the scaling exponent b per tool to test claims like Linclust's linear O(mN) runtime versus near-quadratic scaling of UCLUST and CD-HIT.

## When to use

Use when comparing algorithm scalability: runtime measurements exist for multiple input sizes N (e.g., UniProt subsets) and you need a fitted scaling exponent to distinguish linear (b≈1) from quadratic (b≈2) behavior.

## When NOT to use

- You need exact clustering quality metrics (e.g., GO-annotation cluster consistency) rather than runtime-vs-size scaling behavior
- The dataset has no natural size axis — a single fixed-size run cannot support a power-law fit
- Comparisons across tools would use different coverage or identity thresholds, making the fitted exponents incomparable

## Inputs

- FASTA-formatted protein sequence sets at multiple sizes (e.g., 1/16, 1/8, 1/4, 1/2 fractions and full UniProt 2016_03 release)
- Clustering commands/parameters per tool (Linclust, Linclust/MMseqs2, CD-HIT 4.6, UCLUST 7.0.1090)
- Wall-clock runtimes per run measured with the Linux time command

## Outputs

- runtime_scaling.csv with one fitted power-law exponent b (and prefactor a) per tool (e.g., N^0.94 Linclust/MMseqs2, N^1.01 Linclust, N^1.62 UCLUST, N^2.75 CD-HIT at 90% identity)

## How to apply

Prepare input sets by randomly sampling 1/16, 1/8, 1/4, and 1/2 fractions of the dataset plus the full set (e.g., the UniProt 2016_03 release, optionally with reversed sequences). Run the tool under evaluation on every subset — e.g., mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9 (commit 5e21868) for Linclust, CD-HIT 4.6 with -T 16 -M 0 -n 5 -c 0.9, or UCLUST 7.0.1090 with --id 0.9 — enforcing the same 90% minimum coverage of the shorter sequence across all tools. Measure each run's wall-clock time with the Linux time command. Log-transform runtimes and N, fit T = a*N^b by linear regression in log-log space using numpy/scipy, and record the fitted exponent b and prefactor a per tool (e.g., runtime_scaling.csv). Accept the result when Linclust's exponent is near 1 (linear, consistent with its O(mN) claim) while competitors show substantially larger exponents approaching quadratic behavior.

## Related tools

- **Linclust** (Clustering tool whose linear runtime scaling is the expected fitted result (b≈1) at 90% identity with --cov-mode 1 -c 0.9 --min-seq-id 0.9) — https://github.com/soedinglab/mmseqs2
- **MMseqs2** (Software suite providing Linclust; also run as 'Linclust/MMseqs2' (mmseqs cluster) to obtain a second scaling reference point) — https://github.com/soedinglab/mmseqs2
- **CD-HIT 4.6** (Baseline clustering tool run with -T 16 -M 0 -n 5 -c 0.9 whose fitted exponent (~N^2.75) demonstrates near-quadratic scaling)
- **UCLUST** (Baseline clustering tool run with --id 0.9 whose fitted exponent (~N^1.62) demonstrates rough quadratic scaling)

## Examples

```
mmseqs linclust input.fasta result_tmp result --cov-mode 1 -c 0.9 --min-seq-id 0.9
```

## Evaluation signals

- runtime_scaling.csv exists and contains one exponent per evaluated tool
- Fitted exponent for Linclust is near 1 (linear scaling, consistent with the paper's O(mN) claim)
- Exponents for UCLUST and CD-HIT are substantially larger, approaching quadratic behavior (paper: ~1.62 and ~2.75 at 90% identity)
- All tools were run with identical similarity criteria (90% sequence identity and 90% minimum coverage of the shorter sequence) so the fits are comparable
- Log-log regression residuals show the power-law model T = a*N^b adequately describes runtimes across the 1/16-to-full subset range

## Limitations

- Fitted exponents describe scaling only within the tested range of N and identity threshold (here 90%); exponents can differ at 70% or 50% identity
- Linclust-like prefiltering is approximate: if false clusters are costly, exact similarity verification is still required, pushing complexity toward quadratic time
- Prefactors a are hardware- and thread-count-dependent (CD-HIT was run with -T 16 threads), so only relative exponents, not absolute times, are portable

## Evidence

- [methods] Linclust command parameters: "Linclust and Linclust/MMseqs2 (commit 5e21868) used the commands mmseqs linclust --cov-mode 1 -c 0.9 --min-seq-id 0.9"
- [methods] CD-HIT parameters: "CD-HIT 4.6 was run with the parameters -T 16 -M 0 and -n 5 -c 0.9, -n 4 -c 0.7, and -n 3 -c 0.5"
- [abstract] Linear scaling claim: "We developed Linclust, the first clustering algorithm whose runtime scales as N, independent of K."
- [discussion] Complexity of verification: "In contrast, Linclust’s linear time complexity of O(mN) includes verification of all edges between items using the exact distance measure."
