---
name: de-bruijn-graph-probe-matching
description: Use when raw FASTQ reads from mycobacterial or staphylococcal isolates
  need rapid phylo_group, species, and lineage calls — e.g. distinguishing MTBC from
  NTM or concordance-checking lineage assignments against a reference caller.
when_to_use_negative:
- The goal is antimicrobial-resistance prediction rather than species/lineage identification
  — use the resistance module of Mykrobe predictor instead
- Input is already assembled genomes or variant calls rather than raw sequence data;
  the de Bruijn probe matcher takes raw sequence reads as input
- The organisms of interest are outside Mykrobe's supported species set (M. tuberculosis,
  S. aureus, S. sonnei, S. typhi, S. enterica serotype Paratyphi B)
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0625
tools:
- name: Mykrobe predictor
  role: Core tool matching reads against the de Bruijn graph probe panel to call phylo_group,
    species, and lineage; run with --format  and --panel 202010
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: mykrobe-parser
  role: R parser that converts Mykrobe JSON outputs into tidy CSVs including output-data.csv
    with speciation data (phylo_group, species, lineage with depths and coverage)
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:7c62a0d5f3aa6ac2e8c5d74fcf1b34726b78358f41a9e8a4b42ad1823f9e9357
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  - mykrobe-parser
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# De Bruijn graph probe matching for species/lineage identification

## Summary

Species and lineage identification by matching raw sequencing reads against a de Bruijn graph representation of bacterial diversity via Mykrobe predictor, as used to classify MTBC vs non-tuberculous mycobacteria and assign M. tuberculosis lineages.

## When to use

Use when raw FASTQ reads from mycobacterial or staphylococcal isolates need rapid phylo_group, species, and lineage calls — e.g. distinguishing MTBC from NTM or concordance-checking lineage assignments against a reference caller.

## When NOT to use

- The goal is antimicrobial-resistance prediction rather than species/lineage identification — use the resistance module of Mykrobe predictor instead
- Input is already assembled genomes or variant calls rather than raw sequence data; the de Bruijn probe matcher takes raw sequence reads as input
- The organisms of interest are outside Mykrobe's supported species set (M. tuberculosis, S. aureus, S. sonnei, S. typhi, S. enterica serotype Paratyphi B)

## Inputs

- Raw paired-end sequencing reads (FASTQ) from mycobacterial isolates, e.g. MTBC_A2 (1,157 MTBC isolates) and Myco_Retro (147 mycobacterial isolates) from ENA/NCBI SRA
- Mykrobe species/lineage probe panel (e.g. the 202010 panel)
- Expected species labels for cross-tabulation (MTBC for MTBC_A2, NTM for Myco_Retro); optional precomputed KvarQ lineage calls for concordance checking

## Outputs

- Per-sample JSON of Mykrobe predictor calls with phylo_group, species, and lineage entries plus depths and percent coverage
- output-data.csv (via mykrobe-parser) containing speciation data
- Per-isolate species/lineage call table and concordance summary (e.g. species_lineage_concordance.csv)

## How to apply

Run `mykrobe predict` on each isolate with `--format ` so the full speciation block is retained, selecting the `--panel 202010` (or 201901) species/lineage probe set, and apply the default QC thresholds `--min_depth 5` and `--min_variant_conf 10`. The predictor matches reads against a de Bruijn graph probe panel of M. tuberculosis and related species diversity, emitting per-sample phylo_group, species, and lineage calls with associated depths and percent coverage in the JSON output. Parse these JSON files (e.g. with phac-nml/mykrobe-parser, which populates output-data.csv with speciation data) and cross-tabulate calls against expected labels such as MTBC for MTBC_A2 isolates versus non-tuberculous mycobacteria for Myco_Retro isolates. Compare MTBC lineage calls against an independent caller such as KvarQ to compute concordance; in the source evaluation this yielded 0 MTBC/NTM misclassifications and 100% lineage concordance.

## Related tools

- **Mykrobe predictor** (Core tool matching reads against the de Bruijn graph probe panel to call phylo_group, species, and lineage; run with --format  and --panel 202010) — https://github.com/Mykrobe-tools/mykrobe
- **mykrobe-parser** (R parser that converts Mykrobe JSON outputs into tidy CSVs including output-data.csv with speciation data (phylo_group, species, lineage with depths and coverage)) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Zero samples misclassified between MTBC and NTM across the union of MTBC_A2 and Myco_Retro; any MTBC-vs-NTM switch indicates a failure
- Lineage calls on MTBC_A2 are 100% concordant with precomputed KvarQ lineage calls
- Every per-sample JSON contains non-empty phylo_group, species, and lineage entries with associated depth and percent coverage; unresolved species-level calls within MTBC should be rare
- Samples flagged by Mykrobe as containing non-tuberculous mycobacteria are excluded from downstream quality control, per the parser's convention
- output-data.csv columns match the parser schema (phylo_group, species, lineage with depths and percentages of coverage)

## Limitations

- Only a handful of MTBC samples could not be resolved to species level, so within-MTBC sub-species resolution is not guaranteed for every isolate
- Mykrobe parser is compatible only with specific versions and panels (Mykrobe v0.7.0, 201901/202010 panels) and requires --format  output; CSV output loses information
- The resistance panel's species/lineage panel scope is limited to supported organisms (M. tuberculosis, S. aureus, Shigella sonnei, Salmonella typhi, S. enterica serovar Paratyphi B)

## Evidence

- [abstract] De Bruijn graph probe matching for species/lineage identification: "Here we show how de Bruijn graph representation of bacterial diversity can be used to identify species and resistance profiles of clinical isolates"
