---
name: kegg-functional-annotation-mapping
description: Use when shotgun metagenome reads have been mapped to a non-redundant
  gene catalog (e.g. IGC) and per-sample gene frequencies must be translated into
  KO, EC number and reaction level functional abundance tables for case-control biomarker
  analysis.
when_to_use_negative:
- Input is already a functional abundance table (KO/EC/reaction profiles) — re-normalizing
  aggregated values would double-correct compositional bias; normalization must be
  done at the gene level first
- No appropriate gene catalog or KEGG annotation is available — the article notes
  mapping-based methods are preferred only when appropriate gene catalogs exist; otherwise
  assembly-based approaches are needed
- The goal is purely taxonomic profiling with no functional interpretation — use the
  taxon-level (e.g. MetaPhlAn2) branch of CAMAMED instead
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: CAMAMED
  role: Pipeline that maps genes to the KEGG database and aggregates CSS-normalized
    gene frequencies into KO, EC number and reaction tables
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: Performs cumulative sum-scaling (CSS) normalization of gene-level abundances
    (zero-inflated log-normal model) prior to KEGG functional aggregation
- name: Docker
  role: Ready-to-use camamed_pipeline / camamed_pipeline_db images provide KEGG mapping
    without separate dependency installation
  repo: https://hub.docker.com
- name: KEGG
  role: Reference database from which per-gene KO group, EC number and reaction annotations
    are extracted
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:9eec729656cacff9911cb6a172391723b213e7dd829dc1d96833c796c3f10f61
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - KEGG
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# kegg-functional-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps length- and CSS-normalized gene frequencies to the KEGG database within the CAMAMED pipeline to extract functional annotations at KEGG ortholog (KO) group, EC number and reaction levels, then aggregates normalized gene values per functional category for downstream differential analysis.

## When to use

Use when shotgun metagenome reads have been mapped to a non-redundant gene catalog (e.g. IGC) and per-sample gene frequencies must be translated into KO, EC number and reaction level functional abundance tables for case-control biomarker analysis.

## When NOT to use

- Input is already a functional abundance table (KO/EC/reaction profiles) — re-normalizing aggregated values would double-correct compositional bias; normalization must be done at the gene level first
- No appropriate gene catalog or KEGG annotation is available — the article notes mapping-based methods are preferred only when appropriate gene catalogs exist; otherwise assembly-based approaches are needed
- The goal is purely taxonomic profiling with no functional interpretation — use the taxon-level (e.g. MetaPhlAn2) branch of CAMAMED instead

## Inputs

- CSS-normalized gene frequency table (per-sample gene counts from mapping to a non-redundant gene catalog such as IGC)
- Gene-to-KEGG annotation mapping from the KEGG database (KO groups, EC numbers, reactions per gene)
- Gene length information for length normalization

## Outputs

- Normalized KO group frequency table per sample
- Normalized EC number frequency table per sample
- Normalized reaction frequency table per sample
- css_normalized_frequencies.tsv containing gene, species and functional-level normalized tables

## How to apply

After read mapping yields per-sample gene frequencies (e.g. ~3,354,281 mapped genes against the IGC catalog), filter genes with fewer than five mapped reads summed across all samples, then divide each gene frequency by its gene length and apply CSS normalization with metagenomeSeq. Map the retained genes to the KEGG database to extract annotations at three functional levels: KEGG ortholog (KO) groups, Enzyme Commission (EC) numbers and reactions. For each functional category, compute normalized frequencies by summing the CSS-normalized frequencies of the subset genes carrying that annotation, rather than re-normalizing raw counts at the functional level. Write the aggregated KO, EC and reaction frequency tables (e.g. css_normalized_frequencies.tsv) so differential abundance testing (Kruskal-Wallis H with Benjamini-Hochberg correction) can be run independently at each level. This gene-to-function aggregation is what enables CAMAMED to profile functional differences beyond the gene level, where the paper reports higher detection of changed entities (KO 2.98%, EC 2.55%, reactions 2.45% vs genes 1.92% at P <= 0.01).

## Related tools

- **CAMAMED** (Pipeline that maps genes to the KEGG database and aggregates CSS-normalized gene frequencies into KO, EC number and reaction tables) — https://github.com/mhnb/camamed
- **metagenomeSeq** (Performs cumulative sum-scaling (CSS) normalization of gene-level abundances (zero-inflated log-normal model) prior to KEGG functional aggregation)
- **Docker** (Ready-to-use camamed_pipeline / camamed_pipeline_db images provide KEGG mapping without separate dependency installation) — https://hub.docker.com
- **KEGG** (Reference database from which per-gene KO group, EC number and reaction annotations are extracted)

## Examples

```
docker run camamed/camamed_pipeline_db python camamed_init
```

## Evaluation signals

- Aggregated KO/EC/reaction frequencies must equal the sum of CSS-normalized frequencies of subset genes within each annotation category (not independently normalized)
- Genes with fewer than five mapped reads summed across all samples must be absent from the normalized input before aggregation
- Reproduced differential-analysis ratios at P <= 0.01 should match the reported values: 2.98% KO groups, 2.55% EC numbers, 2.45% reactions, versus 1.92% genes and 2.67% species
- Output tables (e.g. css_normalized_frequencies.tsv) must contain BH-adjusted P values from Kruskal-Wallis testing at all five levels (taxon, gene, KO, EC, reaction)
- Mapped-gene count against the reference catalog should be verifiable (e.g. 3,354,281 genes for the cohort1 IGC mapping)

## Limitations

- metagenomeSeq-based normalization has a reported high false-positive rate for differential abundance analysis when sample size is small
- Without CSS normalization, results from independent colorectal cancer datasets were not consistent, so results are sensitive to skipping the normalization step
- Mapping-based functional analysis depends on the availability of an appropriate non-redundant gene catalog; no software version is stated for CAMAMED or its component tools
- Aggregation is limited to the three KEGG-derived functional levels (KO, EC number, reaction); other functional schemes (e.g. GO) are not covered

## Evidence

- [full_text] computes normalized KO, EC number, and reaction frequencies by summing the normalized frequencies of their subset genes: [evidence span withheld — anchor status: not_found]
- [full_text] Filter the count matrix to remove genes with fewer than five mapped reads summed across all samples: [evidence span withheld — anchor status: not_found]
