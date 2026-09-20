---
name: wilcoxon-signed-rank-testing
description: Use when comparing two methods' error metrics (e.g., MSE of estimated
  source mixing proportions) measured on the same paired benchmark instances — such
  as in vitro gut-mixture sinks with known ground-truth proportions — and a nonparametric
  paired significance test is needed.
when_to_use_negative:
- The comparison samples are independent/unpaired (e.g., different sinks per method)
  rather than the same sinks tested by all methods — a rank-sum/unpaired formulation
  would then be the appropriate framing
- The metric distribution is known to be normal and sample sizes are large enough
  that a paired t-test is preferred, or the data are ordinal with many ties that violate
  signed-rank assumptions
- There is no ground truth (true mixing proportions) available to compute per-sink
  error, so no paired error metric exists to test
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Implementation environment in which the FEAST/STENSL codebase runs and where
    per-sink MSEs are computed and Wilcoxon signed-rank tests are applied
  repo: https://github.com/cozygene/FEAST
- name: STENSL
  role: Method under test whose MSE advantage over baselines is assessed by Wilcoxon
    signed-rank tests on the in vitro benchmark
  repo: https://github.com/cozygene/FEAST/tree/STENSL
- name: FEAST
  role: Baseline source tracking method; host codebase for STENSL, installed via devtools
    from GitHub and run with EM_iterations = 1000 to generate paired MSE values
  repo: https://github.com/cozygene/FEAST
- name: SourceTracker2
  role: Baseline source tracking method whose MSE is compared to STENSL's via Wilcoxon
    signed-rank test (P < 2.2e-16)
- name: RAD
  role: Baseline source tracking method whose MSE is compared to STENSL's via Wilcoxon
    signed-rank test (P < 2.8e-16)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00995-21
    title: 'STENSL: Microbial Source Tracking with ENvironment SeLection'
  source_content_hash: sha256:9c4544decf7044dd71678f84e9bc0c3ed499803c8a8ba1d131a88e5ad5716238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cozygene/FEAST
  tools:
  - FEAST
  - R
  - RAD
  - Rcpp
  - RcppArmadillo
  - STENSL
  - SourceTracker2
  - devtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# wilcoxon-signed-rank-testing

## Summary

Apply Wilcoxon signed-rank tests to nonparametrically compare mixing-proportion estimation errors (MSE) of microbial source tracking methods — STENSL vs FEAST, SourceTracker2, and RAD — across the same set of in vitro gut-mixture sinks, yielding the significance thresholds reported in the STENSL paper (Fig 3).

## When to use

Use when comparing two methods' error metrics (e.g., MSE of estimated source mixing proportions) measured on the same paired benchmark instances — such as in vitro gut-mixture sinks with known ground-truth proportions — and a nonparametric paired significance test is needed.

## When NOT to use

- The comparison samples are independent/unpaired (e.g., different sinks per method) rather than the same sinks tested by all methods — a rank-sum/unpaired formulation would then be the appropriate framing
- The metric distribution is known to be normal and sample sizes are large enough that a paired t-test is preferred, or the data are ordinal with many ties that violate signed-rank assumptions
- There is no ground truth (true mixing proportions) available to compute per-sink error, so no paired error metric exists to test

## Inputs

- Per-sink MSE values for each source tracking method (STENSL, FEAST, SourceTracker2, RAD) computed against known true mixing proportions of in vitro mixtures
- Paired per-sink estimated mixing-proportion matrices (e.g., FEAST's S1 x S2 matrix P including an unknown source column) from each method on identical sink-source configurations
- Ground-truth mixing proportions of the lab-generated in vitro human/mouse gut mixtures

## Outputs

- Pairwise Wilcoxon signed-rank p-values comparing STENSL's MSE to each baseline (expected: SourceTracker2 P < 2.2e-16, FEAST P < 2.1e-16, RAD P < 2.8e-16)
- Ranking of the four methods by MSE against ground truth
- Statistical validation verdict that the reproduced ordering and p-values match Fig 3

## How to apply

First run each source tracking method (STENSL, FEAST, SourceTracker2, RAD) on identical sink-source configurations — e.g., the in vitro dataset where 50 noncontributing sources are mixed with human/mouse gut sources at unknown proportions of 0%–~80% — then compute each method's mean squared error (MSE) of estimated proportions against the lab-generated ground truth per sink. Because the same sinks are evaluated under all methods, the per-sink MSEs are paired samples, so rank the absolute paired differences and apply the Wilcoxon signed-rank test to obtain p-values for each baseline comparison. In the STENSL paper this yielded STENSL vs SourceTracker2 P < 2.2e-16, STENSL vs FEAST P < 2.1e-16, and STENSL vs RAD P < 2.8e-16. Validation requires that the reproduced MSE ordering matches Fig 3 (STENSL most accurate, with existing methods degrading as 50+ candidate sources are considered) and that pairwise p-values fall below the reported significance thresholds.

## Related tools

- **R** (Implementation environment in which the FEAST/STENSL codebase runs and where per-sink MSEs are computed and Wilcoxon signed-rank tests are applied) — https://github.com/cozygene/FEAST
- **STENSL** (Method under test whose MSE advantage over baselines is assessed by Wilcoxon signed-rank tests on the in vitro benchmark) — https://github.com/cozygene/FEAST/tree/STENSL
- **FEAST** (Baseline source tracking method; host codebase for STENSL, installed via devtools from GitHub and run with EM_iterations = 1000 to generate paired MSE values) — https://github.com/cozygene/FEAST
- **SourceTracker2** (Baseline source tracking method whose MSE is compared to STENSL's via Wilcoxon signed-rank test (P < 2.2e-16))
- **RAD** (Baseline source tracking method whose MSE is compared to STENSL's via Wilcoxon signed-rank test (P < 2.8e-16))

## Examples

```
wilcox.test(mse_stensl, mse_sourcetracker2, paired = TRUE, alternative = "less")
```

## Evaluation signals

- Reproduced MSE ordering must match Fig 3: STENSL most accurate, with FEAST/SourceTracker2/RAD degrading as 50+ candidate sources are considered
- Pairwise Wilcoxon p-values must fall below the reported thresholds: SourceTracker2 P < 2.2e-16, FEAST P < 2.1e-16, RAD P < 2.8e-16
- Each method's per-sink mixing-proportion rows must sum to 1 (FEAST's matrix P rows, including the unknown source, sum to 1 by construction)
- Paired structure must be verified: each Wilcoxon test compares MSEs on the identical set of sinks withheld to create unknown proportions of 0%–~80%
- The number of paired observations must equal the number of in vitro test sinks, with no NA entries (NA in P indicates a source was excluded from that sink's analysis)

## Limitations

- The in vitro benchmark data lacks a public accession/deposit — it is only described in supplementary Text S2 — so exact reproduction requires regenerating or obtaining the mixtures
- Detailed simulation and in vitro assembly protocols are delegated to supplementary Texts S1 and S2, not the main text
- Nonparametric tests report significance of the error difference but not effect size; the paper reports only p-value thresholds alongside Fig 3

## Evidence

- [results] On the in vitro data, STENSL was significantly more accurate than the other methods by MSE against lab-generated ground truth: SourceTracker2 P < 2.2e-16, FEAST P < 2.1e-16, RAD P < 2.8e-16 (Wilcoxon ranked-sum test).: [evidence span withheld — anchor status: not_found]
- [abstract] STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments: "STENSL is significantly more accurate in identifying the contributing sources as well as the unknown source, even when considering hundreds of potential source environments"
- [readme] FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1.: "FEAST returns an S1 by S2 matrix P, where S1 is the number sinks and S2 is the number of sources (including an unknown source). Each row in matrix P sums to 1."
- [methods] Install the FEAST codebase (which hosts the STENSL branch) from GitHub with devtools.: "devtools::install_github("cozygene/FEAST", ref = "FEAST")"
