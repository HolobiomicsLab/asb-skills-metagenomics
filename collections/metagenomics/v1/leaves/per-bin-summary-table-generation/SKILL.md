---
name: per-bin-summary-table-generation
description: Use when you have binned contig FASTA files from a metagenome assembly
  and need per-MAG quality metrics (contig count, N50, completeness, contamination)
  consolidated for bin selection.
when_to_use_negative:
- Input is raw reads or an unbinned assembly — no per-bin QC table can be produced
  before genome binning has generated MAG contig sets.
- The goal is assembly QC of unbinned contigs only — QUAST alone on the primary assembly
  suffices without per-bin aggregation.
- Bin refinement or dereplication across multiple binners is the goal — DAS Tool-style
  integration, not QC summarization, is the appropriate skill.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: nf-core/mag
  role: pipeline that runs the QC workflow and aggregates QUAST and BUSCO metrics
    into the per-bin summary table
  repo: https://github.com/nf-core/mag
- name: QUAST
  role: computes per-bin assembly QC features (contig count, total length, N50) for
    each binned contig set
- name: BUSCO
  role: estimates MAG completeness and contamination using near-universal single-copy
    orthologs in automated lineage selection mode
- name: MultiQC
  role: aggregates tool reports across bins for QC visualization
- name: Nextflow
  role: workflow engine used to execute the nf-core/mag QC steps reproducibly
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:cd347a66d42891458ad113e93544cf5b60a8067793e5fe4762d278999bb9b400
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# per-bin-summary-table-generation

## Summary

Aggregates QUAST assembly statistics and BUSCO completeness/contamination estimates into a single per-bin quality-control table (mag_qc_summary.tsv) for reconstructed MAGs, as implemented in the nf-core/mag pipeline.

## When to use

Use when you have binned contig FASTA files from a metagenome assembly and need per-MAG quality metrics (contig count, N50, completeness, contamination) consolidated for bin selection.

## When NOT to use

- Input is raw reads or an unbinned assembly — no per-bin QC table can be produced before genome binning has generated MAG contig sets.
- The goal is assembly QC of unbinned contigs only — QUAST alone on the primary assembly suffices without per-bin aggregation.
- Bin refinement or dereplication across multiple binners is the goal — DAS Tool-style integration, not QC summarization, is the appropriate skill.

## Inputs

- binned contig FASTA files produced by the binning step (one or more MAG bins)
- QUAST per-bin assembly QC reports (contig count, total length, N50)
- BUSCO per-bin completeness/contamination results in automated lineage selection mode

## Outputs

- mag_qc_summary.tsv — one row per input bin with bin name, QUAST statistics, completeness, and contamination
- per-bin QUAST assembly statistics (contig count, total length, N50)
- per-bin BUSCO completeness and contamination estimates

## How to apply

Load the binned contig FASTA files produced by the binning step for each MAG. Run QUAST on every binned contig set to compute per-bin assembly statistics (contig count, total length, N50). Run BUSCO on each bin in automated lineage selection mode so the best-matching lineage database is chosen to estimate completeness and contamination via near-universal single-copy orthologs. Aggregate the QUAST and BUSCO metrics into a single per-bin summary table listing bin name, completeness, and contamination. Validate that mag_qc_summary.tsv exists, contains one row per input bin, and that completeness and contamination fields are present for every bin.

## Related tools

- **nf-core/mag** (pipeline that runs the QC workflow and aggregates QUAST and BUSCO metrics into the per-bin summary table) — https://github.com/nf-core/mag
- **QUAST** (computes per-bin assembly QC features (contig count, total length, N50) for each binned contig set)
- **BUSCO** (estimates MAG completeness and contamination using near-universal single-copy orthologs in automated lineage selection mode)
- **MultiQC** (aggregates tool reports across bins for QC visualization)
- **Nextflow** (workflow engine used to execute the nf-core/mag QC steps reproducibly) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag -profile docker --input samplesheet.csv --outdir results
```

## Evaluation signals

- mag_qc_summary.tsv exists and is non-empty after the run.
- The summary table contains exactly one row per input bin.
- Completeness and contamination fields are present and numeric for every bin row.
- Per-bin assembly statistics (contig count, total length, N50) from QUAST are populated for each bin.
- BUSCO ran in automated lineage selection mode, so each bin has an assigned lineage database underlying its metrics.

## Limitations

- BUSCO completeness/contamination depend on the selected lineage database; bins from poorly represented lineages may receive unreliable estimates.
- The summary table quality inherits limitations of the upstream binning step; bins fragmented across many contigs yield low N50 and skewed statistics.
- The published nf-core/mag version 2.1.0 did not yet include a dedicated bin refinement step, so QC is assessed per binner output rather than on refined bins.

## Evidence

- [other] QUAST and BUSCO metrics aggregated into a per-bin table: [evidence span withheld — anchor status: not_found]
- [other] BUSCO/QUAST roles in nf-core/mag QC: [evidence span withheld — anchor status: not_found]
- [abstract] MAG reconstruction requires QC from reads: "this requires the reconstruction of metagenome assembled genomes (MAGs) from sequencing reads"
- [readme] bin quality metrics in README context: [evidence span withheld — anchor status: external_doc]
