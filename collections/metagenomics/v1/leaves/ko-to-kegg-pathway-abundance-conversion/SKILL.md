---
name: ko-to-kegg-pathway-abundance-conversion
description: Use when you have a PICRUSt2-predicted KO abundance table (rows = KO
  orthologs, columns = samples) and need KEGG pathway-level abundances as input for
  downstream pathway annotation or differential abundance analysis.
when_to_use_negative:
- Input is already a KEGG pathway-level abundance table (e.g. PICRUSt2 metagenome
  pred_metagenome_unstrat at pathway level) — no KO-to-pathway aggregation is needed
- You need EC-level or MetaCyc-level functional profiles rather than KEGG pathway
  aggregation
- You plan differential analysis at the individual KO ortholog level rather than the
  pathway level
tools:
- name: ggpicrust2
  role: Provides ko2kegg_abundance() which converts KO abundance profiles to KEGG
    pathway-level abundance
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Runtime environment in which ggpicrust2 and ko2kegg_abundance() are executed
- name: PICRUSt2
  role: Upstream tool that produces the KO abundance table consumed by this conversion
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:dc4feea4a31f9f17a19c3022ec32833636c2b9f4a51009b344dc78afdd6ce6ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - PICRUSt2
  - R
  - devtools
  - ggpicrust2
  - limma
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# ko-to-kegg-pathway-abundance-conversion

## Summary

Converts a PICRUSt2-format KO (KEGG Orthology) abundance matrix into a KEGG pathway-level abundance table using the ggpicrust2 ko2kegg_abundance() function, filling a capability PICRUSt2 itself lacks, so pathway-level differential abundance analysis can proceed.

## When to use

Use when you have a PICRUSt2-predicted KO abundance table (rows = KO orthologs, columns = samples) and need KEGG pathway-level abundances as input for downstream pathway annotation or differential abundance analysis.

## When NOT to use

- Input is already a KEGG pathway-level abundance table (e.g. PICRUSt2 metagenome pred_metagenome_unstrat at pathway level) — no KO-to-pathway aggregation is needed
- You need EC-level or MetaCyc-level functional profiles rather than KEGG pathway aggregation
- You plan differential analysis at the individual KO ortholog level rather than the pathway level

## Inputs

- PICRUSt2-format KO abundance table (data frame or TSV/CSV: rows = KO ortholog IDs, columns = samples)
- Optional bundled example dataset ko_abundance shipped with ggpicrust2

## Outputs

- KEGG pathway abundance data frame (rows = KEGG pathway IDs, columns = same sample names as the KO input)
- Optionally kegg_pathway_abundance.csv written to disk

## How to apply

Load ggpicrust2 in R and attach the bundled example KO abundance matrix with data(ko_abundance), a PICRUSt2-format matrix with KO orthologs as rows and samples as columns. Run ko2kegg_abundance() on this matrix so KO-level abundances are aggregated (summed) into KEGG pathway-level rows, since PICRUSt2 removes/lacks this conversion step itself. Inspect the resulting data frame to confirm rows are KEGG pathways and sample columns match the input. Write the result to a CSV (e.g. kegg_pathway_abundance.csv) if a persistent artifact is needed, or pass it directly to ggpicrust2 downstream functions; the full ggpicrust2() pipeline integrates this conversion automatically with pathway annotation and DA analysis.

## Related tools

- **ggpicrust2** (Provides ko2kegg_abundance() which converts KO abundance profiles to KEGG pathway-level abundance) — https://github.com/cafferychen777/ggpicrust2
- **R** (Runtime environment in which ggpicrust2 and ko2kegg_abundance() are executed)
- **PICRUSt2** (Upstream tool that produces the KO abundance table consumed by this conversion)

## Examples

```
library(ggpicrust2); data(ko_abundance); kegg_abundance <- ko2kegg_abundance(ko_abundance)
```

## Evaluation signals

- Output rows correspond to KEGG pathway IDs and row count is greater than zero
- Sample columns of the pathway-level output exactly match the sample columns of the input ko_abundance table
- Output parses as a tabular CSV if written to file (e.g. kegg_pathway_abundance.csv exists and is readable as a table)
- No KO ortholog IDs (e.g. K-prefixed identifiers) remain among output rows after conversion

## Limitations

- PICRUSt2 alone cannot perform this conversion; the step depends on ggpicrust2 being installed (via devtools::install_github("cafferychen777/ggpicrust2"))
- Conversion aggregates KO abundances into pathway-level rows, so KO-level resolution is lost; downstream tests operate on pathways, not orthologs

## Evidence

- [intro] convert KO profiles to pathway-level abundance when needed: "convert KO profiles to pathway-level abundance when needed"
- [intro] bundled example dataset: "data(ko_abundance)
data(metadata)"
- [readme] ggpicrust2() integrates ko abundance to kegg pathway abundance conversion: "ggpicrust2() integrates ko abundance to kegg pathway abundance
conversion, annotation of pathway, differential abundance (DA) analysis"
