---
name: scientific-table-reproduction
description: Reference for reproducing published benchmark/runtime tables (e.g., Table
  2 of the CDSnake paper) when a task requires exact numeric values per pipeline per
  dataset, formatted as a machine-readable CSV with header and one row per dataset.
when_to_use_negative:
- The input is already a structured feature/OTU table or OTU.txt output and needs
  transformation rather than reproduction from prose.
- The task asks for new benchmark measurements of the pipelines rather than extraction
  of values already published in the article's table.
- Only qualitative findings (e.g., 'CDSnake outputted less OTUs than DADA2 and deblur')
  are needed, not exact numeric cells.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0091
tools:
- name: CD-HIT-OTU-Miseq
  role: Source pipeline whose runtime values appear as a column of the reproduced
    table
  repo: github.com/ydkondratenko/cdsnake
- name: CDSnake
  role: Snakemake pipeline whose runtime values appear as a column of the reproduced
    table
  repo: github.com/ydkondratenko/cdsnake
- name: DADA2
  role: Comparison pipeline whose runtime values appear as a column of the reproduced
    table
- name: deblur
  role: Comparison pipeline whose runtime values appear as a column of the reproduced
    table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:95d9989cc203fd0b154a5d3c3371385b5a678f6bb4245e765c9e188caf0b9469
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# scientific-table-reproduction

## Summary

Extract and reproduce a published benchmark table verbatim as a structured CSV. This skill is grounded in Table 2 of the CDSnake paper, which reports runtimes in seconds for DADA2, deblur, source CD-HIT-OTU-MiSeq utilities, and CDSnake across the HMP, Balanced, Extreme, and All datasets.

## When to use

Reference for reproducing published benchmark/runtime tables (e.g., Table 2 of the CDSnake paper) when a task requires exact numeric values per pipeline per dataset, formatted as a machine-readable CSV with header and one row per dataset.

## When NOT to use

- The input is already a structured feature/OTU table or OTU.txt output and needs transformation rather than reproduction from prose.
- The task asks for new benchmark measurements of the pipelines rather than extraction of values already published in the article's table.
- Only qualitative findings (e.g., 'CDSnake outputted less OTUs than DADA2 and deblur') are needed, not exact numeric cells.

## Inputs

- Published article text containing the benchmark table (Table 2 of the CDSnake paper, time in seconds)
- Source cards specifying the research question, workflow, and tools (CD-HIT-OTU-Miseq, manual expert review)

## Outputs

- table2_runtimes.csv — plain-text CSV with a header row of pipeline names (DADA2, deblur, source CD-HIT, CDSnake) and 4 data rows (HMP, Balanced, Extreme, All)

## How to apply

Locate the target table in the article (here, the discussion section's time benchmarking table reporting 'Time, sec' for DADA2, deblur, source CD-HIT, and CDSnake). Parse row-by-row, capturing the four per-pipeline runtime values for each dataset row — HMP: 4614.81 / 371.72 / 2149.59 / 2263.83 s; All: 17,509.96 / 376.05 / 39,775.51 / 39,891.53 s — plus the Balanced and Extreme rows. Preserve thousands separators handling consistently and do not confuse the HMP deblur value (371.72, occasionally transcribed as 1371.72 in source workflows) with other cells. Structure results as a plain-text CSV with one row per dataset (HMP, Balanced, Extreme, All) and one column per pipeline, then write to table2_runtimes.csv. Validate that the CSV contains exactly 4 data rows and 4 pipeline columns, and diff every cell against the published table (output_matches_reference).

## Related tools

- **CD-HIT-OTU-Miseq** (Source pipeline whose runtime values appear as a column of the reproduced table) — github.com/ydkondratenko/cdsnake
- **CDSnake** (Snakemake pipeline whose runtime values appear as a column of the reproduced table) — github.com/ydkondratenko/cdsnake
- **DADA2** (Comparison pipeline whose runtime values appear as a column of the reproduced table)
- **deblur** (Comparison pipeline whose runtime values appear as a column of the reproduced table)

## Examples

```
printf 'dataset,DADA2,deblur,source_CD_HIT,CDSnake\nHMP,4614.81,371.72,2149.59,2263.83\n' > table2_runtimes.csv
```

## Evaluation signals

- CSV contains exactly 4 data rows (HMP, Balanced, Extreme, All) and 4 pipeline columns (DADA2, deblur, source CD-HIT, CDSnake).
- Every cell matches the corresponding cell of Table 2 exactly (output_matches_reference against the published table), e.g., HMP row reads 4614.81, 371.72, 2149.59, 2263.83 and All row reads 17509.96, 376.05, 39775.51, 39891.53.
- Units are consistently seconds ('Time, sec') across all cells; no unit or decimal-separator transcription errors.
- A header row with pipeline names precedes the data rows and no extra columns or rows are present.

## Limitations

- The article does not specify the hardware or execution environment used for the time benchmarking, so the reproduced runtimes cannot be contextualized or re-measured.
- Software versions for DADA2, deblur, CD-HIT-OTU-MiSeq, and Snakemake are not stated in the article, limiting reproducibility of the underlying measurements.
- No accessions or availability details are given for the three mock community datasets (HMP, Balanced, Extreme), so rows cannot be independently regenerated.

## Evidence

- [discussion] Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake on three mock community datasets and set of these datasets (All row): "Time benchmarking of DADA2, deblur, source CH-HIT-OUT-Miseq utilities and CDSnake on three mock community datasets and set of these datasets (All row)"
- [discussion] Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [abstract] Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme: "Three mock datasets having highly overlapping paired-end 2 × 250 bp reads were used for benchmarking - Balanced, HMP, and Extreme"
