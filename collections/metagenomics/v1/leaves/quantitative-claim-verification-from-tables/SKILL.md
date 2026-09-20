---
name: quantitative-claim-verification-from-tables
description: Use when a paper asserts one pipeline outperforms others on a count-based
  metric (e.g. total reads retained) and the supporting numbers are available in a
  benchmark table that can be re-tabulated, ranked, and cross-checked row by row.
when_to_use_negative:
- The claim concerns OTU counts or annotation correctness rather than total read counts
  — the paper's finding that 'CDSnake outputted less OTUs than DADA2 and deblur' points
  the opposite direction and needs a different table
- Only aggregate or prose-stated numbers exist with no row-level table to re-tabulate
  and rank
- The comparison mixes pipelines measured on different datasets or units, so a per-row
  ranking would be meaningless
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3941
tools:
- name: CDSnake
  role: Pipeline whose total-count preservation claim is being verified; merging-free
    CD-HIT-OTU-Miseq-based approach
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Comparator pipeline run within QIIME2; its total read counts form the baseline
    ranking
- name: deblur
  role: Comparator pipeline; lowest total counts on most benchmarked mock datasets
- name: pandas
  role: Used to extract count columns, rank pipelines per dataset row, and compute
    fold differences
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:55a94aa243b0d7b657a84257c206f035f49b4f5ca29e72a468d254bf88ddb778
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# quantitative-claim-verification-from-tables

## Summary

A verification skill that re-tabulates benchmark numbers from a paper's results table to test a quantitative claim — here, that CDSnake preserves more raw paired-end 16S reads than the merging-based DADA2 and deblur pipelines by ranking total outputted read counts per mock dataset.

## When to use

Use when a paper asserts one pipeline outperforms others on a count-based metric (e.g. total reads retained) and the supporting numbers are available in a benchmark table that can be re-tabulated, ranked, and cross-checked row by row.

## When NOT to use

- The claim concerns OTU counts or annotation correctness rather than total read counts — the paper's finding that 'CDSnake outputted less OTUs than DADA2 and deblur' points the opposite direction and needs a different table
- Only aggregate or prose-stated numbers exist with no row-level table to re-tabulate and rank
- The comparison mixes pipelines measured on different datasets or units, so a per-row ranking would be meaningless

## Inputs

- Benchmark results table (paper Table 1) with total outputted read counts per pipeline (CDSnake, DADA2 via QIIME2, deblur) per mock dataset (Balanced, HMP, Extreme, pooled 'All')
- The claim text to verify (e.g. 'CDSnake preserves more information from raw data than DADA2 and deblur')
- Article text or results section naming the compared pipelines and datasets

## Outputs

- total_read_counts_comparison.csv with columns dataset, pipeline, total_reads, rank, fold_vs_max
- A verdict statement per dataset row confirming or refuting the claimed ranking (including documented exceptions like Extreme)

## How to apply

Load the benchmark table (Table 1 of the CDSnake paper) containing total read counts for CDSnake, DADA2 (QIIME2), and deblur across the Balanced, HMP, Extreme, and pooled 'All' mock dataset rows. Extract the total-count column per pipeline (e.g. the 'All' row reports CDSnake 2,231,154 vs DADA2 1,392,444 vs deblur 224,388 reads), then use pandas to rank the three pipelines per dataset row and compute fold differences relative to the maximum (e.g. CDSnake over deblur). Check the claim's stated invariant — CDSnake highest everywhere — while flagging stated exceptions such as the Extreme dataset where DADA2 slightly exceeded CDSnake. Emit a per-dataset CSV (dataset, pipeline, total_reads, rank, fold_vs_max) so the verdict is auditable against the source numbers.

## Related tools

- **CDSnake** (Pipeline whose total-count preservation claim is being verified; merging-free CD-HIT-OTU-Miseq-based approach) — github.com/ydkondratenko/cdsnake
- **DADA2** (Comparator pipeline run within QIIME2; its total read counts form the baseline ranking)
- **deblur** (Comparator pipeline; lowest total counts on most benchmarked mock datasets)
- **pandas** (Used to extract count columns, rank pipelines per dataset row, and compute fold differences)

## Examples

```
import pandas as pd; d=pd.read_csv('table1.csv'); d['rank']=d.groupby('dataset').total_reads.rank(ascending=False); d['fold_vs_max']=d.total_reads/d.groupby('dataset').total_reads.transform('max'); d.to_csv('total_read_counts_comparison.csv',index=False)
```

## Evaluation signals

- Re-extracted totals for the pooled 'All' row equal exactly 2,231,154 (CDSnake), 1,392,444 (DADA2), and 224,388 (deblur) reads
- Rank column shows CDSnake = 1 on Balanced, HMP, and All rows, with Extreme flagged as the exception where DADA2 ranks first
- fold_vs_max equals 1.0 for the top-ranked pipeline in every row and is computed as pipeline_total / row_max
- Output CSV schema matches dataset, pipeline, total_reads, rank, fold_vs_max with one row per dataset-pipeline pair (4 datasets × 3 pipelines)
- Verdict wording distinguishes 'highest total count on most datasets' from an unconditional claim, matching the Extreme-dataset exception

## Limitations

- The claim holds on most but not all datasets: the Extreme mock community (27 strains at frequencies spanning five orders of magnitude) is the one row where DADA2 slightly exceeded CDSnake in total counts
- Higher total counts indicate less discarded raw data but do not by themselves imply better OTU accuracy — CDSnake outputted fewer OTUs than DADA2 and deblur, though closer to real strain numbers on Balanced and HMP
- Verification is limited to the three mock datasets and their pooled 'All' row; no accession or availability details for the mock datasets are provided, hindering independent reruns

## Evidence

- [abstract] higher total counts on most datasets: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [other] table extraction and ranking method: [evidence span withheld — anchor status: not_found]
- [results] exception dataset characterization: "the Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude"
- [readme] provenance of compared outputs: "OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
