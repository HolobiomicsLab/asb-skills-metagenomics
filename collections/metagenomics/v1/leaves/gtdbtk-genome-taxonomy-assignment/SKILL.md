---
name: gtdbtk-genome-taxonomy-assignment
description: Use when you have assembled and binned MAG FASTA files with BUSCO quality
  summaries and need GTDB lineage assignments per bin, with a fallback classifier
  for bins that fail quality gates.
when_to_use_negative:
- Bins have no accompanying BUSCO quality summaries, so the completeness/contamination
  gating required to route bins to GTDB-Tk cannot be applied
- Input is raw reads rather than assembled, binned MAG FASTA files — this skill operates
  only on the taxonomic-classification stage after assembly and binning
- Functional (non-taxonomic) annotation of MAGs is the goal; the article lists functional
  annotation as a step not yet included in nf-core/mag
edam_operation: http://edamontology.org/operation_0363
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: GTDB-Tk
  role: Assigns GTDB lineages to MAGs that pass BUSCO completeness/contamination thresholds,
    via classify_wf
- name: CAT/BAT
  role: Fallback classifier applied to all bins that fail the BUSCO gates, using ORF-level
    taxonomic assignment
- name: BUSCO
  role: Estimates completeness and contamination used as quality thresholds gating
    which bins go to GTDB-Tk
- name: nf-core/mag
  role: Host pipeline that orchestrates the binning QC and the GTDB-Tk/CAT/BAT taxonomic
    classification stage
  repo: https://github.com/nf-core/mag
- name: Nextflow
  role: Workflow engine in which the taxonomic classification stage of nf-core/mag
    is implemented
  repo: https://github.com/nf-core/mag
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac007
    title: 'nf-core/mag: a best-practice pipeline for metagenome hybrid assembly and
      binning'
  source_content_hash: sha256:4eadb35b72926465b941495f8c9913df9f5adb70af8b377497b23e9255bb43c6
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
  edam_operation: http://edamontology.org/operation_0363
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gtdbtk-genome-taxonomy-assignment

## Summary

Taxonomic classification of metagenome-assembled genomes (MAGs) in the final stage of nf-core/mag, using GTDB-Tk for quality-gated bins and CAT/BAT as the fallback so that every bin receives a lineage assignment.

## When to use

Use when you have assembled and binned MAG FASTA files with BUSCO quality summaries and need GTDB lineage assignments per bin, with a fallback classifier for bins that fail quality gates.

## When NOT to use

- Bins have no accompanying BUSCO quality summaries, so the completeness/contamination gating required to route bins to GTDB-Tk cannot be applied
- Input is raw reads rather than assembled, binned MAG FASTA files — this skill operates only on the taxonomic-classification stage after assembly and binning
- Functional (non-taxonomic) annotation of MAGs is the goal; the article lists functional annotation as a step not yet included in nf-core/mag

## Inputs

- MAG FASTA bin files from the nf-core/mag binning step
- BUSCO quality summaries (completeness and contamination) per bin
- Read-type configuration (short-read, long-read, or hybrid) to gate tool applicability

## Outputs

- mag_taxonomy.tsv with one row per input MAG, taxonomy column populated, and method column (GTDB-Tk vs CAT/BAT)
- GTDB lineage assignments for quality-passing bins (GTDB-Tk classify_wf output)
- CAT/BAT ORF-level taxonomic assignments for remaining bins

## How to apply

Load the MAG FASTA bin files together with the BUSCO completeness/contamination summaries produced by the upstream binning step of nf-core/mag. Filter bins with BUSCO-derived quality thresholds and route only quality-passing bins to GTDB-Tk, because GTDB-Tk requires marker genes and is therefore only applicable to bins with sufficient gene content; keep in mind that nf-core/mag processes short-read, long-read, and hybrid data, and not every tool applies to every read type. Run GTDB-Tk classify_wf on the passing bins to obtain GTDB lineage assignments. Run CAT/BAT on all remaining bins, since CAT/BAT assigns taxonomy at ORF level and can classify any MAG. Merge the GTDB-Tk and CAT/BAT results into one row per MAG, recording which method produced each assignment, and write the combined table as mag_taxonomy.tsv.

## Related tools

- **GTDB-Tk** (Assigns GTDB lineages to MAGs that pass BUSCO completeness/contamination thresholds, via classify_wf)
- **CAT/BAT** (Fallback classifier applied to all bins that fail the BUSCO gates, using ORF-level taxonomic assignment)
- **BUSCO** (Estimates completeness and contamination used as quality thresholds gating which bins go to GTDB-Tk)
- **nf-core/mag** (Host pipeline that orchestrates the binning QC and the GTDB-Tk/CAT/BAT taxonomic classification stage) — https://github.com/nf-core/mag
- **Nextflow** (Workflow engine in which the taxonomic classification stage of nf-core/mag is implemented) — https://github.com/nf-core/mag

## Examples

```
gtdbtk classify_wf --genome_dir bins_quality_passing --out_dir gtdbtk_out --extension fa
```

## Evaluation signals

- mag_taxonomy.tsv exists and contains exactly one row per input MAG
- Taxonomy and method (GTDB-Tk vs CAT/BAT) columns are populated for every row
- GTDB-Tk assignments appear only for bins that passed the BUSCO-derived completeness/contamination thresholds
- Every bin that failed the BUSCO gates has a CAT/BAT assignment rather than a missing taxonomy
- Merged output does not contain duplicate or conflicting rows for the same MAG from both classifiers

## Limitations

- GTDB-Tk requires marker genes and therefore cannot classify bins lacking them; it must be gated behind BUSCO quality thresholds
- Not every annotation tool applies to every read type, since nf-core/mag processes short-read, long-read, and hybrid data — routing must respect read-type applicability
- Functional annotation is not yet part of the pipeline; the article notes it as future work alongside bin refinement

## Evidence

- [abstract] nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification: "We present the nf-core/mag pipeline for metagenome assembly, binning and taxonomic classification."
- [intro] taxonomic and functional annotation of MAGs: "as well as taxonomic and functional annotation of MAGs"
- [supplementary] nf-core/mag hosted on GitHub: "nf-core/mag code is hosted on GitHub under the nf-core organization https://github.com/nf-core/mag and released under the MIT license."
- [readme] BIgMAG reads nf-core/mag taxonomical annotation output: [evidence span withheld — anchor status: external_doc]
