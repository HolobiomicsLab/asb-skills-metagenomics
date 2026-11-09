---
name: kegg-pathway-set-construction
description: Use when you have PICRUSt2 KO abundance features and need pathway-level
  gene sets (e.g. for ggpicrust2 pathway_gsea()) rather than single-feature differential
  abundance testing.
when_to_use_negative:
- Input is already a pathway-level abundance table rather than KO-level features,
  and single-feature differential abundance analysis is the goal
- Only one comparison group or a design with no contrasts is available, so no enrichment
  contrast can be defined
- Pathway sets cannot reach the minimum size threshold (fewer than 5 features), leaving
  no valid gene sets to test
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: ggpicrust2
  role: Provides pathway_gsea() and ko2kegg_abundance() to map KO abundance into KEGG
    pathway sets and run GSEA on them
  repo: https://github.com/cafferychen777/ggpicrust2
- name: limma
  role: Supplies the camera competitive gene set test used on the constructed pathway
    sets (default method, accounts for inter-gene correlations)
- name: R
  role: Runtime environment for loading bundled datasets and executing the ggpicrust2
    workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad470
    title: 'ggpicrust2: an R package for PICRUSt2 predicted functional profile analysis
      and visualization'
  source_content_hash: sha256:53c907844f9ed45328a095e95ce5501d2aacf1d975cb6a128536fb4e95075da1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cafferychen777/ggpicrust2
  tools:
  - R
  - ggpicrust2
  - limma
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kegg-pathway-set-construction

## Summary

Constructs KEGG pathway gene sets from PICRUSt2-predicted KO abundance features so that gene set enrichment analysis (GSEA) can test whole pathways instead of individual features, using ggpicrust2's pathway-to-gene-set mapping with size filtering.

## When to use

Use when you have PICRUSt2 KO abundance features and need pathway-level gene sets (e.g. for ggpicrust2 pathway_gsea()) rather than single-feature differential abundance testing.

## When NOT to use

- Input is already a pathway-level abundance table rather than KO-level features, and single-feature differential abundance analysis is the goal
- Only one comparison group or a design with no contrasts is available, so no enrichment contrast can be defined
- Pathway sets cannot reach the minimum size threshold (fewer than 5 features), leaving no valid gene sets to test

## Inputs

- PICRUSt2 KO abundance table (e.g. the bundled ko_abundance dataset)
- Sample metadata table (e.g. the bundled metadata dataset)
- KEGG pathway-to-KO mapping used by ggpicrust2/limma gene set construction

## Outputs

- Filtered KEGG pathway gene set collection (pathway IDs mapped to KO features, set sizes between 5 and 500)
- Optionally, pathway-level (KEGG) abundance table via ko2kegg_abundance()

## How to apply

Load the bundled example data with data(ko_abundance) and data(metadata), then build KEGG pathway gene sets directly from the KO abundance features (or first convert KO profiles to pathway-level abundance with ko2kegg_abundance() when pathway-level abundance is needed). Retain only gene sets whose size falls within min_size=5 and max_size=500 so that very small and very large pathway sets do not destabilize enrichment statistics. Feed the filtered gene-set collection into pathway_gsea() from ggpicrust2, preferably with method='camera' (limma camera), which accounts for inter-gene correlations and yields more reliable p-values than preranked approaches. Rationale: by focusing on pathways rather than individual features, GSEA identifies biologically meaningful patterns in PICRUSt2 predicted functional data.

## Related tools

- **ggpicrust2** (Provides pathway_gsea() and ko2kegg_abundance() to map KO abundance into KEGG pathway sets and run GSEA on them) — https://github.com/cafferychen777/ggpicrust2
- **limma** (Supplies the camera competitive gene set test used on the constructed pathway sets (default method, accounts for inter-gene correlations))
- **R** (Runtime environment for loading bundled datasets and executing the ggpicrust2 workflow)

## Examples

```
data(ko_abundance); data(metadata)
```

## Evaluation signals

- gsea_results.csv exists and parses as a valid CSV
- Results table contains pathway ID, NES, p-value and p.adjust columns
- All retained pathway-set sizes fall within the min_size=5 to max_size=500 range
- Every tested pathway ID maps to a KEGG pathway in the constructed gene set collection

## Limitations

- Preranked GSEA variants can produce unreliable p-values because they ignore inter-gene correlations; the camera method is preferred
- No seed or randomness-control information is documented for the GSEA workflow in this context
- No software version numbers for ggpicrust2 or PICRUSt2 are stated, so exact gene-set mapping coverage may vary between releases

## Evidence

- [full_text] Build KEGG pathway gene sets from the KO abundance features, retaining sets with min_size=5 and max_size=500.: [evidence span withheld — anchor status: not_found]
- [readme] The new GSEA module includes: pathway_gsea(): Performs GSEA analysis on PICRUSt2 data: [evidence span withheld — anchor status: external_doc]
- [readme] camera method (new default): Accounts for inter-gene correlations: "**`camera` method (new default)**: Accounts for inter-gene
  correlations, providing more reliable p-values than preranked methods"
- [intro] convert KO profiles to pathway-level abundance when needed: "convert KO profiles to pathway-level abundance when needed"
