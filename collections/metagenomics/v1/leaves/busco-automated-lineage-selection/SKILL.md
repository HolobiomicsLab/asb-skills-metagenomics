---
name: busco-automated-lineage-selection
description: Use when binned contig FASTA files (MAGs) from metagenome binning need
  per-bin completeness and contamination estimates and assembly QC metrics (contig
  count, total length, N50).
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) rather than binned contig FASTA files — the
  reads must first go through assembly and binning.
- The goal is de novo metagenome assembly itself, not assessment of the quality of
  already-reconstructed MAGs.
- The bins are viral/plasmid or otherwise lack the near-universal single-copy orthologs
  BUSCO relies on; an ortholog-based completeness estimate is not meaningful in that
  regime.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: BUSCO
  role: Estimates MAG completeness and contamination using near-universal single-copy
    orthologs, run per bin in automated lineage selection mode
- name: QUAST
  role: Summarizes QC features of the generated assemblies and MAGs (contig count,
    total length, N50) per bin
- name: nf-core/mag
  role: Pipeline orchestrating the bin-QC step, producing the aggregated per-bin summary
    (mag_qc_summary.tsv)
  repo: https://github.com/nf-core/mag
- name: MultiQC
  role: Aggregates QUAST and BUSCO QC metrics into a combined report
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:bd36d5ccca611d1f37779b373f6cae2e440b01bb07e790630aa6904ab5f9b690
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - MultiQC
  - QUAST
  - nf-core/mag
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# busco-automated-lineage-selection

## Summary

Quality-control step of the nf-core/mag pipeline in which binned metagenome-assembled genomes (MAGs) are assessed with BUSCO's automated lineage selection to estimate completeness and contamination, complemented by QUAST assembly statistics.

## When to use

Use when binned contig FASTA files (MAGs) from metagenome binning need per-bin completeness and contamination estimates and assembly QC metrics (contig count, total length, N50).

## When NOT to use

- Input is raw sequencing reads (FASTQ) rather than binned contig FASTA files — the reads must first go through assembly and binning.
- The goal is de novo metagenome assembly itself, not assessment of the quality of already-reconstructed MAGs.
- The bins are viral/plasmid or otherwise lack the near-universal single-copy orthologs BUSCO relies on; an ortholog-based completeness estimate is not meaningful in that regime.

## Inputs

- Binned contig FASTA files (one set per MAG) from the genome binning step
- BUSCO lineage databases (auto-selected in automated lineage mode)
- Assembly FASTA files of the generated assemblies

## Outputs

- mag_qc_summary.tsv: per-bin table with bin name, completeness, and contamination
- QUAST per-bin assembly statistics (contig count, total length, N50)
- BUSCO completeness/contamination estimates per bin
- MultiQC aggregated QC report

## How to apply

Load the binned contig FASTA files produced by the binning step (e.g. MetaBAT2/MaxBin2/CONCOCT output within nf-core/mag) for each MAG. Run QUAST on every binned contig set to compute per-bin assembly statistics such as contig count, total length, and N50. Run BUSCO on each bin in automated lineage selection mode so the best-matching lineage database is chosen, exploiting near-universal single-copy orthologs to estimate completeness and contamination. Aggregate QUAST and BUSCO metrics into a single per-bin summary table listing bin name, completeness, and contamination, with results surfaced via MultiQC. Rationale: BUSCO's ortholog-based approach is the standard used by nf-core/mag (v2.1.0) for bin QC, and the pipeline aggregates all QC reporting through MultiQC.

## Related tools

- **BUSCO** (Estimates MAG completeness and contamination using near-universal single-copy orthologs, run per bin in automated lineage selection mode)
- **QUAST** (Summarizes QC features of the generated assemblies and MAGs (contig count, total length, N50) per bin)
- **nf-core/mag** (Pipeline orchestrating the bin-QC step, producing the aggregated per-bin summary (mag_qc_summary.tsv)) — https://github.com/nf-core/mag
- **MultiQC** (Aggregates QUAST and BUSCO QC metrics into a combined report)

## Examples

```
busco -i mag_bin1.fasta -o busco_mag_bin1 -m genome --auto-lineage
```

## Evaluation signals

- mag_qc_summary.tsv exists and contains one row per input bin.
- Every row of mag_qc_summary.tsv has non-empty completeness and contamination fields.
- BUSCO automated lineage selection chose a lineage database for each bin (no bins left without a lineage).
- Completeness and contamination values fall within plausible 0–100% ranges for each bin.
- QUAST per-bin statistics (contig count, total length, N50) are present and consistent with the input bin FASTA sizes.

## Limitations

- The accuracy of completeness/contamination estimates depends on BUSCO choosing a suitable lineage database via automated lineage selection; distant or atypical lineages can be mis-estimated.
- nf-core/mag v2.1.0 did not yet include a bin refinement step, so no refined/chimeric-bin correction precedes the QC step.
- ML-based alternatives such as CheckM2 use universally trained models that apply regardless of taxonomic lineage, which can behave differently on lineages with few high-quality genomic representatives.

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs): "A typical reference-independent metagenomic workflow consists of preprocessing raw reads, assembly and binning to generate so-called metagenome assembled genomes (MAGs)"
- [readme] a dashboard to display multiple metagenome quality metrics and taxonomical annotation from several tools: [evidence span withheld — anchor status: external_doc]
