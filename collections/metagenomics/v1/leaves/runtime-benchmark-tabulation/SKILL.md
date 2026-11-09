---
name: runtime-benchmark-tabulation
description: Use when comparing end-to-end or per-stage runtimes of two analysis pipelines
  on identical small input sets (e.g. five 50,000-sequence metatranscriptomes) to
  quantify a speed-up factor.
when_to_use_negative:
- The goal is annotation accuracy or genus-level accuracy comparison (95% vs 91%)
  rather than runtime — use an accuracy benchmarking workflow instead
- The inputs are large production samples (millions of sequences) where web-service
  baselines like MG-RAST may be oversubscribed and slow, distorting the comparison
- Only a single pipeline is being timed with no baseline to tabulate against — a simple
  profiler suffices
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SAMSA2
  role: Standalone pipeline being benchmarked; master_script.bash executes preprocessing,
    annotation, and aggregation stages whose wall-clock times are recorded
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner (version 0.8.3) whose fast local annotation
    is the primary source of the SAMSA2 speed-up
  repo: https://github.com/bbuchfink/diamond
- name: MG-RAST
  role: Web service used as the SAMSA 1.0 baseline runtime for the same five 50,000-sequence
    files
- name: SAMSA
  role: Original pipeline (version 1.0) whose MG-RAST-dependent runtimes serve as
    the comparison baseline
- name: Trimmomatic
  role: Read cleaning step within the preprocessing stage whose per-stage runtime
    is tabulated
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: PEAR
  role: Paired-end merging step within the preprocessing stage whose per-stage runtime
    is tabulated
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: SortMeRNA
  role: Ribosomal read filtering step within the preprocessing stage whose per-stage
    runtime is tabulated
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:2b179f97f7fa10eccd4dcc6bccd99376b79fc945c38a0195beb22cda1ee32d74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - DIAMOND
  - MG-RAST
  - PEAR
  - SAMSA
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# runtime-benchmark-tabulation

## Summary

A benchmarking skill for reconstructing the SAMSA2 vs SAMSA 1.0 runtime comparison table, tabulating per-component wall-clock times (preprocessing; RefSeq; SEED Subsystems; CAZy) across five 50,000-sequence packaged metatranscriptomes to quantify the speed-up delivered by DIAMOND-based local annotation.

## When to use

Use when comparing end-to-end or per-stage runtimes of two analysis pipelines on identical small input sets (e.g. five 50,000-sequence metatranscriptomes) to quantify a speed-up factor.

## When NOT to use

- The goal is annotation accuracy or genus-level accuracy comparison (95% vs 91%) rather than runtime — use an accuracy benchmarking workflow instead
- The inputs are large production samples (millions of sequences) where web-service baselines like MG-RAST may be oversubscribed and slow, distorting the comparison
- Only a single pipeline is being timed with no baseline to tabulate against — a simple profiler suffices

## Inputs

- Five 50,000-sequence sample metatranscriptome FASTQ files packaged in the SAMSA2 repository
- Local DIAMOND-indexed reference databases: NCBI RefSeq, SEED Subsystems, CAZy
- MG-RAST web service runtime records for the same five files (SAMSA 1.0 baseline)

## Outputs

- Side-by-side runtime table with per-component times (preprocessing; RefSeq; SEED Subsystems; CAZy) and totals for SAMSA2 and SAMSA 1.0
- Computed speed-up of SAMSA2 total runtime (7 h 41 min) vs SAMSA 1.0 total runtime (1 day 19 h 45 min)
- Per-stage wall-clock time log for each of the five benchmark samples

## How to apply

Clone the SAMSA2 repository (github.com/transcript/samsa2) and use its packaged five 50,000-sequence sample metatranscriptomes as the benchmark input set. Run master_script.bash — which performs preprocessing (Trimmomatic read cleaning, PEAR paired-end merging, SortMeRNA rRNA filtering), DIAMOND v0.8.3 annotation against local RefSeq, SEED Subsystems, and CAZy databases, and aggregation with DIAMOND_analysis_counter.py — recording per-stage wall-clock times for each of the five samples. Obtain the SAMSA 1.0 baseline by submitting the same five files to the MG-RAST web service, noting that MG-RAST runtimes are longer when permission to make data public immediately is not granted. Tabulate per-component runtimes (preprocessing; RefSeq; SEED Subsystems; CAZy) side by side for both pipelines and compute the SAMSA2 speed-up attributable to the DIAMOND aligner and local databases.

## Related tools

- **SAMSA2** (Standalone pipeline being benchmarked; master_script.bash executes preprocessing, annotation, and aggregation stages whose wall-clock times are recorded) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner (version 0.8.3) whose fast local annotation is the primary source of the SAMSA2 speed-up) — https://github.com/bbuchfink/diamond
- **MG-RAST** (Web service used as the SAMSA 1.0 baseline runtime for the same five 50,000-sequence files)
- **SAMSA** (Original pipeline (version 1.0) whose MG-RAST-dependent runtimes serve as the comparison baseline)
- **Trimmomatic** (Read cleaning step within the preprocessing stage whose per-stage runtime is tabulated) — http://www.usadellab.org/cms/?page=trimmomatic
- **PEAR** (Paired-end merging step within the preprocessing stage whose per-stage runtime is tabulated) — https://sco.h-its.org/exelixis/web/software/pear/
- **SortMeRNA** (Ribosomal read filtering step within the preprocessing stage whose per-stage runtime is tabulated) — http://bioinfo.lifl.fr/RNA/sortmerna/

## Examples

```
git clone https://github.com/transcript/samsa2.git && bash setup_and_test/package_installation.bash && bash master_script.bash
```

## Evaluation signals

- Reconstructed table must reproduce the reported totals: SAMSA2 average 7 h 41 min vs SAMSA 1.0 total 1 day 19 h 45 min across the five 50,000-sequence samples
- CAZy-only mapping should complete in under 30 minutes
- Mapping to large databases (RefSeq and SEED Subsystems) should be the most time-consuming SAMSA2 stages in the per-component breakdown
- Each of the five samples must have wall-clock times recorded for all stages: preprocessing, annotation (RefSeq, SEED Subsystems, CAZy), and aggregation
- SAMSA 1.0 baseline runtimes should reflect the MG-RAST web service, with the caveat that reported times are longer if permission to make data public immediately was not granted

## Limitations

- SAMSA 1.0 runtimes via MG-RAST are longer if permission is not given to make the data public immediately, confounding the baseline
- Web-based baselines like MG-RAST may become oversubscribed and slow, so runtime comparisons are not fully controlled
- DIAMOND is set to fast mode by default, prioritizing speed over accuracy; switching to --sensitive may significantly increase memory requirements and runtime
- Pipeline was only tested on Linux despite claiming any OS supporting Python 2.7

## Evidence

- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [discussion] SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately: "SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately"
- [methods] master script performs first 3 of 4 steps: "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
- [readme] DIAMOND fast mode by default: [evidence span withheld — anchor status: external_doc]
- [readme] repo clone command: "git clone https://github.com/transcript/samsa2.git"
