---
name: abundance-l1-norm-error-quantification
description: Use when comparing taxonomic profiling tools on simulated metagenome
  samples and you need a per-sample abundance estimation error metric (L1-norm) alongside
  F1-score, from AMBER rank/cumulative results.
when_to_use_negative:
- The task is taxonomic binning evaluation, not profiling — AMBER's binning metrics
  (purity, completeness, Rand index) apply instead
- You have no ground-truth read-to-taxon mapping, so abundance error cannot be computed
  against known composition
- Input is real (non-simulated) data where the true per-species abundances are unknown
  and the L1-norm error is undefined
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: AMBER
  role: Evaluates per-sample predicted profiles against ground truth, producing rank-based
    and cumulative results from which the L1-norm abundance error is extracted
  repo: github.com/CAMI-challenge/AMBER
- name: ganon2
  role: Profiling tool under evaluation; its .rank.tsv outputs supply the L1-norm
    error compared against kraken2+bracken and kmcp
  repo: https://github.com/pirovc/ganon
- name: kraken2+bracken
  role: Comparator profiling tool (kraken2 v2.1.3, bracken v2.9) whose abundance estimates
    are scored with the same L1-norm metric
- name: Snakemake
  role: Drives the classify pipeline that produces the profiles and the aggregation
    of .rank.tsv/.stats metrics into the results table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:4502a1390472ca5668c6211f9577f3bbeba738c44eab8bc46cf5e876eb16d3ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMBER
  - Snakemake
  - conda
  - ganon2
  - kmcp
  - kraken2+bracken
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# abundance-l1-norm-error-quantification

## Summary

Quantifies the L1-norm error in abundance estimation from taxonomic profiling results by extracting cumulative abundance errors per sample from AMBER 2.0.21-beta .rank.tsv outputs, as used in the ganon2 benchmark on 16 simulated CAMI 1+2 samples.

## When to use

Use when comparing taxonomic profiling tools on simulated metagenome samples and you need a per-sample abundance estimation error metric (L1-norm) alongside F1-score, from AMBER rank/cumulative results.

## When NOT to use

- The task is taxonomic binning evaluation, not profiling — AMBER's binning metrics (purity, completeness, Rand index) apply instead
- You have no ground-truth read-to-taxon mapping, so abundance error cannot be computed against known composition
- Input is real (non-simulated) data where the true per-species abundances are unknown and the L1-norm error is undefined

## Inputs

- Predicted taxonomic profiles in bioboxes format (.profiling.bioboxes.gz) per tool/sample
- Ground-truth read-to-assembly/taxid mapping files (readid_assembly_taxid, gzipped)
- AMBER .rank.tsv files (rank-based and cumulative results) per sample per tool
- Tool .stats files with runtime and peak memory
- Simulated read sets from files/reads/ or the CAMI challenge data server

## Outputs

- Per-sample species-level L1-norm abundance errors per tool
- Per-sample species-level F1-scores (companion metric from the same .rank.tsv files)
- Aggregated profiling_benchmark_results.tsv combining L1 error, F1-score, runtime and peak memory

## How to apply

After profiling the 16 simulated samples (CAMI 1+2 reads) with each tool in default profiling mode with the minimum-abundance threshold applied, evaluate every predicted profile against the readid_assembly_taxid ground truth using AMBER 2.0.21-beta, requesting both rank-based (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) and cumulative results. From each tool's .rank.tsv output, collect the per-sample species-level L1-norm abundance errors together with the F1-scores, and pull runtimes/peak memory from the .stats files. Aggregate these into a single profiling_benchmark_results.tsv so that error and accuracy can be compared jointly — in the ganon2 study, improvements in F1-score median reached 0.35 while keeping a 'balanced L1-norm error in the abundance estimation'. Validate by checking the aggregated L1-norm error against expected targets, e.g. ganon2 L1-norm error should not exceed ~100 in any scenario on full RefSeq.

## Related tools

- **AMBER** (Evaluates per-sample predicted profiles against ground truth, producing rank-based and cumulative results from which the L1-norm abundance error is extracted) — github.com/CAMI-challenge/AMBER
- **ganon2** (Profiling tool under evaluation; its .rank.tsv outputs supply the L1-norm error compared against kraken2+bracken and kmcp) — https://github.com/pirovc/ganon
- **kraken2+bracken** (Comparator profiling tool (kraken2 v2.1.3, bracken v2.9) whose abundance estimates are scored with the same L1-norm metric)
- **Snakemake** (Drives the classify pipeline that produces the profiles and the aggregation of .rank.tsv/.stats metrics into the results table)

## Examples

```
snakemake -s metabench/classify.smk --configfile config/classify_test.yaml --cores 8 --use-conda
```

## Evaluation signals

- ganon2 L1-norm error stays under ~100 in all evaluated scenarios (as reported for the full RefSeq configuration)
- L1-norm errors are jointly compared with F1-score medians (ganon2 ~0.67 vs kraken2+bracken ~0.31 on full RefSeq), confirming a balanced error-accuracy profile
- Values are parsed from AMBER .rank.tsv files and aggregated into profiling_benchmark_results.tsv without missing samples (16 samples per tool)
- Runtimes and peak memory in the aggregated table match the .stats files (e.g. ~43 min ganon2, ~59 min kraken2+bracken, ~4 h 47 min kmcp for 16 RefSeq RG samples)

## Limitations

- The L1-norm error is only defined against simulated samples with known ground truth; the ganon2 evaluation used 16 simulated samples from CAMI 1+2 studies
- The metric is species/rank-level and depends on the profiling tool's minimum-abundance threshold; tools run in default profiling mode for comparability
- AMBER version must match (2.0.21-beta); the ganon_benchmark pipeline requires setting the 'tools_path' to AMBER and installing dependencies via conda

## Evidence

- [abstract] In profiling, improvements in the F1-score median are up to 0.35, keeping a balanced L1-norm error in the abundance estimation: "In profiling, improvements in the F1-score median are up to 0.35, keeping a balanced L1-norm error in the abundance estimation"
- [methods] AMBER version pinned in the benchmark pipeline: "AMBER==2.0.21-beta (https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz)"
- [methods] 1M read subset provided for evaluation: "A sub-set of a random 1 million reads from the complete set used to evaluate the tools is provided in this repository under `files/reads/`"
- [readme] MetaBench profiling evaluation scope: [evidence span withheld — anchor status: external_doc]
