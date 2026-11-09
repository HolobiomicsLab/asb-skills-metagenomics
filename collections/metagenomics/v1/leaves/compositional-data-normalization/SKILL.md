---
name: compositional-data-normalization
description: Use when KO-level read-count or relative-abundance tables from shotgun
  metagenomic annotation must be converted to compositionally meaningful per-genome
  copy numbers before downstream functional comparisons.
when_to_use_negative:
- When the goal is plain relative-abundance normalization rather than per-genome copy-number
  correction — MetaLAFFA supports switching the 'method' parameter from MUSiCC to
  'relative'
- When input is already a MUSiCC-corrected copy-number table; re-running correction
  would double-correct
- When annotating at the pathway or module aggregation level rather than the KO orthology-group
  level
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaLAFFA
  role: Pipeline whose functional annotation phase runs the ortholog abundance-correction
    step via config/steps/ortholog_abundance_correction.py
  repo: https://github.com/borenstein-lab/MetaLAFFA
- name: MUSiCC
  role: 'Default correction method: inter-sample normalization plus intra-sample correction
    with learned models, based on universally single-copy bacterial genes (Manor and
    Borenstein 2015)'
- name: Python
  role: Module-based configuration and custom scripts implementing the correction
    step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-020-03815-9
    title: 'MetaLAFFA: a flexible, end-to-end, distributed computing-compatible metagenomic
      functional annotation pipeline'
  source_content_hash: sha256:ea7c6defdd814f167ba51271e38be733e80bb0d0af7791bb17c9166e67b1ef7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/borenstein-lab/MetaLAFFA
  tools:
  - MUSiCC
  - MetaLAFFA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# compositional-data-normalization

## Summary

Normalize metagenomic functional ortholog (KO) abundances using MUSiCC, MetaLAFFA's default ortholog abundance-correction method, which converts ortholog read counts into average copy number per genome via inter-sample normalization and intra-sample correction rather than leaving simple relative abundances.

## When to use

Use when KO-level read-count or relative-abundance tables from shotgun metagenomic annotation must be converted to compositionally meaningful per-genome copy numbers before downstream functional comparisons.

## When NOT to use

- When the goal is plain relative-abundance normalization rather than per-genome copy-number correction — MetaLAFFA supports switching the 'method' parameter from MUSiCC to 'relative'
- When input is already a MUSiCC-corrected copy-number table; re-running correction would double-correct
- When annotating at the pathway or module aggregation level rather than the KO orthology-group level

## Inputs

- KO (KEGG orthology group) abundance table (TSV) with fractionally counted, gene-length-normalized counts from the ortholog counting step
- Shotgun metagenomic sample set (multiple samples, since MUSiCC inter-sample normalization is used)

## Outputs

- MUSiCC-corrected KO abundance table (TSV) of KOs by samples, values being average KO copy number per genome per sample

## How to apply

Load the KO abundance table produced by MetaLAFFA's ortholog counting step, in which gene abundances have already been fractionally counted (multi-match reads contribute fractional counts) and gene-length normalized. Invoke the ortholog abundance correction step implemented in config/steps/ortholog_abundance_correction.py, keeping the 'method' parameter at its default MUSiCC setting rather than switching to 'relative'. Run MUSiCC (Manor and Borenstein 2015) with inter-sample normalization and intra-sample correction using learned models, which leverages universally single-copy bacterial genes as the normalization basis. Write the corrected table — average KO copy number per genome per sample — to disk as a tab-separated values file.

## Related tools

- **MetaLAFFA** (Pipeline whose functional annotation phase runs the ortholog abundance-correction step via config/steps/ortholog_abundance_correction.py) — https://github.com/borenstein-lab/MetaLAFFA
- **MUSiCC** (Default correction method: inter-sample normalization plus intra-sample correction with learned models, based on universally single-copy bacterial genes (Manor and Borenstein 2015))
- **Python** (Module-based configuration and custom scripts implementing the correction step)

## Examples

```
method = "MUSiCC"  # default in config/steps/ortholog_abundance_correction.py; use "relative" only to skip correction
```

## Evaluation signals

- Output TSV exists and parses as a tab-delimited table of KOs (rows) by samples (columns)
- Values represent MUSiCC-corrected average copy numbers per genome, not raw read-count relative abundances
- The 'method' parameter in config/steps/ortholog_abundance_correction.py was left at its default (MUSiCC), not 'relative'
- Correction is applied to the KO-level profile downstream of fractional gene counting and gene length normalization, preserving consistency with those upstream steps

## Limitations

- MUSiCC relies on universally single-copy bacterial genes as the normalization basis, so heavily disrupted or non-bacterial communities may be poorly modeled
- Known runtime issue: MetaLAFFA can hit an 'undefined symbol: omp_get_num_procs' error when trying to run MUSiCC (documented in the repository FAQ)

## Evidence

- [readme] Default MUSiCC step exists as a distinct pipeline stage: "8. Correcting orthology group abundances"
- [readme] Known MUSiCC runtime failure mode documented in FAQ: "How do I fix the "undefined symbol: omp_get_num_procs" error when MetaLAFFA tries to run MUSiCC?"
- [methods] Upstream gene abundance semantics feeding the correction: "so MetaLAFFA uses a fractional gene counting method by default. Gene length normalization and fractional counting are performed by a custom Python script"
- [intro] Fractional counting rule for multi-matching reads: "If a read matches equally well to multiple genes, then that read contributes a fractional count"
- [readme] Pipeline context: functional orthology group abundance annotation: "MetaLAFFA is a pipeline for annotating shotgun metagenomic data with abundances of functional orthology groups."
