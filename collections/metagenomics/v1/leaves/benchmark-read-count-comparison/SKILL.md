---
name: benchmark-read-count-comparison
description: Decision support for choosing a 16S paired-end read processing pipeline
  when you have raw paired-end FASTQ data and want to compare how many total reads
  (and OTUs) each candidate pipeline (merging-free CD-HIT-based vs merging-based)
  retains across benchmark datasets.
when_to_use_negative:
- You already have a finalized feature/OTU table and only need taxonomy or diversity
  analysis — no raw-read count comparison is possible
- The comparison targets OTU counts rather than read counts; note CDSnake outputs
  fewer OTUs than DADA2 and deblur, so read-count benchmarks should not be conflated
  with OTU-count accuracy benchmarks
- Your reads are single-end or non-overlapping pairs; the merging-free rationale applies
  to highly overlapping paired-end reads (e.g. 2 x 250 bp)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: CDSnake
  role: Benchmarked Snakemake pipeline whose merging-free CD-HIT-OTU-Miseq total read
    counts are compared against merging-based pipelines
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Merging-based baseline pipeline (run in QIIME2) whose total read counts are
    compared as a lower-preservation benchmark
- name: deblur
  role: Merging-based baseline pipeline (run in QIIME2) whose total read counts are
    compared as a lower-preservation benchmark
- name: QIIME2
  role: Workflow in which the DADA2 and deblur baselines are run for the benchmark
- name: pandas
  role: Used to extract total-count columns, rank pipelines per dataset, and compute
    fold differences in the comparison workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:60761216b2bf18a85ad5338754ab20d9c810364595c1605655b178bf4d89aa56
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - QIIME2
  - deblur
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# benchmark-read-count-comparison

## Summary

A benchmarking skill for comparing total outputted read counts across 16S rRNA OTU-retrieval pipelines (CDSnake/CD-HIT-OTU-Miseq vs. merging-based DADA2 and deblur) over paired-end mock datasets, to quantify how much raw data each pipeline preserves.

## When to use

Decision support for choosing a 16S paired-end read processing pipeline when you have raw paired-end FASTQ data and want to compare how many total reads (and OTUs) each candidate pipeline (merging-free CD-HIT-based vs merging-based) retains across benchmark datasets.

## When NOT to use

- You already have a finalized feature/OTU table and only need taxonomy or diversity analysis — no raw-read count comparison is possible
- The comparison targets OTU counts rather than read counts; note CDSnake outputs fewer OTUs than DADA2 and deblur, so read-count benchmarks should not be conflated with OTU-count accuracy benchmarks
- Your reads are single-end or non-overlapping pairs; the merging-free rationale applies to highly overlapping paired-end reads (e.g. 2 x 250 bp)

## Inputs

- Benchmark results table of total outputted read counts for CDSnake, DADA2 (QIIME2), and deblur across mock datasets (Balanced, HMP, Extreme, All)
- Raw paired-end 16S FASTQ reads (2 x 250 bp, highly overlapping) from mock community datasets
- Total counts from merged/OTU-table outputs of each pipeline (e.g. OTU.txt from the CDSnake pooled folder)

## Outputs

- total_read_counts_comparison.csv with columns: dataset, pipeline, total_reads, rank, fold_vs_max
- Per-dataset ranking showing whether CDSnake has the highest total count on each mock dataset

## How to apply

Load the benchmark table containing total read counts for each pipeline (CDSnake, DADA2 in QIIME2, and deblur) across the mock dataset rows (Balanced, HMP, Extreme, and the pooled 'All' row). Extract the total-count column per pipeline with pandas (e.g. All row: CDSnake 2,231,154 vs DADA2 1,392,444 vs deblur 224,388 reads), rank the pipelines within each dataset row, and compute fold differences versus the maximum (e.g. CDSnake over deblur). Verify whether the merging-free CDSnake pipeline, which avoids reads merging via separate clustering of paired reads, has the highest total count in every row — the Extreme dataset is the known exception where DADA2 slightly exceeds it. Write the per-dataset comparison to a CSV with columns dataset, pipeline, total_reads, rank, and fold_vs_max. Rationale: merging-based pipelines must exclude incorrectly or uncertainly merged read pairs, whereas CD-HIT-OTU-Miseq discards a smaller portion of reads during OTU retrieval, so total counts are a direct measure of raw-data preservation.

## Related tools

- **CDSnake** (Benchmarked Snakemake pipeline whose merging-free CD-HIT-OTU-Miseq total read counts are compared against merging-based pipelines) — github.com/ydkondratenko/cdsnake
- **DADA2** (Merging-based baseline pipeline (run in QIIME2) whose total read counts are compared as a lower-preservation benchmark)
- **deblur** (Merging-based baseline pipeline (run in QIIME2) whose total read counts are compared as a lower-preservation benchmark)
- **QIIME2** (Workflow in which the DADA2 and deblur baselines are run for the benchmark)
- **pandas** (Used to extract total-count columns, rank pipelines per dataset, and compute fold differences in the comparison workflow)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('benchmark_totals.csv'); df['rank'] = df.groupby('dataset')['total_reads'].rank(ascending=False); df['fold_vs_max'] = df.groupby('dataset')['total_reads'].transform(lambda s: s.max()/s); df.to_csv('total_read_counts_comparison.csv', index=False)"
```

## Evaluation signals

- On the pooled 'All' row, CDSnake total counts (2,231,154) exceed DADA2 (1,392,444) and deblur (224,388) — the expected ordering
- CDSnake ranks first on Balanced, HMP, and All rows; Extreme is the single allowed exception where DADA2 is slightly higher
- Output CSV contains exactly the columns dataset, pipeline, total_reads, rank, fold_vs_max with one row per dataset-pipeline pair
- Fold-vs-max equals 1.0 for the top-ranked pipeline in each dataset row and >1.0 for the others (e.g. ~9.96 for CDSnake over deblur on All)
- Pipelines under comparison are consistent across datasets (the same three: CDSnake, DADA2, deblur)

## Limitations

- The Extreme dataset is the only benchmark row where DADA2 slightly exceeds CDSnake in total counts, so 'CDSnake always wins' is not a valid invariant
- CDSnake is generally slower than DADA2 and deblur, so total-count preservation comes at a runtime cost
- CDSnake outputs fewer OTUs than DADA2 and deblur; high read counts should not be misread as more (or more accurate) taxa
- No accessions or availability details are given for the three mock community datasets, and no software versions or benchmarking hardware are specified, limiting reproducibility of the comparison

## Evidence

- [abstract] CDSnake preserves more raw paired-end data: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [intro] Merging-free rationale for higher counts: "Incorrectly or uncertainly merged reads either have to be excluded from downstream analysis or retained with high risk of spurious sequences creation."
- [abstract] Mock datasets benchmarked: "Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme"
- [readme] Trimmomatic MINLEN filtering affects retained reads: "Trimmomatic will try to use THREADS number of threads and will discard cropped reads if they become shorter than 100 nucleotides."
- [results] Per-dataset ranking workflow step: "on Balanced and HMP datasets number of OTUs outputted by CDSnake was closer to real number of strains which were used for mock community generation"
