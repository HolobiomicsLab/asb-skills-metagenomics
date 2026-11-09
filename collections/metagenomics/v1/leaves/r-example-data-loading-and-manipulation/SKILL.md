---
name: r-example-data-loading-and-manipulation
description: Use when you have a PICRUSt2-format KO abundance table in R and need
  to load example data or aggregate KO ortholog-level abundances into KEGG pathway-level
  abundances for downstream analysis.
when_to_use_negative:
- Input is already a KEGG pathway abundance table (e.g. metagenome predictions at
  pathway level) — no KO-to-pathway conversion is needed
- You are running the full pipeline via ggpicrust2(), which already integrates ko
  abundance to kegg pathway abundance conversion internally
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ggpicrust2
  role: Provides the ko2kegg_abundance() function that converts the KO abundance table
    into a KEGG pathway abundance table and bundles the ko_abundance/metadata example
    datasets
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Execution environment in which the example datasets are loaded with data()
    and the converted table is inspected and written to CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:8a1f4cab5eeb930fa2b9660065d8236ee72e5688434ed9619d8e632bfeb1255d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# r-example-data-loading-and-manipulation

## Summary

Load and manipulate ggpicrust2's bundled example datasets in R — data(ko_abundance) provides a PICRUSt2-format KO abundance matrix (rows = KO orthologs, columns = samples) that can be converted to KEGG pathway-level abundance with ko2kegg_abundance(), a capability PICRUSt2 itself lacks.

## When to use

Use when you have a PICRUSt2-format KO abundance table in R and need to load example data or aggregate KO ortholog-level abundances into KEGG pathway-level abundances for downstream analysis.

## When NOT to use

- Input is already a KEGG pathway abundance table (e.g. metagenome predictions at pathway level) — no KO-to-pathway conversion is needed
- You are running the full pipeline via ggpicrust2(), which already integrates ko abundance to kegg pathway abundance conversion internally

## Inputs

- PICRUSt2-format KO abundance matrix (rows = KO orthologs, columns = samples), e.g. the bundled ko_abundance dataset
- Optional sample metadata table (e.g. the bundled metadata dataset)

## Outputs

- KEGG pathway abundance table (pathway-level rows, same sample columns as input), suitable for export as kegg_pathway_abundance.csv

## How to apply

In R, load ggpicrust2 and attach the bundled example data with data(ko_abundance) and data(metadata); ko_abundance is a PICRUSt2-format KO abundance matrix with KO orthologs as rows and samples as columns. Run ko2kegg_abundance() on this matrix so KO-level abundances are aggregated into KEGG pathway-level abundance rows while retaining the same sample columns. Note that ggpicrust2() also integrates this KO-to-KEGG pathway conversion as the first stage of its pipeline, so the manual conversion is only needed when working with the KO table directly. Inspect the resulting data frame to confirm pathway-level rows and unchanged sample columns, then export it (e.g. to kegg_pathway_abundance.csv) for downstream differential abundance analysis or annotation.

## Related tools

- **ggpicrust2** (Provides the ko2kegg_abundance() function that converts the KO abundance table into a KEGG pathway abundance table and bundles the ko_abundance/metadata example datasets) — https://github.com/cafferychen777/ggpicrust2
- **R** (Execution environment in which the example datasets are loaded with data() and the converted table is inspected and written to CSV)

## Examples

```
data(ko_abundance); kegg_pathway <- ko2kegg_abundance(ko_abundance)
```

## Evaluation signals

- The output file (e.g. kegg_pathway_abundance.csv) exists and parses as a tabular CSV
- The converted table's rows correspond to KEGG pathways (row count greater than zero)
- The sample columns of the converted table match the sample columns of the ko_abundance input matrix
- The row identifiers have switched from KO ortholog IDs to KEGG pathway IDs after ko2kegg_abundance()

## Limitations

- PICRUSt2 alone cannot perform this KO-to-KEGG pathway abundance conversion; the step requires ggpicrust2
- Reference data coverage affects annotation: README notes KO reference data was updated from 23,917 to 27,531 unique KO IDs and EC reference from 3,180 to 8,371 entries, so older package versions may miss recently discovered enzymes/KOs

## Evidence

- [intro] Convert KO abundance profiles to pathway-level (KEGG) abundance: "convert KO profiles to pathway-level abundance when needed"
- [intro] data(ko_abundance); data(metadata) example datasets: "data(ko_abundance)
data(metadata)"
- [readme] ggpicrust2 pipeline integrates the KO-to-KEGG conversion: "ggpicrust2() integrates ko abundance to kegg pathway abundance
conversion, annotation of pathway, differential abundance (DA) analysis,
part of DA results visualization."
- [readme] ko2kegg_abundance() is a documented function of the package: "- [ko2kegg_abundance()](#ko2kegg_abundance)"
