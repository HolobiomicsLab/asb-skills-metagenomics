---
name: goodness-of-fit-statistics-computation
description: Use when benchmark timing data must be characterized by a scaling law
  — e.g. verifying claimed algorithmic complexity (O(n^2) vs O(n)) from measured runtimes
  of CATHAI graph generation.
when_to_use_negative:
- When the analysis goal is CATHAI installation, deployment, or data processing (docker
  process/run) rather than characterizing computational complexity from timing data
- When input data are not paired runtime-vs-parameter measurements (e.g. only a single
  timing point) — a power-law exponent cannot be fit
- When the scaling parameter is not runtime or sample/iteration count (e.g. accuracy
  metrics), since the log-log OLS validation targets complexity exponents specifically
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python 3.9
  role: Environment for log-transformation and ordinary least-squares power-law fitting
    of the benchmark runtimes
- name: R
  role: Ecosystem in which CATHAI's analysis/visualization packages (e.g. ggplot2)
    run, useful for plotting the fits
- name: CATHAI
  role: Subject application whose precomputed graph-generation times (sample count
    and Fruchterman–Reingold iteration threshold) are benchmarked and fit
  repo: https://github.com/FordeGenomics/cathai
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac040
    title: 'CATHAI: cluster analysis tool for healthcare-associated infections'
  source_content_hash: sha256:4e1834d40446327e66b10cfb46891b8d41c4a8979da1043abe552228112ce7de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FordeGenomics/cathai
  tools:
  - CATHAI
  - Python 3.9
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# goodness-of-fit-statistics-computation

## Summary

Fits power-law models runtime = a * n^b to CATHAI precomputed graph-generation benchmark data via ordinary least-squares regression on log-log axes, recovering scaling exponents and R-squared goodness-of-fit for both the sample-count and Fruchterman–Reingold iteration-threshold series.

## When to use

Use when benchmark timing data must be characterized by a scaling law — e.g. verifying claimed algorithmic complexity (O(n^2) vs O(n)) from measured runtimes of CATHAI graph generation.

## When NOT to use

- When the analysis goal is CATHAI installation, deployment, or data processing (docker process/run) rather than characterizing computational complexity from timing data
- When input data are not paired runtime-vs-parameter measurements (e.g. only a single timing point) — a power-law exponent cannot be fit
- When the scaling parameter is not runtime or sample/iteration count (e.g. accuracy metrics), since the log-log OLS validation targets complexity exponents specifically

## Inputs

- Benchmark computation-time table (chained input 'benchmark_table') of CATHAI graph-generation runtimes across varying sample counts
- Benchmark runtimes across varying Fruchterman–Reingold iteration-threshold values
- Python 3.9 environment (e.g. conda env 'cathai' with python=3.9)

## Outputs

- complexity_fit_results.csv containing fitted exponents, coefficients, and R-squared goodness-of-fit statistics for both the sample-count and iteration-threshold power-law fits

## How to apply

Load the benchmark computation-time table (chained input 'benchmark_table') produced by the CATHAI benchmarking sub-task, which times precomputed graph generation while varying the number of samples and the iteration threshold passed to the Fruchterman–Reingold layout (default app args are '1000 1000'). In Python 3.9, log-transform runtimes and sample counts and fit runtime = a * n^b by OLS on log-log axes to recover exponent b; repeat the same log-log regression for the iteration-threshold series. Compute R-squared goodness-of-fit for both regressions and record fitted exponents, coefficients, and fit statistics to complexity_fit_results.csv. Accept the paper's complexity claims when the sample-count exponent is close to 2 (O(n^2)) and the iteration-threshold exponent is close to 1 (O(n)) with high regression goodness-of-fit; the paper found sample size has a much greater impact than iterations, so expect the quadratic term to dominate.

## Related tools

- **Python 3.9** (Environment for log-transformation and ordinary least-squares power-law fitting of the benchmark runtimes)
- **R** (Ecosystem in which CATHAI's analysis/visualization packages (e.g. ggplot2) run, useful for plotting the fits)
- **CATHAI** (Subject application whose precomputed graph-generation times (sample count and Fruchterman–Reingold iteration threshold) are benchmarked and fit) — https://github.com/FordeGenomics/cathai

## Examples

```
import numpy as np; b, log_a = np.polyfit(np.log(samples), np.log(runtimes), 1); r2 = np.corrcoef(np.log(samples), np.log(runtimes))[0,1]**2
```

## Evaluation signals

- Fitted exponent for the sample-count series is close to 2, consistent with the reported O(n^2) relationship
- Fitted exponent for the iteration-threshold series is close to 1, consistent with the reported O(n) relationship
- R-squared goodness-of-fit values are high for both log-log regressions
- Fitted coefficients, exponents, and fit statistics are present in complexity_fit_results.csv without missing values
- Quadratic-in-sample-count term accounts for a much larger share of runtime variation than the linear iteration term

## Limitations

- Power-law exponents are only valid over the sample-count and iteration-threshold ranges actually benchmarked (the app's default processing args are '1000 1000'); extrapolation beyond the tested range is unsupported
- The fit validates the paper's complexity claims only for CATHAI's precomputed graph-generation step, not other pipeline stages
- Runtime measurements are machine-dependent, so fitted coefficient a (not exponent b) is hardware-specific

## Evidence

- [methods] docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process: "docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process"
- [readme] conda env definition: "conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0"
- [readme] R package install line: [evidence span withheld — anchor status: not_found]
