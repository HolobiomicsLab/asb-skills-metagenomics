---
name: busco-quality-threshold-filtering
description: Decision support for routing metagenome-assembled genome (MAG) FASTA
  bins between GTDB-Tk and CAT/BAT during the taxonomic annotation stage of nf-core/mag,
  using BUSCO quality summaries as the gate.
when_to_use_negative:
- Input bins have no BUSCO quality summary available — the routing gate cannot be
  evaluated without upstream completeness/contamination estimates.
- The goal is taxonomic classification of contigs or reads rather than assembled MAG
  bins (CAT/BAT and GTDB-Tk here operate on binned genome FASTAs).
- The pipeline run is assembly/binning-only; nf-core/mag's functional annotation and
  bin refinement steps are noted as not yet included, so this skill only concerns
  the taxonomic annotation stage.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: nf-core/mag
  role: Host pipeline that runs the taxonomic classification stage and applies the
    BUSCO-based routing between GTDB-Tk and CAT/BAT
  repo: https://github.com/nf-core/mag
- name: GTDB-Tk
  role: Assigns GTDB lineages via classify_wf to MAGs passing the BUSCO completeness/contamination
    thresholds (requires marker genes)
- name: CAT/BAT
  role: Taxonomically classifies all remaining bins via ORF-level assignment as a
    fallback for bins excluded from GTDB-Tk
- name: BUSCO
  role: Provides the marker-gene-based completeness and contamination estimates used
    as the quality gate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:a15433824aa77ed91687df1046ba176adc229af9179985ef69ffda40817b0ba1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - CAT/BAT
  - GTDB-Tk
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# busco-quality-threshold-filtering

## Summary

A routing step in the nf-core/mag pipeline that gates MAGs by BUSCO-derived completeness and contamination thresholds so that only marker-gene-containing, quality-passing bins go to GTDB-Tk, while all remaining bins are classified with CAT/BAT and both results are merged into mag_taxonomy.tsv.

## When to use

Decision support for routing metagenome-assembled genome (MAG) FASTA bins between GTDB-Tk and CAT/BAT during the taxonomic annotation stage of nf-core/mag, using BUSCO quality summaries as the gate.

## When NOT to use

- Input bins have no BUSCO quality summary available — the routing gate cannot be evaluated without upstream completeness/contamination estimates.
- The goal is taxonomic classification of contigs or reads rather than assembled MAG bins (CAT/BAT and GTDB-Tk here operate on binned genome FASTAs).
- The pipeline run is assembly/binning-only; nf-core/mag's functional annotation and bin refinement steps are noted as not yet included, so this skill only concerns the taxonomic annotation stage.

## Inputs

- MAG FASTA bin files from the binning step of nf-core/mag
- BUSCO quality summaries (completeness and contamination estimates) for each bin
- Read-type configuration (short-read, long-read, or hybrid)

## Outputs

- mag_taxonomy.tsv with one row per MAG, populated taxonomy and method (GTDB-Tk vs CAT/BAT) columns
- GTDB-Tk classify_wf lineage assignments for quality-passing bins
- CAT/BAT taxonomic classifications for the remaining bins

## How to apply

Load the MAG FASTA bin files together with the BUSCO quality summaries produced by the upstream binning/QC step of nf-core/mag. Apply the BUSCO-derived completeness and contamination cut-offs so that only bins containing sufficient marker genes are routed to GTDB-Tk (GTDB-Tk requires marker genes, so it cannot classify every MAG). Run GTDB-Tk classify_wf on the quality-passing bins, then run CAT/BAT on all remaining bins, since CAT/BAT can taxonomically classify any MAG via ORF-level assignment. Finally merge the GTDB-Tk and CAT/BAT results into one row per MAG and write the combined table as mag_taxonomy.tsv, keeping in mind that nf-core/mag processes short-read, long-read, and hybrid data so not every tool applies to every read type.

## Related tools

- **nf-core/mag** (Host pipeline that runs the taxonomic classification stage and applies the BUSCO-based routing between GTDB-Tk and CAT/BAT) — https://github.com/nf-core/mag
- **GTDB-Tk** (Assigns GTDB lineages via classify_wf to MAGs passing the BUSCO completeness/contamination thresholds (requires marker genes))
- **CAT/BAT** (Taxonomically classifies all remaining bins via ORF-level assignment as a fallback for bins excluded from GTDB-Tk)
- **BUSCO** (Provides the marker-gene-based completeness and contamination estimates used as the quality gate)

## Examples

```
gtdbtk classify_wf --genome_dir mag_bins_qc_pass --out_dir gtdbtk_out --extension fa
```

## Evaluation signals

- mag_taxonomy.tsv exists and contains one row per input MAG.
- Taxonomy and method (GTDB-Tk vs CAT/BAT) columns are populated for every MAG in mag_taxonomy.tsv.
- GTDB-Tk assignments appear only for bins that passed the BUSCO completeness/contamination thresholds; all other MAGs are annotated as CAT/BAT-classified.
- Every input MAG appears exactly once in the merged table (no duplicate rows from the two classification paths).

## Limitations

- GTDB-Tk requires marker genes and therefore cannot classify every MAG; bins without sufficient marker content fall back to CAT/BAT.
- Not every tool applies to every read type — nf-core/mag processes short-read, long-read, and hybrid data, so the gating must account for the read-type configuration.
- The published nf-core/mag version (2.1.0) does not include functional annotation or assembly/bin refinement steps, so quality gating relies on the upstream BUSCO estimates as-is.

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [abstract] The pipeline is easy to install, portable and reproducible, with dependencies provided within containers: "The pipeline is easy to install-all dependencies are provided within containers-portable and reproducible."
- [intro] as well as taxonomic and functional annotation of MAGs: "as well as taxonomic and functional annotation of MAGs"
- [readme] CheckM2 predicts completeness and contamination of genomic bins regardless of taxonomic lineage (readme context on bin quality estimation): "to predict the completeness and contamination of genomic bins"
