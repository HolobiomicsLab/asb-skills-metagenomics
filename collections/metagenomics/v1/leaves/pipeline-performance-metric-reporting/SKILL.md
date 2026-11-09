---
name: pipeline-performance-metric-reporting
description: Use when benchmarking or comparing the time performance of multiple OTU-retrieval
  pipelines (e.g. DADA2, deblur, CD-HIT-OTU-Miseq, CDSnake) across datasets, to populate
  a runtime comparison table in seconds.
when_to_use_negative:
- The goal is OTU-count accuracy or annotation correctness rather than runtime; that
  is reported in a separate table (OTUs, annotated, correctly annotated) and needs
  different validation
- Input data is already a feature/OTU table or non-16S data; runtime comparison of
  OTU-retrieval pipelines requires raw paired-end FASTQ reads
- Only one pipeline is being evaluated with no baselines; a comparative runtime table
  is meaningless without DADA2/deblur or source-CD-HIT reference runs
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: CDSnake
  role: Snakemake pipeline whose wall-clock runtime is reported and compared against
    alternatives
  repo: github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: Source utility suite benchmarked as a baseline (without Snakemake orchestration)
- name: DADA2
  role: Baseline OTU-retrieval pipeline (run via QIIME2) timed for comparison
- name: deblur
  role: Baseline OTU-retrieval pipeline (run via QIIME2) timed for comparison
- name: Snakemake
  role: Workflow manager launching CDSnake for the timed runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:28206a56965c9982b1c0e1f28dc70b8226973eb26ba274b895a3f2f3f4265346
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - Snakemake
  - deblur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pipeline-performance-metric-reporting

## Summary

Skill for extracting and reporting wall-clock runtime benchmarks (in seconds) for competing 16S OTU-retrieval pipelines, as done in Table 2 of the CDSnake paper, which timed DADA2, deblur, the source CD-HIT-OTU-Miseq utilities, and CDSnake across the HMP, Balanced, Extreme, and combined 'All' mock datasets.

## When to use

Use when benchmarking or comparing the time performance of multiple OTU-retrieval pipelines (e.g. DADA2, deblur, CD-HIT-OTU-Miseq, CDSnake) across datasets, to populate a runtime comparison table in seconds.

## When NOT to use

- The goal is OTU-count accuracy or annotation correctness rather than runtime; that is reported in a separate table (OTUs, annotated, correctly annotated) and needs different validation
- Input data is already a feature/OTU table or non-16S data; runtime comparison of OTU-retrieval pipelines requires raw paired-end FASTQ reads
- Only one pipeline is being evaluated with no baselines; a comparative runtime table is meaningless without DADA2/deblur or source-CD-HIT reference runs

## Inputs

- Paired-end 16S rRNA FASTQ reads from mock communities (Balanced, HMP, Extreme mock datasets)
- Same input folder used by all compared pipelines for fairness
- Pipeline configuration (edited Snakefile with INPUT_FOLDER, CDHIT, DATABASE, TRIMMOMATIC, p, q, THREADS, MINLEN parameters for CDSnake)

## Outputs

- Runtime table (Table 2) with time in seconds per pipeline (DADA2, deblur, source CD-HIT, CDSnake) per dataset (HMP, Balanced, Extreme, All)
- CSV such as table2_runtimes.csv with a header row and one data row per dataset

## How to apply

Run each pipeline end-to-end (DADA2 and deblur within QIIME2, the CD-HIT-OTU-Miseq utilities alone, and the CDSnake Snakemake pipeline) on the same paired-end 2 x 250 bp mock-community FASTQ inputs (HMP, Balanced, Extreme, and the pooled 'All' set). Record elapsed wall-clock time in seconds per pipeline per dataset, then tabulate as a table with one column per pipeline and one row per dataset (HMP: 4614.81 / 371.72 / 2149.59 / 2263.83 s; All: 17,509.96 / 376.05 / 39,775.51 / 39,891.53 s). Keep the same hardware/environment across all pipelines since the CDSnake article does not state versions or the execution environment. Interpretation: CDSnake is generally slower than DADA2 and deblur, but outputs higher total counts, preserving more information from raw data, so report time alongside output quantity/quality rather than in isolation.

## Related tools

- **CDSnake** (Snakemake pipeline whose wall-clock runtime is reported and compared against alternatives) — github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Source utility suite benchmarked as a baseline (without Snakemake orchestration))
- **DADA2** (Baseline OTU-retrieval pipeline (run via QIIME2) timed for comparison)
- **deblur** (Baseline OTU-retrieval pipeline (run via QIIME2) timed for comparison)
- **Snakemake** (Workflow manager launching CDSnake for the timed runs)

## Examples

```
snakemake --cores 8  # run CDSnake in the input folder to time the pipeline against DADA2, deblur, and source CD-HIT-OTU-Miseq baselines
```

## Evaluation signals

- Output table has exactly 4 data rows (HMP, Balanced, Extreme, All) and 4 pipeline columns (DADA2, deblur, source CD-HIT, CDSnake)
- Every runtime cell matches the published Table 2 exactly (e.g. HMP: 4614.81 / 371.72 / 2149.59 / 2263.83 s; All: 17,509.96 / 376.05 / 39,775.51 / 39,891.53 s)
- All values are positive numeric seconds; CDSnake times on HMP/All should exceed deblur times (CDSnake is generally slower)
- CSV is plain-text comma-separated with a header row (output_matches_reference against the published table)

## Limitations

- Hardware/execution environment for the time benchmarking is not specified in the article, limiting reproducibility of absolute times
- Software versions are not stated for DADA2, deblur, CD-HIT-OTU-MiSeq, or Snakemake
- No accessions or availability details are given for the three mock community datasets used in the benchmark

## Evidence

- [abstract] Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data.: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [discussion] Time benchmarking row: "Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake"
- [discussion] Table 2 fragment: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [readme] CDSnake launch command: "To run pipeline with default Snakemake settings, use command snakemake."
