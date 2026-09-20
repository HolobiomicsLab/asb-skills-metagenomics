---
name: mag-taxonomic-classification
description: Use when assigning taxonomy to metagenome-assembled genomes (MAGs) produced
  by nf-core/mag or similar binning workflows, especially when bins have heterogeneous
  quality and a BUSCO-gated GTDB-Tk plus CAT/BAT fallback is appropriate.
when_to_use_negative:
- Input is raw reads or unassembled contigs rather than reconstructed MAG bins — this
  skill covers the final taxonomic annotation stage only
- All bins already have reference genome assignments and do not require marker-gene-based
  or ORF-based taxonomic classification
- Bins lack BUSCO completeness/contamination summaries, making the quality gating
  that determines GTDB-Tk eligibility impossible
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: nf-core/mag
  role: Host pipeline that orchestrates the taxonomic classification stage and read-type
    gating
  repo: https://github.com/nf-core/mag
- name: GTDB-Tk
  role: Classifies quality-passing (BUSCO-gated) MAGs into GTDB lineages via classify_wf
- name: CAT/BAT
  role: Fallback taxonomic classifier applied to bins that do not pass BUSCO thresholds,
    using ORF-level assignment
- name: BUSCO
  role: Provides the completeness/contamination estimates used as quality thresholds
    gating GTDB-Tk application
- name: Nextflow
  role: Workflow engine in which the nf-core/mag classification stage is implemented
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:8f509f89709406899e5b9cbca5acdbca32f1a980ade292dc1dd3330e784121cc
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# mag-taxonomic-classification

## Summary

Reconstructs the final taxonomic classification stage of the nf-core/mag pipeline, in which MAGs passing BUSCO-based quality thresholds are classified with GTDB-Tk while remaining bins fall back to CAT/BAT, merging both results into a single per-bin taxonomy table.

## When to use

Use when assigning taxonomy to metagenome-assembled genomes (MAGs) produced by nf-core/mag or similar binning workflows, especially when bins have heterogeneous quality and a BUSCO-gated GTDB-Tk plus CAT/BAT fallback is appropriate.

## When NOT to use

- Input is raw reads or unassembled contigs rather than reconstructed MAG bins — this skill covers the final taxonomic annotation stage only
- All bins already have reference genome assignments and do not require marker-gene-based or ORF-based taxonomic classification
- Bins lack BUSCO completeness/contamination summaries, making the quality gating that determines GTDB-Tk eligibility impossible

## Inputs

- MAG FASTA bin files from the binning step of nf-core/mag
- BUSCO quality summaries (completeness/contamination estimates) per bin
- Read-type configuration (short-read, long-read, or hybrid) from the nf-core/mag run

## Outputs

- mag_taxonomy.tsv with one row per input MAG, populated taxonomy column, and a method column (GTDB-Tk vs CAT/BAT)

## How to apply

Load the MAG FASTA bin files together with the BUSCO quality summaries produced by the upstream binning step of nf-core/mag. Apply the BUSCO-derived completeness and contamination thresholds to gate which bins are routed to GTDB-Tk, since GTDB-Tk classify_wf requires marker genes and therefore only works on quality-passing bins; note nf-core/mag handles short-read, long-read, and hybrid data, and tool applicability varies by read type. Run GTDB-Tk classify_wf on the quality-passing bins to obtain GTDB lineage assignments, then run CAT/BAT on all remaining bins to classify them via ORF-level assignment, as CAT/BAT can taxonomically classify any MAG. Merge GTDB-Tk and CAT/BAT results into one row per MAG, recording which method produced each assignment, and write the combined table as mag_taxonomy.tsv.

## Related tools

- **nf-core/mag** (Host pipeline that orchestrates the taxonomic classification stage and read-type gating) — https://github.com/nf-core/mag
- **GTDB-Tk** (Classifies quality-passing (BUSCO-gated) MAGs into GTDB lineages via classify_wf)
- **CAT/BAT** (Fallback taxonomic classifier applied to bins that do not pass BUSCO thresholds, using ORF-level assignment)
- **BUSCO** (Provides the completeness/contamination estimates used as quality thresholds gating GTDB-Tk application)
- **Nextflow** (Workflow engine in which the nf-core/mag classification stage is implemented) — https://github.com/nf-core/mag

## Examples

```
nextflow run nf-core/mag --input samplesheet.csv --gtdbtk_db gtdbtk_db/ --cat_db cat_db/ -profile docker
```

## Evaluation signals

- mag_taxonomy.tsv exists and contains one row per input MAG
- Every row has a populated taxonomy value and a method column indicating GTDB-Tk or CAT/BAT
- GTDB-Tk assignments appear only for bins that passed the BUSCO completeness/contamination thresholds
- No input MAG is missing from the merged table — bins failing BUSCO gates must appear with CAT/BAT assignments
- Read-type gating is respected: tools not applicable to a given read type (short/long/hybrid) are not applied

## Limitations

- GTDB-Tk requires marker genes, so it cannot classify MAGs that fail BUSCO completeness/contamination thresholds — these must be routed to CAT/BAT
- Not every tool applies to every read type; nf-core/mag processes short-read, long-read, and hybrid data and the classification step must respect this
- Functional annotation is not part of the current nf-core/mag version described and is listed as future work

## Evidence

- [abstract] We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification.: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] as well as taxonomic and functional annotation of MAGs: "as well as taxonomic and functional annotation of MAGs"
- [intro] only a few pipelines such as Muffin (7) and ATLAS (8) make use of workflow management systems: "only a few pipelines such as Muffin (7) and ATLAS (8) make use of workflow management systems"
- [abstract] The pipeline is easy to install, portable and reproducible, with dependencies provided within containers: "The pipeline is easy to install-all dependencies are provided within containers-portable and reproducible."
