---
name: metagenome-binning-quality-assessment
description: Use when binning has produced MAG FASTA bins and you need BUSCO- or CheckM2-estimated
  completeness/contamination scores to gate which bins go to GTDB-Tk versus CAT/BAT,
  or to report MAG quality.
when_to_use_negative:
- Input is raw metagenomic reads or assemblies that have not yet been binned into
  MAG FASTA bins — quality assessment needs discrete genome bins
- The goal is taxonomic classification independent of quality gating — CAT/BAT can
  classify any MAG without BUSCO thresholds, so no quality step is needed first
- Analyzing metatranscriptomic or functional-annotation workflows — the paper notes
  functional annotation is a step 'not yet included' in nf-core/mag 2.1.0
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: nf-core/mag
  role: Nextflow pipeline whose binning and taxonomic-classification stages produce
    the MAG bins and the BUSCO-gated GTDB-Tk/CAT-BAT routing assessed here
  repo: https://github.com/nf-core/mag
- name: BUSCO
  role: marker-gene-based estimator of MAG completeness and contamination whose thresholds
    gate which bins are eligible for GTDB-Tk
- name: GTDB-Tk
  role: applied via classify_wf only to bins passing BUSCO quality thresholds because
    it requires marker genes
- name: CAT/BAT
  role: fallback taxonomic classifier applied to all remaining bins via ORF-level
    assignment, since it can classify any MAG
- name: CheckM2
  role: machine-learning alternative for predicting completeness and contamination
    of genome bins independent of taxonomic lineage
  repo: chklovski/CheckM2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:5cdf5ccd702a77ec533bd8687a5f9dcecacb8fba34c67f505c43dec5fbc46831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - CAT/BAT
  - CheckM2
  - GTDB-Tk
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metagenome-binning-quality-assessment

## Summary

Assesses the quality (completeness and contamination) of metagenome-assembled genomes (MAGs) produced by the binning step of nf-core/mag using marker-gene-based estimators (BUSCO, CheckM2), and uses these estimates to gate downstream taxonomic classification with GTDB-Tk versus CAT/BAT.

## When to use

Use when binning has produced MAG FASTA bins and you need BUSCO- or CheckM2-estimated completeness/contamination scores to gate which bins go to GTDB-Tk versus CAT/BAT, or to report MAG quality.

## When NOT to use

- Input is raw metagenomic reads or assemblies that have not yet been binned into MAG FASTA bins — quality assessment needs discrete genome bins
- The goal is taxonomic classification independent of quality gating — CAT/BAT can classify any MAG without BUSCO thresholds, so no quality step is needed first
- Analyzing metatranscriptomic or functional-annotation workflows — the paper notes functional annotation is a step 'not yet included' in nf-core/mag 2.1.0

## Inputs

- MAG FASTA bin files produced by the binning step of nf-core/mag
- BUSCO quality summaries (completeness and contamination estimates) from upstream binning
- Quality thresholds (completeness/contamination cut-offs) configured for the pipeline run

## Outputs

- Per-MAG quality metrics (completeness, contamination) from BUSCO
- A quality-gated split of bins: GTDB-Tk-eligible subset vs CAT/BAT fallback subset
- Merged mag_taxonomy.tsv with one row per MAG including taxonomy and classification-method columns

## How to apply

Load the MAG FASTA bin files together with the upstream binning outputs of nf-core/mag (Nextflow DSL2 pipeline for metagenome assembly, binning and taxonomic classification). Run marker-gene-based quality estimation (BUSCO) on every bin to obtain completeness and contamination estimates, mindful of read-type gating (short-read, long-read, hybrid). Apply the BUSCO-derived completeness/contamination cut-offs as a filter step: only bins passing thresholds carry the marker genes required by GTDB-Tk, so they are routed to GTDB-Tk classify_wf; all remaining bins fall back to CAT/BAT, which classifies any MAG via ORF-level assignment. Validate that the merged mag_taxonomy.tsv contains one row per input MAG with taxonomy and method (GTDB-Tk vs CAT/BAT) columns populated, and that GTDB-Tk assignments appear only for bins that passed the BUSCO thresholds.

## Related tools

- **nf-core/mag** (Nextflow pipeline whose binning and taxonomic-classification stages produce the MAG bins and the BUSCO-gated GTDB-Tk/CAT-BAT routing assessed here) — https://github.com/nf-core/mag
- **BUSCO** (marker-gene-based estimator of MAG completeness and contamination whose thresholds gate which bins are eligible for GTDB-Tk)
- **GTDB-Tk** (applied via classify_wf only to bins passing BUSCO quality thresholds because it requires marker genes)
- **CAT/BAT** (fallback taxonomic classifier applied to all remaining bins via ORF-level assignment, since it can classify any MAG)
- **CheckM2** (machine-learning alternative for predicting completeness and contamination of genome bins independent of taxonomic lineage) — chklovski/CheckM2

## Evaluation signals

- mag_taxonomy.tsv exists and contains one row per input MAG with taxonomy and method (GTDB-Tk vs CAT/BAT) columns populated
- GTDB-Tk assignments appear only for bins that passed the BUSCO completeness/contamination thresholds; all other rows are attributed to CAT/BAT
- The set of GTDB-Tk-eligible bins equals the set of bins meeting the configured BUSCO quality cut-offs (no bin is routed to both tools)
- Every input MAG is represented in the output table (no bin lost during the merge of GTDB-Tk and CAT/BAT results)
- Reported completeness/contamination values fall in valid ranges (0-100%) and are consistent with the BUSCO summary files

## Limitations

- GTDB-Tk requires marker genes and therefore cannot classify bins failing BUSCO quality thresholds — those bins must fall back to CAT/BAT
- nf-core/mag processes short-read, long-read, and hybrid data, and not every tool applies to every read type, so quality assessment must respect read-type gating
- Functional annotation and bin refinement steps are not yet included in the described pipeline version, so quality assessment covers taxonomy routing only

## Evidence

- [abstract] nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] nf-core/mag as Nextflow pipeline for hybrid assembly and MAG classification: "Here we introduce nf-core/mag, a Nextflow pipeline for hybrid assembly of metagenomes, binning and taxonomic classification of MAGs"
- [readme] CheckM2 predicts completeness and contamination of genomic bins: [evidence span withheld — anchor status: external_doc]
- [discussion] co-abundances used for genome binning step: "it can utilize sample-wise group information to perform co-assembly and/or to compute co-abundances used for the genome binning step"
