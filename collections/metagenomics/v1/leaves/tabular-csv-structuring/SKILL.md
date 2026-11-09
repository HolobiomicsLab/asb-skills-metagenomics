---
name: tabular-csv-structuring
description: Reference for reproducing published benchmark/runtime tables as machine-readable
  CSV when a source table presents per-pipeline timings (here, seconds) in a dense
  concatenated layout across multiple datasets.
when_to_use_negative:
- The task concerns Table 1 OTU counts (annotated/correctly annotated OTUs per pipeline,
  e.g. DADA2's 91 OTUs on Balanced) rather than Table 2 runtime benchmarking.
- The data is already in machine-readable tabular form (CSV/TSV) — no re-structuring
  is needed.
- The source table's digit layout cannot be unambiguously segmented into per-pipeline
  values and no independent reference (e.g. the published article) is available for
  validation.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DADA2
  role: Benchmarked pipeline whose per-dataset runtime (seconds) forms one column
    of the structured CSV
- name: deblur
  role: Benchmarked pipeline whose per-dataset runtime (seconds) forms one column
    of the structured CSV
- name: CD-HIT-OTU-MiSeq
  role: Source utilities whose runtime appears as the 'source CD-HIT' column of the
    structured CSV
- name: CDSnake
  role: Snakemake pipeline being benchmarked; its runtime appears as the final column
    of the structured CSV
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:27d58a97b3fa0c5c5490b1c4d3300e5557feb37537d12f7e57a8f1e331a4a403
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-MiSeq
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tabular-csv-structuring

## Summary

Structures the CDSnake paper's Table 2 benchmark runtimes — measured in seconds for DADA2, deblur, the source CD-HIT-OTU-MiSeq utilities, and CDSnake across the HMP, Balanced, Extreme, and combined All mock datasets — into a plain-text CSV with a header row and one row per dataset.

## When to use

Reference for reproducing published benchmark/runtime tables as machine-readable CSV when a source table presents per-pipeline timings (here, seconds) in a dense concatenated layout across multiple datasets.

## When NOT to use

- The task concerns Table 1 OTU counts (annotated/correctly annotated OTUs per pipeline, e.g. DADA2's 91 OTUs on Balanced) rather than Table 2 runtime benchmarking.
- The data is already in machine-readable tabular form (CSV/TSV) — no re-structuring is needed.
- The source table's digit layout cannot be unambiguously segmented into per-pipeline values and no independent reference (e.g. the published article) is available for validation.

## Inputs

- Table 2 ('Time, sec') of the CDSnake article, as rendered text or HTML containing concatenated runtime digits for DADA2, deblur, source CD-HIT, and CDSnake
- The benchmark dataset row labels: HMP, Balanced, Extreme, All

## Outputs

- table2_runtimes.csv — plain-text comma-separated file with a header row (one column per pipeline) and 4 data rows (one per dataset: HMP, Balanced, Extreme, All), values in seconds

## How to apply

Locate Table 2 of the CDSnake article (BMC Bioinformatics, 10.1186/s12859-020-03591-6), which reports 'Time, sec' for four pipelines: DADA2, deblur, source CD-HIT, and CDSnake. Parse each dataset row (HMP, Balanced, Extreme, All) and split the concatenated digits into four per-pipeline values, taking care to disambiguate runs of concatenated numbers (e.g. the table string 'HMP4614.81371.722149.592263.83' splits into 4614.81 / 371.72 / 2149.59 / 2263.83, so deblur's HMP runtime is 371.72 s, not 1371.72). Write the result to table2_runtimes.csv as comma-separated plain text with a header row naming the pipelines. Known anchor values: HMP row 4614.81 / 371.72 / 2149.59 / 2263.83 s and All row 17,509.96 / 376.05 / 39,775.51 / 39,891.53 s; the Balanced and Extreme rows must be parsed from the published table itself. Validate the CSV by checking it contains exactly 4 data rows and 4 pipeline columns with every cell matching the reference table (output_matches_reference).

## Related tools

- **DADA2** (Benchmarked pipeline whose per-dataset runtime (seconds) forms one column of the structured CSV)
- **deblur** (Benchmarked pipeline whose per-dataset runtime (seconds) forms one column of the structured CSV)
- **CD-HIT-OTU-MiSeq** (Source utilities whose runtime appears as the 'source CD-HIT' column of the structured CSV)
- **CDSnake** (Snakemake pipeline being benchmarked; its runtime appears as the final column of the structured CSV) — github.com/ydkondratenko/cdsnake

## Examples

```
printf '%s\n' dataset,dada2,deblur,source_cdhit,cdsnake HMP,4614.81,371.72,2149.59,2263.83 > table2_runtimes.csv
```

## Evaluation signals

- The CSV contains exactly 4 data rows (HMP, Balanced, Extreme, All) and 4 pipeline columns, matching Table 2's shape (output_matches_reference).
- Every runtime cell equals the corresponding Table 2 cell exactly, e.g. HMP: DADA2=4614.81, deblur=371.72, source CD-HIT=2149.59, CDSnake=2263.83 s.
- All-datasets row matches the published combined values: 17,509.96 / 376.05 / 39,775.51 / 39,891.53 s.
- Digit-segmentation is correct: concatenated strings like 'HMP4614.81371.722149.592263.83' must yield decimal values (two digits after each point), so deblur's HMP value is 371.72, not 1371.72.
- All values are numeric and expressed in seconds, consistent with the table's 'Time, sec' header.

## Limitations

- The article does not specify the hardware/execution environment used for time benchmarking, so the reported seconds are not directly comparable across machines.
- Software versions are not stated for DADA2, deblur, CD-HIT-OTU-MiSeq, or Snakemake, limiting exact reproduction of the measured runtimes.
- The rendered table concatenates digits without delimiters, creating ambiguity when segmenting values; cross-checking against the published article is required.

## Evidence

- [discussion] table rendering of Table 2: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [discussion] scope of the benchmark: "Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake on three mock community datasets and set of these datasets (All row)"
- [other] expected values and output target: [evidence span withheld — anchor status: not_found]
