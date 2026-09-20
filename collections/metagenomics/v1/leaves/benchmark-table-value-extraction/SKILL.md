---
name: benchmark-table-value-extraction
description: Reference for extracting per-pipeline OTU counts, total read counts,
  and annotation/correct-annotation values from benchmark tables that compare CD-HIT-OTU-Miseq-based
  CDSnake against QIIME2 DADA2 and deblur on mock communities.
when_to_use_negative:
- The input is raw paired-end FASTQ reads rather than published benchmark values —
  use the CDSnake/DADA2/deblur pipelines themselves instead
- The comparison needs time benchmarking or annotation-quality columns (annotated/correctly
  annotated counts) that the table transcription does not cover without extra care
  to the table's compact numeric encoding
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3307
tools:
- name: CDSnake
  role: Pipeline whose benchmarked OTU counts and total counts are transcribed from
    Table 1
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Comparison pipeline (QIIME2 workflow) whose OTU counts and total counts are
    transcribed from Table 1
- name: deblur
  role: Comparison pipeline (QIIME2 workflow) whose OTU counts and total counts are
    transcribed from Table 1
- name: CD-HIT-OTU-Miseq
  role: Underlying CD-HIT utilities whose separate clustering of paired reads underlies
    the benchmarked CDSnake results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:edc5ea672f2d5825b31a81faf851cc8285137bbed5b468a68f803178b1398e89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# benchmark-table-value-extraction

## Summary

Transcribe benchmark result tables from an article's results section (e.g. Table 1 of the CDSnake paper: OTU counts, annotation statistics, and total counts for CDSnake vs DADA2 vs deblur on the Balanced, HMP, and Extreme mock datasets) into a validated CSV for downstream comparison.

## When to use

Reference for extracting per-pipeline OTU counts, total read counts, and annotation/correct-annotation values from benchmark tables that compare CD-HIT-OTU-Miseq-based CDSnake against QIIME2 DADA2 and deblur on mock communities.

## When NOT to use

- The input is raw paired-end FASTQ reads rather than published benchmark values — use the CDSnake/DADA2/deblur pipelines themselves instead
- The comparison needs time benchmarking or annotation-quality columns (annotated/correctly annotated counts) that the table transcription does not cover without extra care to the table's compact numeric encoding

## Inputs

- Published benchmark table (Table 1) reporting OTU counts, annotation statistics, and total counts for CDSnake, DADA2, and deblur on Balanced, HMP, and Extreme mock datasets
- Article text stating true strain numbers per mock community (Balanced: 57; HMP: 21; Extreme: 27)

## Outputs

- CSV file (e.g. otu_counts_table1.csv) with columns pipeline, dataset, otu_count, total_count, true_strains and 9 data rows
- Validated transcription cross-checked cell-by-cell against the source table

## How to apply

Locate the benchmark table in the results section and transcribe each pipeline-dataset cell: for example, on the Balanced dataset (57 true strains) DADA2 outputted 91 OTUs with 29,344 total counts, deblur 34 OTUs with 200,317 counts, and CDSnake 55 OTUs with 524,604 counts. Append the true number of strains per mock community as a reference column (57 Balanced, 21 HMP, 27 Extreme) so OTU counts can be judged against ground truth — the paper reports CDSnake's counts were closer to real strain numbers on Balanced and HMP. Assemble values into a CSV with columns pipeline, dataset, otu_count, total_count, and true_strains (9 data rows). Cross-check every cell against the printed table for exact numerical agreement before saving.

## Related tools

- **CDSnake** (Pipeline whose benchmarked OTU counts and total counts are transcribed from Table 1) — github.com/ydkondratenko/cdsnake
- **DADA2** (Comparison pipeline (QIIME2 workflow) whose OTU counts and total counts are transcribed from Table 1)
- **deblur** (Comparison pipeline (QIIME2 workflow) whose OTU counts and total counts are transcribed from Table 1)
- **CD-HIT-OTU-Miseq** (Underlying CD-HIT utilities whose separate clustering of paired reads underlies the benchmarked CDSnake results)

## Examples

```
csvstack || echo 'pipeline,dataset,otu_count,total_count,true_strains' > otu_counts_table1.csv && python -c "import csv; rows=list(csv.reader(open('otu_counts_table1.csv'))); assert len(rows)==10"
```

## Evaluation signals

- Row count equals 9 (3 pipelines × 3 datasets) with columns pipeline, dataset, otu_count, total_count, true_strains
- Balanced row values match Table 1 exactly: CDSnake 55 OTUs / 524,604 counts; DADA2 91 OTUs / 29,344 counts; deblur 34 OTUs / 200,317 counts
- true_strains column reads 57 for Balanced, 21 for HMP, 27 for Extreme, matching the article's stated community compositions
- Relative ordering matches reported findings: CDSnake outputted fewer OTUs than DADA2 and deblur but higher total counts
- Every transcribed cell reproduces the printed table value with zero mismatches on a cell-by-cell diff

## Limitations

- Exact strain numbers must be taken from the article text (e.g. Balanced: 57 bacteria and archaea); conflicting claims in derived summaries (e.g. 59 actual strains) should be resolved against the results section
- No accession or availability details are given for the three mock community datasets, so the table cannot be independently regenerated from raw data via this skill alone
- Software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated, limiting reproducibility of the benchmark being transcribed

## Evidence

- [results] DADA2 outputted 91 OTUs on the Balanced dataset (actual strains: 59), with 57 annotated and 39 correctly annotated: [evidence span withheld — anchor status: not_found]
- [abstract] CDSnake outputted less OTUs than DADA2 and deblur.: "CDSnake outputted less OTUs than DADA2 and deblur."
- [results] The Balanced community contained 57 bacteria and archaea at nominally equal frequencies: "The Balanced community contained 57 bacteria and archaea at nominally equal frequencies"
- [readme] OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder.: "OTU table for all samples will be saved in file “OTU.txt” in “pooled” folder."
