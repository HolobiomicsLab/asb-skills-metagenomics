---
name: sweep-results-tabulation
description: Use when a workflow sweeps a single tool parameter (e.g., MMseqs2 sensitivity
  1–7.5) over identical inputs and per-run outputs (counts, quality scores) must be
  assembled into one tabular CSV for range-based validation.
when_to_use_negative:
- The analysis varies more than one tool parameter simultaneously — the table assumes
  a one-dimensional sweep where only a single parameter differs between runs
- Per-run outputs are already aggregated in a table or database; re-tabulating adds
  no value
- The task is to run or parameterize the clustering itself (e.g., Snekmer's cluster
  mode or MMseqs2 execution) rather than to record and validate its sweep results
tools:
- name: Python
  role: Assembles per-sensitivity coverage and Pfam V values into the tabular summary
    (pandas DataFrame) and writes sensitivity_sweep_table.csv
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: MMseqs2
  role: Upstream tool whose easy-cluster runs, swept across sensitivity settings,
    produce the protein coverage and Pfam V values being tabulated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:94cbcc173c430d4316a3a9fca2daeec1be42f483035ab8f699a203aeb5e15a84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - MMseqs2
  - Python
  - Snekmer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# sweep-results-tabulation

## Summary

Tabulate per-parameter results from a method parameter sweep into a single CSV summary, recording each swept parameter value alongside its measured outputs so that the sweep's stability can be assessed and validated against reported ranges.

## When to use

Use when a workflow sweeps a single tool parameter (e.g., MMseqs2 sensitivity 1–7.5) over identical inputs and per-run outputs (counts, quality scores) must be assembled into one tabular CSV for range-based validation.

## When NOT to use

- The analysis varies more than one tool parameter simultaneously — the table assumes a one-dimensional sweep where only a single parameter differs between runs
- Per-run outputs are already aggregated in a table or database; re-tabulating adds no value
- The task is to run or parameterize the clustering itself (e.g., Snekmer's cluster mode or MMseqs2 execution) rather than to record and validate its sweep results

## Inputs

- Per-run metric records from each sweep setting: one (parameter value, primary metric, secondary metric) triple per run
- Sweep parameter values list (e.g., MMseqs2 sensitivity 1, 2, ..., 7.5)
- Optional reference annotations used for quality metrics (e.g., Pfam domain annotations for the clustered protein set)

## Outputs

- sensitivity_sweep_table.csv — a tabular summary with one row per swept parameter value, columns for protein coverage and Pfam V score

## How to apply

Fix the input dataset and run the target tool once per parameter value, varying only the swept parameter between runs (e.g., MMseqs2 easy-cluster sensitivity values from 1 to 7.5 on the same multi-species FASTA protein set). For each run, record the primary metric (e.g., number of clustered proteins, i.e., protein coverage) and any secondary quality metric (e.g., Pfam V score against Pfam annotations) alongside the parameter setting. Assemble the per-parameter metrics into a two-column summary using Python with pandas and write it to a CSV such as sensitivity_sweep_table.csv. The rationale is that a flat table makes the sweep directly auditable: expected metric ranges (e.g., coverage 20,240–28,977 proteins, Pfam V 0.90–0.92) can be checked mechanically rather than by re-running analyses.

## Related tools

- **Python** (Assembles per-sensitivity coverage and Pfam V values into the tabular summary (pandas DataFrame) and writes sensitivity_sweep_table.csv) — https://github.com/PNNL-CompBio/Snekmer
- **MMseqs2** (Upstream tool whose easy-cluster runs, swept across sensitivity settings, produce the protein coverage and Pfam V values being tabulated)

## Examples

```
import pandas as pd; df = pd.DataFrame({'sensitivity': svals, 'protein_coverage': coverages, 'pfam_v': vscores}); df.to_csv('sensitivity_sweep_table.csv', index=False)
```

## Evaluation signals

- Protein coverage column falls within the reported range of 20,240 to 28,977 proteins across swept sensitivities
- Pfam V score column falls within 0.90 to 0.92 across swept sensitivities
- Table has exactly one row per swept parameter value, with no missing or duplicated sensitivity settings
- Only the sensitivity parameter differs between the runs contributing rows to the table (identical protein input)
- File sensitivity_sweep_table.csv parses cleanly with pandas.read_csv and contains the expected columns

## Limitations

- Validation ranges are specific to the 10-genome protein set and the MMseqs2 sensitivity 1–7.5 sweep; other datasets or parameter ranges will have different valid ranges
- The skill records results only; it does not execute the sweep runs or compute the underlying metrics (protein coverage counts, Pfam V evaluation)
- Exact-match kmer representations may be limited in sensitivity and coverage, so analogous sweeps on other clustering approaches may not exhibit the tight 0.90–0.92 quality-score stability seen here

## Evidence

- [methods] Assemble the per-sensitivity coverage and Pfam V values into a tabular summary with Python (pandas) and write it to sensitivity_sweep_table.csv.: [evidence span withheld — anchor status: not_found]
- [results] protein coverage ranged from a minimum of 20,240 to a maximum of 28,977 proteins, and the Pfam V score ranged only from 0.90 to 0.92.: [evidence span withheld — anchor status: not_found]
- [methods] run MMseqs2 easy-cluster on the identical protein input so that only the clustering sensitivity parameter varies between runs: [evidence span withheld — anchor status: not_found]
- [abstract] Snekmer is written in Python using Snakemake.: "Snekmer is written in Python using Snakemake."
