---
name: assembly-statistics-interpretation
description: Use when binned contig FASTA files (MAGs) have been produced by a binning
  step and you must judge per-bin assembly quality before downstream taxonomic or
  functional annotation.
when_to_use_negative:
- Input is raw metagenomic reads (Illumina/Nanopore/PacBio) that have not yet been
  assembled and binned — assembly must precede QC of MAGs
- The goal is taxonomic classification of MAGs rather than quality assessment — use
  the pipeline's taxonomic classification step instead
- Only assembly continuity metrics (N50, total length) are needed with no completeness/contamination
  requirement — plain QUAST without BUSCO suffices
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QUAST
  role: Computes per-bin assembly statistics (contig count, total length, N50) on
    binned contig sets
- name: BUSCO
  role: Estimates MAG completeness and contamination using near-universal single-copy
    orthologs in automated lineage selection mode
- name: nf-core/mag
  role: Pipeline that orchestrates the MAG QC step and aggregates QUAST/BUSCO metrics
  repo: https://github.com/nf-core/mag
- name: MultiQC
  role: Aggregates QUAST and BUSCO outputs into a single QC report
- name: CheckM2
  role: Alternative/complementary estimator of bin completeness and contamination
    using machine learning
  repo: https://github.com/chklovski/CheckM2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:7816311c38cc42ec44cb82f27d997ec71714e86b46e7e9901bfd40208fc36625
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - CheckM2
  - MultiQC
  - QUAST
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# assembly-statistics-interpretation

## Summary

Interprets per-bin assembly statistics (contig count, total length, N50) computed by QUAST alongside BUSCO completeness/contamination to quality-control metagenome-assembled genomes (MAGs) in the nf-core/mag pipeline.

## When to use

Use when binned contig FASTA files (MAGs) have been produced by a binning step and you must judge per-bin assembly quality before downstream taxonomic or functional annotation.

## When NOT to use

- Input is raw metagenomic reads (Illumina/Nanopore/PacBio) that have not yet been assembled and binned — assembly must precede QC of MAGs
- The goal is taxonomic classification of MAGs rather than quality assessment — use the pipeline's taxonomic classification step instead
- Only assembly continuity metrics (N50, total length) are needed with no completeness/contamination requirement — plain QUAST without BUSCO suffices

## Inputs

- Binned contig FASTA files produced by the binning step (one set per MAG)
- BUSCO lineage databases (selected automatically in automated lineage selection mode)

## Outputs

- Per-bin QUAST assembly statistics table (contig count, total length, N50)
- Per-bin BUSCO completeness and contamination estimates
- Aggregated per-bin QC summary table (e.g. mag_qc_summary.tsv) listing bin name, completeness, and contamination
- MultiQC report aggregating QUAST and BUSCO metrics

## How to apply

Load the binned contig FASTA files produced for each MAG and run QUAST on every binned contig set to compute per-bin assembly statistics such as contig count, total length, and N50. In parallel, run BUSCO on each bin in automated lineage selection mode so the best-matching lineage database is chosen to estimate completeness and contamination from near-universal single-copy orthologs. Aggregate the QUAST and BUSCO metrics into a single per-bin summary (e.g. mag_qc_summary.tsv) listing bin name, completeness, and contamination. The rationale is that QUAST captures assembly continuity (N50, length) while BUSCO captures biological completeness, so a bin with high N50 but low completeness, or high contamination, should not be carried forward as a high-quality MAG. Metrics can be collected for MultiQC reporting within the nf-core/mag pipeline.

## Related tools

- **QUAST** (Computes per-bin assembly statistics (contig count, total length, N50) on binned contig sets)
- **BUSCO** (Estimates MAG completeness and contamination using near-universal single-copy orthologs in automated lineage selection mode)
- **nf-core/mag** (Pipeline that orchestrates the MAG QC step and aggregates QUAST/BUSCO metrics) — https://github.com/nf-core/mag
- **MultiQC** (Aggregates QUAST and BUSCO outputs into a single QC report)
- **CheckM2** (Alternative/complementary estimator of bin completeness and contamination using machine learning) — https://github.com/chklovski/CheckM2

## Examples

```
quast.py -o quast_mag bins/*.fasta && busco -i bins/*.fasta --mode auto --out_path busco_mag
```

## Evaluation signals

- mag_qc_summary.tsv exists and contains one row per input bin
- Completeness and contamination fields are present and populated for every bin
- Per-bin QUAST statistics (contig count, total length, N50) are reported for every binned contig set
- BUSCO lineage is auto-selected per bin and corresponds to the best-matching lineage database
- Completeness values fall in the expected 0–100% range and contamination values are non-negative

## Limitations

- As of nf-core/mag 2.1.0, a dedicated bin refinement step was not yet included in the pipeline, so per-bin QC cannot be followed by in-pipeline bin refinement
- BUSCO completeness/contamination estimates depend on lineage database selection; lineages with few high-quality representatives can bias estimates, which motivates ML alternatives like CheckM2
- N50 and total length summarize contiguity only and do not indicate whether the recovered bin is biologically complete or contaminated

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [methods] nf-core/mag estimates MAG completeness and contamination with BUSCO, which uses near-universal single-copy orthologs, while QUAST summarizes QC features of the generated assemblies and MAGs.: [evidence span withheld — anchor status: not_found]
- [readme] CheckM2 has universally trained machine learning models it applies regardless of taxonomic lineage to predict the completeness and contamination of genomic bins.: [evidence span withheld — anchor status: external_doc]
- [discussion] for example, by adding functional annotation as well as assembly and bin refinement steps: "for example, by adding functional annotation as well as assembly and bin refinement steps"
