---
name: metagenomic-count-filtering
description: Use when a mapped gene frequency count table from a mapping-based metagenomic
  pipeline is sparse and compositional and must be filtered, length-corrected and
  CSS-normalized before taxonomic or functional (KO/EC/reaction) profiling.
when_to_use_negative:
- Input is already a normalized or relative-frequency table (e.g. outputs of RAIDA-style
  relative-frequency conversion) — reapplying CSS would double-correct compositional
  bias
- The analysis follows an assembly-based strategy (contig assembly and gene prediction)
  rather than mapping-based quantification against a gene catalog
- Only raw read-level QC is needed — this skill operates on mapped count tables, not
  raw sequencing data
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: CAMAMED
  role: Pipeline whose normalization module performs the five-read filtering, gene-length
    division, CSS normalization and KO/EC/reaction aggregation
  repo: https://github.com/mhnb/camamed
- name: metagenomeSeq
  role: R package supplying the cumulative sum-scaling (CSS) normalization and zero-inflated
    log-normal handling of sparse compositional data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:527637b2b6c683e6c8dad29a10fb9ac8ec43de000775055793b3baca571ebefe
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
  - metagenomeSeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenomic-count-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess CAMAMED mapped gene frequency count tables by removing genes with fewer than five mapped reads summed across all samples, dividing frequencies by gene length, and applying cumulative sum-scaling (CSS) normalization via metagenomeSeq at gene and species levels before aggregating to KO, EC number and reaction tables.

## When to use

Use when a mapped gene frequency count table from a mapping-based metagenomic pipeline is sparse and compositional and must be filtered, length-corrected and CSS-normalized before taxonomic or functional (KO/EC/reaction) profiling.

## When NOT to use

- Input is already a normalized or relative-frequency table (e.g. outputs of RAIDA-style relative-frequency conversion) — reapplying CSS would double-correct compositional bias
- The analysis follows an assembly-based strategy (contig assembly and gene prediction) rather than mapping-based quantification against a gene catalog
- Only raw read-level QC is needed — this skill operates on mapped count tables, not raw sequencing data

## Inputs

- Mapped gene frequency count table (genes x samples) from read mapping to a non-redundant gene catalog
- Gene length annotations for each catalog gene
- Species-level (taxon) abundance count table
- Gene-to-KEGG annotation mapping (KO groups, EC numbers, reactions)

## Outputs

- Filtered, length-corrected, CSS-normalized gene frequency table
- CSS-normalized species frequency table
- Normalized KO, EC number and reaction frequency tables aggregated by summing subset genes
- css_normalized_frequencies.tsv (combined normalized gene, species and functional-level tables)

## How to apply

Load the gene frequency count table produced by upstream read mapping to a non-redundant gene catalog and first drop any gene whose mapped reads summed across all samples are fewer than five. Divide each retained gene's frequency by its gene length to remove length dependence. Apply cumulative sum-scaling (CSS) normalization using the metagenomeSeq package, which exploits a zero-inflated log-normal model well suited to sparse compositional abundance data; repeat CSS independently at the species (taxon) level. Finally, aggregate normalized gene frequencies into KEGG ortholog (KO), enzyme commission (EC) number and reaction tables by summing gene-level values within each functional annotation category, and write the normalized gene, species and functional tables to css_normalized_frequencies.tsv. CSS is applied because correction for zero-inflation and compositional bias is largely neglected in other pipelines, and CAMAMED showed the two colorectal cancer gut datasets were inconsistent without CSS.

## Related tools

- **CAMAMED** (Pipeline whose normalization module performs the five-read filtering, gene-length division, CSS normalization and KO/EC/reaction aggregation) — https://github.com/mhnb/camamed
- **metagenomeSeq** (R package supplying the cumulative sum-scaling (CSS) normalization and zero-inflated log-normal handling of sparse compositional data)

## Examples

```
python camamed_init
```

## Evaluation signals

- No gene with fewer than five mapped reads summed across all samples remains in the output table
- Each gene frequency equals its input count divided by gene length before CSS scaling is applied
- CSS normalization factors are computed per sample (cumulative sum-scaling) and sparse zeros are retained rather than imputed to a constant
- Each KO, EC number and reaction value equals the sum of the CSS-normalized frequencies of its subset genes
- All normalized gene, species and functional tables are present in css_normalized_frequencies.tsv

## Limitations

- metagenomeSeq has a reported high false-positive rate for differential abundance analysis when sample size is small, so downstream DA results on small cohorts need caution
- No software version is stated for CAMAMED or its component tools in the article or repository README, so exact CSS parameter defaults may vary
- Without applying CSS the two colorectal cancer gut metagenome datasets gave inconsistent results, indicating sensitivity of cross-dataset reproducibility to this normalization

## Evidence
