---
name: abundance-table-format-validation
description: Use when a PICRUSt2-derived KO or pathway abundance table is about to
  be transformed or analyzed, and you must confirm rows are features (KO orthologs
  or KEGG pathways), columns are samples, and post-conversion output preserves sample
  columns.
when_to_use_negative:
- The table is already pathway-level and no conversion (e.g. ko2kegg_abundance) is
  planned, so only trivial checks apply
- The input is a differential abundance results table or per-sequence contribution
  file (e.g. pred_metagenome_contrib.tsv), which has a different schema than a feature-by-sample
  abundance matrix
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: ggpicrust2
  role: Provides ko2kegg_abundance() to convert the KO abundance matrix into a KEGG
    pathway abundance table whose schema is then validated
  repo: https://github.com/cafferychen777/ggpicrust2
- name: R
  role: Execution environment in which the ggpicrust2 package, its bundled ko_abundance
    dataset, and validation of the converted data frame are run
- name: PICRUSt2
  role: Produces the upstream functional profile predictions; its KO abundance output
    format is the expected input structure
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:60f8f230ada5c21d24f9ccbefc55311887bd5564e70824a42ba843741ec0f48e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - PICRUSt2
  - R
  - ggpicrust2
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# abundance-table-format-validation

## Summary

Validate that an abundance table (e.g., a PICRUSt2-format KO table or a converted KEGG pathway table) has the expected feature-by-sample matrix structure before and after conversion steps such as ggpicrust2's ko2kegg_abundance(). This guards downstream differential abundance analysis against malformed or mis-shaped inputs.

## When to use

Use when a PICRUSt2-derived KO or pathway abundance table is about to be transformed or analyzed, and you must confirm rows are features (KO orthologs or KEGG pathways), columns are samples, and post-conversion output preserves sample columns.

## When NOT to use

- The table is already pathway-level and no conversion (e.g. ko2kegg_abundance) is planned, so only trivial checks apply
- The input is a differential abundance results table or per-sequence contribution file (e.g. pred_metagenome_contrib.tsv), which has a different schema than a feature-by-sample abundance matrix

## Inputs

- PICRUSt2-format KO abundance matrix (rows = KEGG Orthology (KO) identifiers, columns = samples), e.g. the bundled ggpicrust2 ko_abundance dataset
- Optionally, a previously converted KEGG pathway abundance table to re-validate

## Outputs

- Validated KEGG pathway abundance data frame (rows = KEGG pathways, columns = samples matching the input)
- Written abundance table file, e.g. kegg_pathway_abundance.csv (tabular CSV)

## How to apply

In R, load ggpicrust2 and attach the bundled example dataset with data(ko_abundance), which is a PICRUSt2-format KO abundance matrix with rows = KO orthologs and columns = samples. Run ko2kegg_abundance() on this matrix to aggregate KO-level abundances into KEGG pathway-level rows. Then inspect the resulting data frame to confirm it contains pathway-level rows while retaining the same sample columns as the ko_abundance input, and write it out (e.g., to kegg_pathway_abundance.csv) as a tabular CSV. The rationale is that ggpicrust2 provides KO-to-KEGG pathway conversion that PICRUSt2 itself lacks, so shape preservation across this conversion is the key correctness invariant.

## Related tools

- **ggpicrust2** (Provides ko2kegg_abundance() to convert the KO abundance matrix into a KEGG pathway abundance table whose schema is then validated) — https://github.com/cafferychen777/ggpicrust2
- **R** (Execution environment in which the ggpicrust2 package, its bundled ko_abundance dataset, and validation of the converted data frame are run)
- **PICRUSt2** (Produces the upstream functional profile predictions; its KO abundance output format is the expected input structure)

## Examples

```
library(ggpicrust2); data(ko_abundance); kegg <- ko2kegg_abundance(ko_abundance); stopifnot(ncol(kegg) == ncol(ko_abundance)); write.csv(kegg, "kegg_pathway_abundance.csv")
```

## Evaluation signals

- The output file (e.g. kegg_pathway_abundance.csv) exists and parses as a tabular CSV
- Row count of the converted table is greater than zero and rows correspond to KEGG pathways
- Sample column names in the converted table exactly match the sample columns of the ko_abundance input
- The input matrix has the PICRUSt2 format: rows are KO orthologs and columns are samples before conversion
- The converted data frame is a data frame with pathway-level rows (not KO-level rows) after ko2kegg_abundance()

## Limitations

- PICRUSt2 itself removes/lacks the KO-to-KEGG pathway abundance conversion capability, so validation depends on ggpicrust2's conversion being available and correctly run
- No software version numbers for ggpicrust2 or PICRUSt2 are stated in the source material, so version-specific behavior cannot be pinned down

## Evidence

- [other] The ggpicrust2 function ko2kegg_abundance() converts a KO abundance table into a KEGG pathway abundance table, a capability that PICRUSt2 itself removes/lacks.: [evidence span withheld — anchor status: not_found]
- [readme] ggpicrust2 integrates ko abundance to kegg pathway abundance conversion, annotation of pathway, differential abundance (DA) analysis: [evidence span withheld — anchor status: external_doc]
- [intro] convert KO profiles to pathway-level abundance when needed: "convert KO profiles to pathway-level abundance when needed"
- [intro] It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods: "It offers a wide range of features, including pathway name/description annotations, advanced differential abundance (DA) methods, and visualization of DA results."
