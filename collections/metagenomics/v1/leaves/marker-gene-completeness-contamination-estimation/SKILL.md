---
name: marker-gene-completeness-contamination-estimation
description: Use when binned contig FASTA files (MAGs) have been produced by a binning
  step and you need per-bin completeness/contamination estimates plus assembly statistics
  to judge bin quality.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) rather than already-binned contig FASTA files
  — this skill operates downstream of assembly and binning.
- A machine-learning alternative is preferred when bins represent lineages with few
  high-quality genomic representatives, e.g. CheckM2's universally trained models
  instead of marker-gene lineage lookup.
- The goal is taxonomic classification of MAGs rather than quantitative completeness/contamination
  assessment of bin quality.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: BUSCO
  role: Estimates MAG completeness and contamination using near-universal single-copy
    orthologs, run per bin in automated lineage selection mode
- name: QUAST
  role: Summarizes QC features (contig count, total length, N50) of the generated
    assemblies and MAGs
- name: nf-core/mag
  role: Pipeline that orchestrates the binning step producing the bins and the subsequent
    MAG quality-control step
  repo: https://github.com/nf-core/mag
- name: MultiQC
  role: Aggregates the QUAST and BUSCO metrics into a single per-bin summary/report
- name: Nextflow
  role: Workflow engine in which the QC steps (QUAST, BUSCO) are executed reproducibly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:b35aa367318909d863fd43a8d8fd92415976841a5bf8973949eedfef900fd9c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - MultiQC
  - Nextflow
  - QUAST
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-gene-completeness-contamination-estimation

## Summary

Quality-controls metagenome-assembled genomes (MAGs) by estimating completeness and contamination from near-universal single-copy marker genes with BUSCO, complemented by QUAST assembly statistics (contig count, total length, N50) for each binned contig set produced by nf-core/mag.

## When to use

Use when binned contig FASTA files (MAGs) have been produced by a binning step and you need per-bin completeness/contamination estimates plus assembly statistics to judge bin quality.

## When NOT to use

- Input is raw sequencing reads (FASTQ) rather than already-binned contig FASTA files — this skill operates downstream of assembly and binning.
- A machine-learning alternative is preferred when bins represent lineages with few high-quality genomic representatives, e.g. CheckM2's universally trained models instead of marker-gene lineage lookup.
- The goal is taxonomic classification of MAGs rather than quantitative completeness/contamination assessment of bin quality.

## Inputs

- binned contig FASTA files (one or more MAGs per sample) produced by the binning step
- BUSCO lineage databases (selected automatically in automated lineage selection mode)

## Outputs

- per-bin QUAST assembly statistics (contig count, total length, N50)
- per-bin BUSCO completeness and contamination estimates
- aggregated per-bin QC summary table (mag_qc_summary.tsv) with bin name, completeness, contamination
- aggregated QC report for MultiQC visualization

## How to apply

Load the binned contig FASTA files produced by the binning step for each MAG in the nf-core/mag pipeline. Run QUAST on every binned contig set to compute per-bin assembly statistics such as contig count, total length and N50. Run BUSCO on each bin in automated lineage selection mode so the best-matching lineage database is chosen, exploiting near-universal single-copy orthologs to estimate completeness and contamination. Aggregate the QUAST and BUSCO metrics into a single per-bin summary table (e.g. mag_qc_summary.tsv) listing bin name, completeness, and contamination, and optionally summarize results with MultiQC. Validate the output by checking that the summary file exists, contains one row per input bin, and that completeness and contamination fields are present for every bin.

## Related tools

- **BUSCO** (Estimates MAG completeness and contamination using near-universal single-copy orthologs, run per bin in automated lineage selection mode)
- **QUAST** (Summarizes QC features (contig count, total length, N50) of the generated assemblies and MAGs)
- **nf-core/mag** (Pipeline that orchestrates the binning step producing the bins and the subsequent MAG quality-control step) — https://github.com/nf-core/mag
- **MultiQC** (Aggregates the QUAST and BUSCO metrics into a single per-bin summary/report)
- **Nextflow** (Workflow engine in which the QC steps (QUAST, BUSCO) are executed reproducibly)

## Examples

```
busco -i bin_1.fasta -m genome --auto-lineage -o bin_1_busco
```

## Evaluation signals

- mag_qc_summary.tsv exists and contains exactly one row per input bin
- completeness and contamination fields are present (non-empty, numeric) for every bin
- QUAST output includes per-bin assembly statistics (contig count, total length, N50) for each binned contig set
- BUSCO was run in automated lineage selection mode and a lineage database was selected for each bin
- QUAST and BUSCO metrics are consistently aggregated into the per-bin summary without orphan bins or duplicate rows

## Limitations

- Marker-gene (BUSCO) completeness/contamination estimation relies on lineage databases; bins from lineages with few reference genomes may be poorly estimated.
- The published nf-core/mag version 2.1.0 lacks a bin refinement step, so QC metrics are computed on bins straight from the binning step without refinement.
- Functional annotation steps were not yet included in the pipeline version described, so QC is limited to completeness/contamination and assembly statistics.

## Evidence

- [other] nf-core/mag estimates MAG completeness and contamination with BUSCO, which uses near-universal single-copy orthologs: [evidence span withheld — anchor status: not_found]
- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [readme] Unlike CheckM1, CheckM2 has universally trained machine learning models it applies regardless of taxonomic lineage to predict the completeness and contamination of genomic bins.: [evidence span withheld — anchor status: external_doc]
- [discussion] for example, by adding functional annotation as well as assembly and bin refinement steps: "for example, by adding functional annotation as well as assembly and bin refinement steps"
