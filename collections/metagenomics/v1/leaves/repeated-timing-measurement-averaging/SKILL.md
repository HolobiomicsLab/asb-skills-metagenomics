---
name: repeated-timing-measurement-averaging
description: Use when quantifying the runtime scaling of CATHAI's graph-generation/plotting
  pipeline against sample count (n), e.g. to reproduce the timing-versus-sample-count
  benchmark table or to decide the maximum sample size for usable operation.
when_to_use_negative:
- The goal is user-facing clustering accuracy or visualization quality rather than
  runtime scaling of graph generation and plotting
- Input distances are not pairwise genomic-cluster distances loadable into CATHAI's
  shiny/data directory, so the Docker process step cannot be exercised
- Only a single timing run per configuration is feasible and the averaging protocol
  (10 repetitions) cannot be applied, making the benchmark table non-comparable
tools:
- name: CATHAI
  role: Application whose graph-generation and plotting performance is benchmarked
    via its Docker process entrypoint
  repo: https://github.com/FordeGenomics/cathai
- name: Docker
  role: Used to build the CATHAI image and run the data-processing/graph-generation
    step with bind-mounted data for each timing measurement
- name: Python 3.9
  role: Runtime environment (via conda env 'cathai') in which the processing and timing
    workload executes
- name: conda
  role: Creates the cathai environment (python=3.9, uwsgi=2.0, redis=5.0) supporting
    the benchmarked pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac040
    title: 'CATHAI: cluster analysis tool for healthcare-associated infections'
  source_content_hash: sha256:caf5f7f7c5e6edb1943ed0787bbe3407a2fa0d046678f399a9e57d751b7c5b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FordeGenomics/cathai
  tools:
  - CATHAI
  - Docker
  - Python 3.9
  - conda
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# repeated-timing-measurement-averaging

## Summary

Benchmark method for characterizing how CATHAI's precomputed network graph generation and local plotting scale with sample count, using 10 averaged wall-clock repetitions per sample count across iteration thresholds to establish the usable operating envelope.

## When to use

Use when quantifying the runtime scaling of CATHAI's graph-generation/plotting pipeline against sample count (n), e.g. to reproduce the timing-versus-sample-count benchmark table or to decide the maximum sample size for usable operation.

## When NOT to use

- The goal is user-facing clustering accuracy or visualization quality rather than runtime scaling of graph generation and plotting
- Input distances are not pairwise genomic-cluster distances loadable into CATHAI's shiny/data directory, so the Docker process step cannot be exercised
- Only a single timing run per configuration is feasible and the averaging protocol (10 repetitions) cannot be applied, making the benchmark table non-comparable

## Inputs

- Pairwise distance matrices for genomic clusters at increasing sample counts n (up to and beyond the 1000-sample point exercised by the tool's process entrypoint)
- Iteration threshold values (numeric arguments passed to the CATHAI process/run entrypoints, e.g. 1000 1000)

## Outputs

- benchmark_timing_table.csv — averaged (10 repetitions per count) wall-clock runtime, one row per sample count, across iteration thresholds
- Validated operating envelope: peak performance at n <= 500; sample counts above 1000 not recommended

## How to apply

Build the CATHAI Docker image with 'docker image build --file ./docker/Dockerfile --tag cathai ./docker'. Generate pairwise distance matrices at increasing sample counts n, then for each n mount the data directory into the container ('--mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data"') and invoke the process entrypoint (e.g. 'cathai 1000 1000 process') to trigger graph generation, recording wall-clock time. Repeat each timing measurement 10 times per sample count and compute the average runtime; benchmark across multiple iteration thresholds (the article's process/run entrypoints take two numeric arguments, e.g. 1000 1000). Compile the averaged runtimes into one row per sample count and save as benchmark_timing_table.csv. The expected result is quadratic scaling with sample count, peak performance for n <= 500, and degraded performance beyond 1000 samples, which defines the recommended operating envelope.

## Related tools

- **CATHAI** (Application whose graph-generation and plotting performance is benchmarked via its Docker process entrypoint) — https://github.com/FordeGenomics/cathai
- **Docker** (Used to build the CATHAI image and run the data-processing/graph-generation step with bind-mounted data for each timing measurement)
- **Python 3.9** (Runtime environment (via conda env 'cathai') in which the processing and timing workload executes)
- **conda** (Creates the cathai environment (python=3.9, uwsgi=2.0, redis=5.0) supporting the benchmarked pipeline)

## Examples

```
docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process
```

## Evaluation signals

- The reproduced benchmark_timing_table.csv shows averaged runtimes computed from 10 repeated measurements per sample count
- Averaged runtimes exhibit quadratic growth with sample count rather than linear growth
- Timing table indicates peak performance for n <= 500 and clearly degraded performance beyond 1000 samples
- Each timing run used the documented Docker invocation with the data directory bind-mounted and the process entrypoint (e.g. 'cathai 1000 1000 process')
- Validation confirms the measured operating envelope matches the reported one: usable operation at n <= 500, above-1000 samples not recommended

## Limitations

- CATHAI graph generation and plotting scale quadratically with sample count, so runtimes grow quickly at large n
- Sample counts above 1000 are not recommended for usable operation of the tool
- Shiny server should only be bound on 127.0.0.1 or firewalled, per README security notes, which constrains how the benchmarked webapp can be exposed during testing

## Evidence

- [methods] docker image build --file ./docker/Dockerfile --tag cathai ./docker: "docker image build --file ./docker/Dockerfile --tag cathai ./docker"
- [readme] docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process: "docker run -p 8080:5000 --mount type=bind,source="./shiny/data/",target="/opt/cathai/shiny/data" cathai 1000 1000 process"
- [methods] conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0: "conda create -n cathai -c conda-forge python=3.9 uwsgi=2.0 redis=5.0"
- [results] CATHAI graph generation and plotting scale quadratically with sample count, with peak performance for n <= 500 and sample counts above 1000 not recommended; benchmarking used 10 averaged repetitions across iteration thresholds.: [evidence span withheld — anchor status: not_found]
