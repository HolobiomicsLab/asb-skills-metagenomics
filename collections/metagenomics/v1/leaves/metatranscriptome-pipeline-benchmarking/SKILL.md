---
name: metatranscriptome-pipeline-benchmarking
description: Use when you need to quantify runtime or throughput of a standalone metatranscriptome
  analysis pipeline against a web-service baseline (e.g., SAMSA2 vs MG-RAST/SAMSA
  1.0) on a fixed set of test files.
when_to_use_negative:
- The goal is annotating a single large production metatranscriptome rather than measuring
  comparative runtime — run the pipeline directly instead of benchmarking
- The data are already processed feature/abundance tables rather than raw FASTQ reads,
  so no preprocessing or DIAMOND annotation stage is involved
- A web-service workflow (e.g., MG-RAST account-based) is acceptable and no local
  database installation is desired
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_3174
tools:
- name: SAMSA2
  role: Standalone metatranscriptome pipeline being benchmarked; runs preprocessing,
    DIAMOND annotation, and aggregation steps
  repo: https://github.com/transcript/samsa2
- name: DIAMOND
  role: Accelerated BLAST-like aligner (version 0.8.3) used for annotation against
    local RefSeq, SEED Subsystems, and CAZy databases; primary source of SAMSA2's
    speed advantage
  repo: https://github.com/bbuchfink/diamond
- name: Trimmomatic
  role: Read cleaner used in the SAMSA2 preprocessing stage; its runtime is timed
    as part of the benchmark
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: PEAR
  role: Paired-end read merger in the SAMSA2 preprocessing stage
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: SortMeRNA
  role: rRNA-filtering step in SAMSA2 preprocessing that speeds up downstream annotation
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
- name: MG-RAST
  role: Web service used to obtain SAMSA 1.0 baseline runtimes on the same five sample
    files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:fb2c9480d90f50b7fcdc96e0953a74932e0f65329e3f4742cd5012acaac17258
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# metatranscriptome-pipeline-benchmarking

## Summary

A benchmarking procedure comparing the standalone SAMSA2 metatranscriptome pipeline against the MG-RAST-dependent SAMSA 1.0, using five packaged 50,000-sequence sample metatranscriptomes and per-component wall-clock timings (preprocessing, RefSeq, SEED Subsystems, CAZy annotation).

## When to use

Use when you need to quantify runtime or throughput of a standalone metatranscriptome analysis pipeline against a web-service baseline (e.g., SAMSA2 vs MG-RAST/SAMSA 1.0) on a fixed set of test files.

## When NOT to use

- The goal is annotating a single large production metatranscriptome rather than measuring comparative runtime — run the pipeline directly instead of benchmarking
- The data are already processed feature/abundance tables rather than raw FASTQ reads, so no preprocessing or DIAMOND annotation stage is involved
- A web-service workflow (e.g., MG-RAST account-based) is acceptable and no local database installation is desired

## Inputs

- Five sample metatranscriptome FASTQ files of 50,000 sequences each (packaged in the SAMSA2 repository)
- Locally DIAMOND-indexed reference databases: NCBI RefSeq, SEED Subsystems, CAZy
- SAMSA 1.0 baseline runtimes obtained via the MG-RAST web service

## Outputs

- Per-component wall-clock time table (preprocessing; RefSeq; SEED Subsystems; CAZy) for SAMSA2 vs SAMSA 1.0
- Total pipeline runtimes per sample and average across the five samples
- Computed speed-up factor of SAMSA2 over SAMSA 1.0

## How to apply

Clone the SAMSA2 repository and use its five packaged 50,000-sequence sample metatranscriptomes as the benchmark input set. Run master_script.bash, which performs the first three of four pipeline steps (preprocessing with PEAR/Trimmomatic/SortMeRNA, DIAMOND v0.8.3 annotation against local RefSeq, SEED Subsystems, and optionally CAZy databases, and aggregation with DIAMOND_analysis_counter.py), recording per-stage wall-clock times per sample. Obtain the SAMSA 1.0 baseline by submitting the same five files to the MG-RAST web service, noting runtimes are longer if permission to make data public immediately is not granted. Tabulate per-component and total runtimes side by side and compute the speed-up; validation targets are SAMSA2 total 7 h 41 min vs SAMSA 1.0 total 1 day 19 h 45 min, with CAZy-only mapping completing in under 30 minutes.

## Related tools

- **SAMSA2** (Standalone metatranscriptome pipeline being benchmarked; runs preprocessing, DIAMOND annotation, and aggregation steps) — https://github.com/transcript/samsa2
- **DIAMOND** (Accelerated BLAST-like aligner (version 0.8.3) used for annotation against local RefSeq, SEED Subsystems, and CAZy databases; primary source of SAMSA2's speed advantage) — https://github.com/bbuchfink/diamond
- **Trimmomatic** (Read cleaner used in the SAMSA2 preprocessing stage; its runtime is timed as part of the benchmark) — http://www.usadellab.org/cms/?page=trimmomatic
- **PEAR** (Paired-end read merger in the SAMSA2 preprocessing stage) — https://sco.h-its.org/exelixis/web/software/pear/
- **SortMeRNA** (rRNA-filtering step in SAMSA2 preprocessing that speeds up downstream annotation) — http://bioinfo.lifl.fr/RNA/sortmerna/
- **MG-RAST** (Web service used to obtain SAMSA 1.0 baseline runtimes on the same five sample files)

## Examples

```
git clone https://github.com/transcript/samsa2.git; bash master_script.bash  # run the five 50,000-sequence sample FASTQ files and time each stage
```

## Evaluation signals

- Reconstructed SAMSA2 total runtime matches the reported 7 h 41 min average across the five 50,000-sequence samples
- SAMSA 1.0 (MG-RAST) baseline total matches the reported 1 day 19 h 45 min
- CAZy-only mapping completes in under 30 minutes per sample
- Per-stage timings show database mapping steps (RefSeq, SEED Subsystems) as the most time-consuming SAMSA2 components
- All four pipeline phases (preprocessing, annotation, aggregation, analysis) complete without checkpoint-file interruptions on every sample

## Limitations

- SAMSA 1.0 / MG-RAST runtimes are longer if permission is not given to MG-RAST to make the data public immediately, complicating a fair baseline
- Benchmark was performed with DIAMOND version 0.8.3 and results depend on that version and DIAMOND running in --fast mode; --sensitive mode may significantly increase memory requirements and runtime
- Pipeline was tested only on Linux despite claiming support for any OS running Python 2.7; aggregation scripts require Python 2

## Evidence

- [abstract] SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases.: "SAMSA2 is faster due to the use of the DIAMOND aligner, and more flexible and reproducible because it uses local databases."
- [methods] Pipeline steps are the first 3 of 4 in master_script: "use DIAMOND to annotate your reads against a database of your choosing (note that database must be local and DIAMOND-indexed)"
- [discussion] SAMSA 1.0 runtimes are longer without immediate public release: "SAMSA 1.0 runtimes are longer if permission is not given to MG-RAST to make the data public immediately"
- [readme] master_script performs preprocessing, annotation, aggregation: "Make changes to the master_script.bash, which performs the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
- [readme] rRNA filtering provides additional speed: "SortMeRNA - removes ribosomal reads, as these don't contribute to the mRNA profile of the metatranscriptome."
