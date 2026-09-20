---
name: cam-simulated-read-taxonomic-classification
description: Use when benchmarking taxonomic profiling tools (e.g. ganon2, kraken2+bracken,
  kmcp) on simulated read sets such as the 16 CAMI-derived samples, needing AMBER
  rank-based metrics and runtime comparisons.
when_to_use_negative:
- Analyzing real (non-simulated) reads without ground-truth read-to-taxid assignments
  — AMBER-based F1/L1 evaluation requires true taxonomic labels
- Evaluating assembly-level or binning (not read-level taxonomic profiling) workflows
  — separate classify-at-assembly-level and binning steps exist
- Input already consists of AMBER .rank.tsv or bioboxes profiles — the classification
  step is redundant; only the aggregation/evaluation part applies
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: ganon2
  role: Sequence classifier/profiler run in default profiling mode on the simulated
    samples
  repo: https://github.com/pirovc/ganon
- name: kraken2+bracken
  role: Comparison profiler run on the same RefSeq indices and samples
- name: kmcp
  role: Third profiler benchmarked on the same task
- name: Snakemake
  role: Drives the build and classify Snakemake pipelines with conda-managed dependencies
- name: AMBER
  role: Computes rank-based profiling metrics (sens, prec, f1s) and cumulative results
    from predicted vs. ground-truth profiles
  repo: https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- name: conda
  role: Manages all tool dependencies via --use-conda
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:bc4c8715ae868f1b93309037c17630bdfe80631d6e3c734288bcd556fcaf87e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pirovc/ganon
  tools:
  - AMBER
  - Snakemake
  - conda
  - ganon2
  - kmcp
  - kraken2+bracken
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cam-simulated-read-taxonomic-classification

## Summary

Taxonomically classify/profile simulated metagenomic reads (including CAMI 1+2 challenge samples) against RefSeq-based indices and evaluate the resulting profiles with AMBER, producing per-sample rank-based F1-scores, L1-norm abundance errors, and runtime statistics.

## When to use

Use when benchmarking taxonomic profiling tools (e.g. ganon2, kraken2+bracken, kmcp) on simulated read sets such as the 16 CAMI-derived samples, needing AMBER rank-based metrics and runtime comparisons.

## When NOT to use

- Analyzing real (non-simulated) reads without ground-truth read-to-taxid assignments — AMBER-based F1/L1 evaluation requires true taxonomic labels
- Evaluating assembly-level or binning (not read-level taxonomic profiling) workflows — separate classify-at-assembly-level and binning steps exist
- Input already consists of AMBER .rank.tsv or bioboxes profiles — the classification step is redundant; only the aggregation/evaluation part applies

## Inputs

- Simulated paired FASTQ read sets (16 samples from CAMI 1+2 and other studies; 1M-read subsets under files/reads/)
- Ground-truth files: readid_assembly_taxid.txt.gz per sample
- NCBI RefSeq reference sequences (.fna) and assembly_summary.txt
- NCBI taxdump files (nodes.dmp, names.dmp, merged.dmp)
- Tool databases/indexes (ganon2 .hibf/.tax, kraken2+bracken, kmcp)
- Snakemake configuration files (files/config/build_complete_eval.yaml, files/config/classify_toy_taxid.yaml)

## Outputs

- Per-tool per-sample taxonomic profiles in bioboxes format
- AMBER .rank.tsv files with rank-based metrics (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s)
- Cumulative AMBER evaluation results
- .stats files with runtime and peak memory per sample
- Aggregated profiling_benchmark_results.tsv (species-level F1-score, L1-norm error, runtime)

## How to apply

Unpack auxiliary ground-truth files (acc_len_taxid_assembly, assembly_summary.txt, taxdump .dmp, readid_assembly_taxid) with gzip -d and obtain simulated read sets from files/reads/ or the CAMI challenge data server. Build tool indices for the RefSeq configurations with the build Snakemake pipeline (build/Snakefile, files/config/build_complete_eval.yaml, --use-conda, 48 cores, -k). Classify the 16 samples at taxonomic level with the classify Snakemake pipeline (classify/Snakefile, files/config/classify_toy_taxid.yaml, --use-conda, 48 cores), invoking each tool in default profiling mode with the minimum-abundance threshold applied. Evaluate per-sample profiles with AMBER 2.0.21-beta to compute rank-based (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) and cumulative results. Collect species-level F1-scores and L1-norm errors from .rank.tsv files plus runtimes and peak memory from .stats files into profiling_benchmark_results.tsv, and check aggregated medians against reported targets (e.g. ganon2 median profiling F1 ~0.67 vs ~0.31 for kraken2+bracken on full RefSeq).

## Related tools

- **ganon2** (Sequence classifier/profiler run in default profiling mode on the simulated samples) — https://github.com/pirovc/ganon
- **kraken2+bracken** (Comparison profiler run on the same RefSeq indices and samples)
- **kmcp** (Third profiler benchmarked on the same task)
- **Snakemake** (Drives the build and classify Snakemake pipelines with conda-managed dependencies)
- **AMBER** (Computes rank-based profiling metrics (sens, prec, f1s) and cumulative results from predicted vs. ground-truth profiles) — https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- **conda** (Manages all tool dependencies via --use-conda)

## Examples

```
snakemake -s classify/Snakefile --configfile files/config/classify_toy_taxid.yaml --cores 48 --use-conda -k
```

## Evaluation signals

- ganon2 median species-level profiling F1-score reaches ~0.67 on full RefSeq vs ~0.31 for kraken2+bracken (50% of samples above median)
- ganon2 L1-norm abundance error stays at or below ~100 in all scenarios
- RefSeq RG runtimes approximate ~43 min (ganon2), ~59 min (kraken2+bracken), ~4 h 47 min (kmcp) over all 16 samples
- All .rank.tsv files contain the expected AMBER columns (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) for each of the 16 samples
- profiling_benchmark_results.tsv aggregates one row per tool/sample with non-empty F1, L1-norm error, and runtime values

## Limitations

- Results depend on reference database version — outdated or incomplete references lead to underperformance or inconsistent results; RefSeq grows ~20% annually
- Evaluation is limited to 16 simulated samples from specific studies (CAMI 1+2 etc.); complete read sets must be fetched separately from the CAMI data server
- The repository only ships a 1-million-read random subset under files/reads/, so full reproduction requires downloading complete read sets

## Evidence

- [readme] The pipeline is divided into build and classify steps.: "The pipeline is divided into build and classify steps."
- [methods] AMBER dependency and evaluation tool pinning: "AMBER==2.0.21-beta (https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz)"
- [methods] Provided read subset location: "A sub-set of a random 1 million reads from the complete set used to evaluate the tools is provided in this repository under `files/reads/`"
- [abstract] Profiling improvement finding: "In profiling, improvements in the F1-score median are up to 0.35, keeping a balanced L1-norm error in the abundance estimation"
- [readme] Ground-truth unpacking step: "# unpack gt files
	gzip -d files/reads/cami/*/*readid_assembly_taxid.txt.gz"
