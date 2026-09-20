---
name: catbat-bin-taxonomic-classification
description: Use when assigning taxonomy to MAG FASTA bins, especially bins that fail
  BUSCO-derived completeness/contamination quality thresholds and therefore cannot
  be processed with GTDB-Tk's marker-gene-based classification.
when_to_use_negative:
- The bin has already passed the BUSCO quality thresholds and contains sufficient
  marker genes — route it to GTDB-Tk instead of CAT/BAT.
- The goal is metagenome assembly or binning itself rather than taxonomic annotation
  of existing MAGs — nf-core/mag stages assembly and binning separately before taxonomic
  classification.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: CAT/BAT
  role: Taxonomically classifies any MAG via ORF-level assignment; applied to bins
    that do not pass the BUSCO quality gating for GTDB-Tk
- name: GTDB-Tk
  role: GTDB lineage assignment via classify_wf, run only on quality-passing bins
    that contain the marker genes it requires
- name: BUSCO
  role: Provides the completeness/contamination estimates used to filter which bins
    are routed to GTDB-Tk versus CAT/BAT
- name: nf-core/mag
  role: Pipeline that orchestrates the BUSCO-gated routing, tool execution, and merging
    into mag_taxonomy.tsv
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine in which nf-core/mag and this classification stage are implemented
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:0a8ed58e50af7e73e26a4c035788f573351ba8799f9df042a0e58d7554568566
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/mag
  tools:
  - BUSCO
  - CAT/BAT
  - GTDB-Tk
  - Nextflow
  - nf-core/mag
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# CAT/BAT bin taxonomic classification

## Summary

Classifies metagenome-assembled genomes (MAGs) taxonomically using CAT/BAT's ORF-level assignment, serving as the fallback for bins that lack the marker genes required by GTDB-Tk in nf-core/mag's final taxonomic classification stage.

## When to use

Use when assigning taxonomy to MAG FASTA bins, especially bins that fail BUSCO-derived completeness/contamination quality thresholds and therefore cannot be processed with GTDB-Tk's marker-gene-based classification.

## When NOT to use

- The bin has already passed the BUSCO quality thresholds and contains sufficient marker genes — route it to GTDB-Tk instead of CAT/BAT.
- The goal is metagenome assembly or binning itself rather than taxonomic annotation of existing MAGs — nf-core/mag stages assembly and binning separately before taxonomic classification.

## Inputs

- MAG FASTA bin files from the upstream binning step of nf-core/mag
- BUSCO quality summaries (completeness/contamination estimates) for each bin
- GTDB-Tk and CAT/BAT taxonomic results for the respective bin subsets

## Outputs

- mag_taxonomy.tsv: one row per input MAG with taxonomy and method (GTDB-Tk vs CAT/BAT) columns populated

## How to apply

In nf-core/mag's taxonomic annotation stage, load the MAG FASTA bin files together with their BUSCO quality summaries from the upstream binning step. Filter bins with the BUSCO-derived quality thresholds so only quality-passing bins (those containing the marker genes GTDB-Tk needs) are routed to GTDB-Tk classify_wf; route all remaining bins to CAT/BAT, which classifies them via ORF-level assignment and does not depend on marker genes. Keep the read-type gating in mind since nf-core/mag processes short-read, long-read, and hybrid data and not every tool applies to every read type. Merge the GTDB-Tk and CAT/BAT results into one row per MAG and write the combined table as mag_taxonomy.tsv. The rationale, per the source card, is that CAT/BAT can taxonomically classify any MAG, whereas GTDB-TK requires marker genes and is therefore only applied to MAGs passing BUSCO-estimated completeness and contamination quality thresholds.

## Related tools

- **CAT/BAT** (Taxonomically classifies any MAG via ORF-level assignment; applied to bins that do not pass the BUSCO quality gating for GTDB-Tk)
- **GTDB-Tk** (GTDB lineage assignment via classify_wf, run only on quality-passing bins that contain the marker genes it requires)
- **BUSCO** (Provides the completeness/contamination estimates used to filter which bins are routed to GTDB-Tk versus CAT/BAT)
- **nf-core/mag** (Pipeline that orchestrates the BUSCO-gated routing, tool execution, and merging into mag_taxonomy.tsv) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine in which nf-core/mag and this classification stage are implemented) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --gtdb /path/to/gtdb --catbat_db /path/to/catbat_db --outdir results -profile docker
```

## Evaluation signals

- mag_taxonomy.tsv exists after the run and contains exactly one row per input MAG.
- Both the taxonomy column and the method (GTDB-Tk vs CAT/BAT) column are populated for every row.
- GTDB-Tk assignments appear only for bins that passed the BUSCO-derived completeness/contamination thresholds.
- Every bin receives a taxonomy from exactly one of the two tools (no bin is double-assigned or missing).
- Routing respects the read-type gating: tools not applicable to a given read type (short-read, long-read, hybrid) are not invoked for that data.

## Limitations

- GTDB-Tk cannot classify bins lacking the marker genes it requires, so low-completeness or high-contamination bins that fail BUSCO thresholds must fall back to CAT/BAT.
- Not every annotation tool applies to every read type (short-read, long-read, hybrid) in nf-core/mag, so tool availability depends on the input data type.
- Functional annotation, as well as assembly and bin refinement steps, were not yet included in the presented version 2.1.0 of nf-core/mag, limiting what this stage can annotate beyond taxonomy.

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [results] In nf-core/mag's taxonomic annotation stage, CAT/BAT can taxonomically classify any MAG, whereas GTDB-TK requires marker genes and is therefore only applied to MAGs passing BUSCO-estimated completeness and contamination quality thresholds.: [evidence span withheld — anchor status: not_found]
- [intro] as well as taxonomic and functional annotation of MAGs: "as well as taxonomic and functional annotation of MAGs"
- [discussion] for example, by adding functional annotation as well as assembly and bin refinement steps: "for example, by adding functional annotation as well as assembly and bin refinement steps"
