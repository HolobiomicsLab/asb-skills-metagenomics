---
name: mock-community-benchmark-analysis
description: Use when paired-end 2 × 250 bp 16S mock community datasets are available
  and you need to rank OTU-retrieval pipelines by read preservation, OTU-count accuracy,
  or runtime.
when_to_use_negative:
- Input is already an OTU table or feature table rather than raw paired-end FASTQ
  reads that must be processed identically by each pipeline
- Real (non-mock) environmental samples where the true number and composition of strains
  is unknown, making OTU-count accuracy checks against ground truth impossible
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CDSnake
  role: Snakemake pipeline chaining CD-HIT-OTU-Miseq utilities to retrieve annotated
    OTUs from paired-end reads without merging; benchmarked for total counts and runtime
  repo: github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: Source utilities performing separate clustering of paired reads, avoiding
    the merging step; benchmarked directly and wrapped by CDSnake
- name: DADA2
  role: QIIME2-based OTU-retrieval pipeline used as a comparison baseline for counts
    and runtime
- name: deblur
  role: QIIME2-based OTU-retrieval pipeline used as a comparison baseline; fastest
    tool in every tested case
- name: QIIME2
  role: Workflow hosting the DADA2 and deblur comparison pipelines
- name: Trimmomatic
  role: Quality filtering within CDSnake (THREADS and MINLEN parameters; discards
    cropped reads shorter than 100 nt)
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:c2b8d834705b7f1cb3eb417a8cdfea7c4b9ae72ac5a83fbe56c5ae495f3a4df4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - QIIME2
  - Trimmomatic
  - deblur
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mock-community-benchmark-analysis

## Summary

Benchmark 16S amplicon pipelines (CDSnake/CD-HIT-OTU-Miseq, QIIME2-DADA2, deblur) on mock community datasets (Balanced, HMP, Extreme) by comparing total output read counts, OTU counts versus known strain numbers, and runtime in seconds.

## When to use

Use when paired-end 2 × 250 bp 16S mock community datasets are available and you need to rank OTU-retrieval pipelines by read preservation, OTU-count accuracy, or runtime.

## When NOT to use

- Input is already an OTU table or feature table rather than raw paired-end FASTQ reads that must be processed identically by each pipeline
- Real (non-mock) environmental samples where the true number and composition of strains is unknown, making OTU-count accuracy checks against ground truth impossible

## Inputs

- Paired-end 16S FASTQ reads for mock community datasets (Balanced, HMP, Extreme) with _R1_/_R2_ or _1/_2 filename conventions
- Benchmark result tables of total read counts per pipeline (CDSnake, DADA2, deblur) per dataset
- Runtime measurements in seconds for DADA2, deblur, source CD-HIT-OTU-MiSeq utilities, and CDSnake
- Known mock community strain composition (number and frequency of strains per dataset)

## Outputs

- total_read_counts_comparison.csv with columns dataset, pipeline, total_reads, rank, fold_vs_max
- pipeline_runtime_ranking.csv with per-dataset runtime ranks and flags for claimed patterns
- Per-dataset OTU-count comparison against true mock strain numbers

## How to apply

Load the per-pipeline, per-dataset benchmark tables (e.g. total read counts such as CDSnake 2,231,154 vs DADA2 1,392,444 vs deblur 224,388 on the pooled 'All' set, and runtimes in seconds such as HMP: DADA2 4614.81, deblur 1371.72, source CD-HIT 2149.59, CDSnake 2263.83). Run each pipeline on the same mock inputs — Balanced (57 strains), HMP (21 bacteria), Extreme (27 strains spanning five orders of magnitude) — then rank pipelines by total counts and runtime with pandas, computing fold differences versus the maximum. Compare OTU counts to the known number of mock strains (e.g. DADA2's 91 OTUs vs 59 actual strains on Balanced) to judge accuracy, and verify claimed patterns such as CDSnake yielding the highest total counts everywhere except Extreme, and deblur being fastest in every case. Write per-dataset ranking tables (columns like dataset, pipeline, total_reads, rank, fold_vs_max) to CSV with a flag for whether each expected pattern holds.

## Related tools

- **CDSnake** (Snakemake pipeline chaining CD-HIT-OTU-Miseq utilities to retrieve annotated OTUs from paired-end reads without merging; benchmarked for total counts and runtime) — github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Source utilities performing separate clustering of paired reads, avoiding the merging step; benchmarked directly and wrapped by CDSnake)
- **DADA2** (QIIME2-based OTU-retrieval pipeline used as a comparison baseline for counts and runtime)
- **deblur** (QIIME2-based OTU-retrieval pipeline used as a comparison baseline; fastest tool in every tested case)
- **QIIME2** (Workflow hosting the DADA2 and deblur comparison pipelines)
- **Trimmomatic** (Quality filtering within CDSnake (THREADS and MINLEN parameters; discards cropped reads shorter than 100 nt))

## Examples

```
cd input_folder && snakemake  # run CDSnake on paired-end 16S reads; OTU table written to pooled/OTU.txt
```

## Evaluation signals

- Computed ranks match Table 2 expectations: deblur has minimal runtime in every row, and each dataset row contains exactly four ranked pipelines
- CDSnake has the highest total read count on Balanced, HMP, and pooled 'All' rows (e.g. All: 2,231,154 vs DADA2 1,392,444 and deblur 224,388); Extreme is the only exception where DADA2 slightly exceeds CDSnake
- OTU counts on Balanced and HMP are closer to the true strain number (57 and 21) for CDSnake than for DADA2 or deblur (e.g. DADA2's 91 OTUs on Balanced)
- Fold-difference column (fold_vs_max) is finite and consistent with the reported per-dataset counts
- Output CSVs contain one row per pipeline per dataset with non-null total_reads and runtime values

## Limitations

- Software versions are not stated for DADA2, deblur, CD-HIT-OTU-MiSeq, or Snakemake, limiting exact reproducibility
- No accessions or availability details are given for the three mock community datasets, and the hardware/execution environment for time benchmarking is unspecified
- CDSnake is generally slower than the other pipelines, running slightly longer than the source CD-HIT-OTU-MiSeq utilities, and DADA2 slightly exceeds it on the Extreme dataset
- Reads must be unzipped with .fastq extension and filenames must contain _R1_/_R2_ or end with _1/_2 before running CDSnake

## Evidence

- [results] CDSnake yielded the highest total counts on most benchmarked mock datasets: [evidence span withheld — anchor status: not_found]
- [abstract] Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme: "Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme"
- [abstract] Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [results] Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude: "the Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude"
- [readme] Trimmomatic will discard cropped reads if they become shorter than 100 nucleotides: "Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides."
