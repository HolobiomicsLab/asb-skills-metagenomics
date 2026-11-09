---
name: phage-replication-cycle-classification
description: Use when you have a set of predicted viral contigs (e.g. from VirSorter2/VIBRANT/DeepVirFinder
  + CheckV) and need to assign each contig a lytic, lysogenic, or chronic replication-cycle
  label plus candidate host.
when_to_use_negative:
- Input is raw metagenomic reads rather than a curated set of predicted viral contigs
  — the classifier expects contigs already identified as viral
- 'Samples contain excessive environmental contamination: ViroProfiler notes this
  ''could increase the running time of the pipeline and result in lower precision'',
  so pre-isolate the viral fraction before sequencing'
- You only need lytic/lysogenic calls and cannot run both tools — the skill's core
  value is reconciling BACPHLIP and Replidec per contig
edam_operation: http://edamontology.org/operation_2492
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: BACPHLIP
  role: Scores lytic/lysogenic lifestyle from genetic signatures, run with --multi_fasta
    on the viral contig multi-FASTA
  repo: https://github.com/adamhockenberry/bacphlip
- name: Replidec
  role: Alternative replication-cycle predictor classifying contigs as lytic, lysogenic,
    or chronic
  repo: https://github.com/deng-lab/Replidec
- name: iPhoP
  role: Host prediction step whose output is merged with replication-cycle calls into
    the per-contig table
- name: ViroProfiler
  role: Containerized pipeline orchestrating the VIRALHOST_IPHOP, BACPHLIP and REPLIDEC
    processes
  repo: https://github.com/deng-lab/viroprofiler
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:36f19e7e431212e85eed1a15c4e682d5b2fb7b27d61150a7f57c7e887fd53298
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/adamhockenberry/bacphlip
  tools:
  - BACPHLIP
  - Replidec
  - ViroProfiler
  - iPhoP
  edam_operation: http://edamontology.org/operation_2492
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# phage-replication-cycle-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predicts the replication cycle (lytic, lysogenic, or chronic) of predicted viral contigs using two complementary tools — BACPHLIP (lytic/lysogenic lifestyle scoring) and Replidec (three-cycle classification) — within the ViroProfiler pipeline, reconciling their calls into a per-contig table alongside iPhoP host predictions.

## When to use

Use when you have a set of predicted viral contigs (e.g. from VirSorter2/VIBRANT/DeepVirFinder + CheckV) and need to assign each contig a lytic, lysogenic, or chronic replication-cycle label plus candidate host.

## When NOT to use

- Input is raw metagenomic reads rather than a curated set of predicted viral contigs — the classifier expects contigs already identified as viral
- Samples contain excessive environmental contamination: ViroProfiler notes this 'could increase the running time of the pipeline and result in lower precision', so pre-isolate the viral fraction before sequencing
- You only need lytic/lysogenic calls and cannot run both tools — the skill's core value is reconciling BACPHLIP and Replidec per contig

## Inputs

- Viral contig set (multi-FASTA) identified upstream by VirSorter2, VIBRANT, DeepVirFinder and CheckV
- ViroProfiler custom.config specifying resource limits for VIRALHOST_IPHOP, BACPHLIP and REPLIDEC processes

## Outputs

- Per-contig table with one row per input viral contig, containing iPhoP host predictions and BACPHLIP/Replidec replication-cycle labels (lytic, lysogenic, or chronic)

## How to apply

Start from the viral contig set produced upstream by VirSorter2, VIBRANT, DeepVirFinder and CheckV in ViroProfiler. Run iPhoP (process VIRALHOST_IPHOP, configured with 12 CPUs and 300 GB memory in the ViroProfiler custom.config) on the contigs to assign candidate hosts. Run BACPHLIP with the --multi_fasta flag (process BACPHLIP, 4 CPUs, 80 GB) on the multi-FASTA of predicted viral contigs to score lytic/lysogenic lifestyle; BACPHLIP works from genetic signatures of viral sequences rather than homology alone. Run Replidec (process REPLIDEC, 4 CPUs, 80 GB) as the alternative replication-cycle predictor to classify contigs as lytic, lysogenic, or chronic. Reconcile BACPHLIP and Replidec calls per contig and merge them with the iPhoP host predictions into one per-contig table.

## Related tools

- **BACPHLIP** (Scores lytic/lysogenic lifestyle from genetic signatures, run with --multi_fasta on the viral contig multi-FASTA) — https://github.com/adamhockenberry/bacphlip
- **Replidec** (Alternative replication-cycle predictor classifying contigs as lytic, lysogenic, or chronic) — https://github.com/deng-lab/Replidec
- **iPhoP** (Host prediction step whose output is merged with replication-cycle calls into the per-contig table)
- **ViroProfiler** (Containerized pipeline orchestrating the VIRALHOST_IPHOP, BACPHLIP and REPLIDEC processes) — https://github.com/deng-lab/viroprofiler

## Examples

```
bacphlip --multi_fasta viral_contigs.fasta
```

## Evaluation signals

- Output table has one row per input viral contig
- Host and replication-cycle columns are populated (not empty) for each contig
- Replication-cycle labels fall within the expected set: lytic, lysogenic, or chronic
- Pipeline processes VIRALHOST_IPHOP, BACPHLIP, and REPLIDEC completed without error
- BACPHLIP and Replidec calls are reconciled per contig rather than emitted as separate inconsistent labels

## Limitations

- BACPHLIP scores only lytic vs lysogenic lifestyle; the 'chronic' cycle label requires Replidec
- Disagreement between BACPHLIP and Replidec calls must be reconciled manually or by policy; no consensus rule is specified in the source card
- ViroProfiler is 'specifically designed for classifying viruses in samples with isolated viruses' — metagenomic contamination lowers precision

## Evidence

- [other] BACPHLIP and Replidec predict viral replication cycles by using genetic signatures of viral sequences associated with three cycle types: lytic, lysogenic, and chronic.: [evidence span withheld — anchor status: not_found]
- [readme] BACPHLIP - a bacteriophage lifestyle prediction tool: [evidence span withheld — anchor status: not_found]
- [readme] Replidec: Replication Cycle Detector for Phages: [evidence span withheld — anchor status: external_doc]
