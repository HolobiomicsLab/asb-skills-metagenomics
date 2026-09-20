---
name: 16s-amplicon-pipeline-evaluation
description: Use when you need to decide which 16S OTU-retrieval pipeline preserves
  raw paired-end read information, or when benchmarking merging-free (CD-HIT-OTU-Miseq/CDSnake)
  against merging-based pipelines (DADA2, deblur) on mock community datasets.
when_to_use_negative:
- Your input is already a processed OTU/feature table — this skill evaluates raw paired-end
  FASTQ-to-OTU pipelines, not downstream table analysis.
- Your reads are single-end or non-overlapping paired-end data, where the merging-free
  CD-HIT-OTU-Miseq rationale (separate clustering of R1/R2) and the 2 × 250 bp overlapping-read
  benchmarking design do not apply.
- You need to benchmark ASV inference on real (non-mock) communities with no known
  strain ground truth for the OTU-count-vs-reality comparison.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CDSnake
  role: Merging-free pipeline under evaluation; runs CD-HIT-OTU-Miseq tools consecutively
    via Snakemake to produce the OTU table (OTU.txt in the pooled folder)
  repo: https://github.com/ydkondratenko/cdsnake
- name: CD-HIT-OTU-Miseq
  role: Core utilities evaluated for avoiding the reads-merging process via separate
    clustering of paired reads
- name: QIIME2
  role: Workflow framework hosting the DADA2 and deblur comparison pipelines
- name: DADA2
  role: Merging-based 16S pipeline used as benchmarking comparator for total counts,
    OTU counts, and annotation accuracy
- name: deblur
  role: Merging-based 16S pipeline used as benchmarking comparator
- name: Trimmomatic
  role: Quality filtering step within CDSnake; crops R1/R2 to 'good' quality parts
    (default 200/180 bp) and discards reads shorter than MINLEN 100
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:6999b7477b887e9e50935df8cd51b5924d9a7643d89cdd55b61f5e844b0ef380
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ydkondratenko/cdsnake
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - QIIME2
  - Trimmomatic
  - deblur
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# 16s-amplicon-pipeline-evaluation

## Summary

Benchmarking method for comparing 16S rRNA amplicon OTU-retrieval pipelines (CDSnake/CD-HIT-OTU-Miseq vs QIIME2-DADA2 vs deblur) on mock community paired-end 2 × 250 bp datasets, using total outputted read counts, OTU counts, and annotation accuracy as evaluation metrics.

## When to use

Use when you need to decide which 16S OTU-retrieval pipeline best preserves raw paired-end read information, or when benchmarking merging-free (CD-HIT-OTU-Miseq/CDSnake) against merging-based pipelines (DADA2, deblur) on mock community datasets.

## When NOT to use

- Your input is already a processed OTU/feature table — this skill evaluates raw paired-end FASTQ-to-OTU pipelines, not downstream table analysis.
- Your reads are single-end or non-overlapping paired-end data, where the merging-free CD-HIT-OTU-Miseq rationale (separate clustering of R1/R2) and the 2 × 250 bp overlapping-read benchmarking design do not apply.
- You need to benchmark ASV inference on real (non-mock) communities with no known strain ground truth for the OTU-count-vs-reality comparison.

## Inputs

- Paired-end 16S FASTQ reads from mock community datasets (Balanced, HMP, Extreme, and pooled 'All')
- Annotated 16S reference database (reformatted Greengenes or SILVA fasta)
- Table of known strain counts per mock community (e.g. 57 for Balanced, 21 for HMP, 27 for Extreme)
- Benchmarking output tables with total read counts, OTU counts, annotation counts, and run times per pipeline

## Outputs

- Per-dataset comparison table (CSV, e.g. total_read_counts_comparison.csv with columns dataset, pipeline, total_reads, rank, fold_vs_max)
- OTU count vs true-strain-count comparison per dataset per pipeline, including annotated and correctly annotated OTU tallies
- Time benchmarking table (seconds per pipeline per dataset)

## How to apply

Run each pipeline on the same paired-end 16S FASTQ inputs from mock communities (Balanced: 57 strains; HMP: 21 strains; Extreme: 27 strains spanning five orders of magnitude in frequency) and pool them into an 'All' row. Extract per-dataset total read counts for CDSnake, DADA2 (QIIME2), and deblur, then rank pipelines by count and compute fold differences (e.g. All row: CDSnake 2,231,154 vs DADA2 1,392,444 vs deblur 224,388) to verify the merging-free approach discards fewer reads. As a second axis, compare OTU counts per pipeline against the known number of mock community strains, and record how many OTUs are annotated and correctly annotated (e.g. DADA2 gave 91 OTUs on Balanced vs 59 real strains, with 57 annotated and 39 correctly annotated). Also capture wall-clock time per pipeline per dataset. Write the comparison table (columns: dataset, pipeline, total_reads, rank, fold_vs_max) to a CSV such as total_read_counts_comparison.csv.

## Related tools

- **CDSnake** (Merging-free pipeline under evaluation; runs CD-HIT-OTU-Miseq tools consecutively via Snakemake to produce the OTU table (OTU.txt in the pooled folder)) — https://github.com/ydkondratenko/cdsnake
- **CD-HIT-OTU-Miseq** (Core utilities evaluated for avoiding the reads-merging process via separate clustering of paired reads)
- **QIIME2** (Workflow framework hosting the DADA2 and deblur comparison pipelines)
- **DADA2** (Merging-based 16S pipeline used as benchmarking comparator for total counts, OTU counts, and annotation accuracy)
- **deblur** (Merging-based 16S pipeline used as benchmarking comparator)
- **Trimmomatic** (Quality filtering step within CDSnake; crops R1/R2 to 'good' quality parts (default 200/180 bp) and discards reads shorter than MINLEN 100)

## Examples

```
cd input_folder && snakemake
```

## Evaluation signals

- CDSnake should yield the highest total read count on most datasets — verified by the All row: CDSnake 2,231,154 vs DADA2 1,392,444 and deblur 224,388.
- Extreme is the known exception where DADA2 slightly exceeds CDSnake in total counts; any other inversion flags a benchmarking error.
- On Balanced and HMP, CDSnake's OTU count should be closer to the true strain count (59 and 21 respectively) than DADA2's (91 OTUs on Balanced) or deblur's (34 OTUs on Balanced).
- Each pipeline row must have complete totals, rank, and fold_vs_max values in the CSV; ranks should be 1–3 with no ties except the documented Extreme case.
- Time benchmarking should show CDSnake generally slower than DADA2 and deblur, consistent with the article's findings.

## Limitations

- CDSnake is generally slower than DADA2 and deblur, so time-to-result is traded for higher read retention.
- The Extreme dataset is an exception to the count-preservation advantage — DADA2 slightly exceeded CDSnake there.
- The hardware/execution environment for the time benchmarking is not specified, and software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated.
- No accessions or availability details are given for the three mock community datasets, limiting reproducibility.
- CDSnake outputted fewer OTUs than DADA2 and deblur, so pipeline choice affects downstream OTU-table granularity.

## Evidence

- [results] CDSnake yielded the highest total counts on most benchmarked mock datasets: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [intro] Merging-free rationale: "Recently CD-HIT-OTU-Miseq [3] was presented as a new approach, entirely avoiding reads merging due to separate clustering of paired reads"
- [results] Mock dataset design: "the Extreme community contained 27 bacterial strains at frequencies spanning five orders of magnitude"
- [methods] Cropping and MINLEN parameters: "Default lengths of “good” parts for R1 and R2 are set to 200 and 180 bp correspondingly."
- [readme] Pipeline invocation via Snakemake: "To run pipeline with default Snakemake settings, use command snakemake."
