---
name: genomic-epidemiology-reporting
description: Use when in-silico virulence/typing gene calls from sequencing reads
  must be validated against PCR or another phenotypic reference across a clinical
  isolate cohort, with per-isolate concordance reported.
when_to_use_negative:
- Parsing M. tuberculosis resistance results with phac-nml/mykrobe-parser — the parser
  is written for Mykrobe's MTB-specific 201901/202010 panels, not S. aureus PVL probes
- Only text/CSV Mykrobe output is available — per the README, CSV 'does not fit easily
  in tabular/csv format' and some per-probe information is lost
- Samples are suspected mixed cultures — the article removed samples exceeding 40
  confident heterozygous SNPs as contaminated by an unrelated strain
edam_operation: http://edamontology.org/operation_0483
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3303
tools:
- name: Mykrobe predictor
  role: Runs de Bruijn graph-based species and gene-presence prediction on raw read
    data, emitting per-probe PVL calls (depth, % coverage) as JSON
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: phac-nml/mykrobe-parser
  role: R script demonstrating the JSON-in / tidy-CSV-out pattern for Mykrobe results
    (MTB-specific; template for custom PVL JSON parsing)
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:301a334bd630823b81bb14106d75adf829fc04f1f0d336682d2abcc8d2d8d957
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe predictor
  - phac-nml/mykrobe-parser
  edam_operation: http://edamontology.org/operation_0483
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3303
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# genomic-epidemiology-reporting

## Summary

Generates per-isolate in-silico gene-presence genotypes (e.g. PVL lukPV-S/lukPV-K) from bacterial outbreak-isolate read sets using Mykrobe predictor, then joins them to a PCR reference phenotype table to report typing concordance across the outbreak cohort.

## When to use

Use when in-silico virulence/typing gene calls from sequencing reads must be validated against PCR or another phenotypic reference across a clinical isolate cohort, with per-isolate concordance reported.

## When NOT to use

- Parsing M. tuberculosis resistance results with phac-nml/mykrobe-parser — the parser is written for Mykrobe's MTB-specific 201901/202010 panels, not S. aureus PVL probes
- Only text/CSV Mykrobe output is available — per the README, CSV 'does not fit easily in tabular/csv format' and some per-probe information is lost
- Samples are suspected mixed cultures — the article removed samples exceeding 40 confident heterozygous SNPs as contaminated by an unrelated strain

## Inputs

- FASTQ read sets for the 67 S. aureus clinical outbreak isolates (ENA / NCBI SRA, Supplementary Data 5)
- Reference PCR phenotype table for the 67 isolates (23 PVL-negative, 44 PVL-positive)
- Mykrobe predictor JSON output per isolate (per-probe depth and percentage coverage)

## Outputs

- Per-isolate PVL presence/absence calls with supporting probe depth and percentage coverage
- Concordance summary (true positives, true negatives, discordant calls) in pvl_concordance.csv

## How to apply

Install Mykrobe via bioconda (or `pip3 install . && mykrobe panels update_metadata && mykrobe panels update_species all`) and run `mykrobe predict` on each of the 67 S. aureus outbreak-isolate read sets (Supplementary Data 5, retrieved from ENA/NCBI SRA) with `--format json` so per-probe results are emitted as machine-readable JSON. Parse each isolate's JSON to extract the PVL presence/absence call together with supporting probe depth and percentage coverage; note that the phac-nml Mykrobe Parser only consumes JSON output and only supports the M. tuberculosis 201901/202010 panels, so S. aureus PVL extraction requires custom JSON parsing. Join the called PVL status to the reference PCR phenotype table (23 PVL-negative, 44 PVL-positive isolates) and compute concordance counts (true positives, true negatives, discordant calls). Write the per-isolate calls plus the concordance summary to pvl_concordance.csv, recording the Mykrobe/panel version in the output for reproducibility.

## Related tools

- **Mykrobe predictor** (Runs de Bruijn graph-based species and gene-presence prediction on raw read data, emitting per-probe PVL calls (depth, % coverage) as JSON) — https://github.com/Mykrobe-tools/mykrobe
- **phac-nml/mykrobe-parser** (R script demonstrating the JSON-in / tidy-CSV-out pattern for Mykrobe results (MTB-specific; template for custom PVL JSON parsing)) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample outbreak01 --species saureus --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- All 67 isolates concordant: 44/44 correct positive PVL calls and 23/23 correct negative calls (100% concordance) vs PCR
- Each isolate's JSON output contains per-probe depth and percentage-coverage values that back the presence/absence call
- Failed gene predictions are explicitly flagged (parser convention: '[antimicrobial]_prediction' columns set to 'failed' and genetic regions to 'NA') rather than silently missing
- pvl_concordance.csv contains one row per isolate (67 rows) plus a concordance summary, with tool/panel version recorded

## Limitations

- phac-nml/mykrobe-parser is M. tuberculosis specific and tied to the 201901/202010 panels (Mykrobe v0.7.0 compatibility), so S. aureus PVL output must be parsed with custom code
- Some results cannot be represented in CSV without information loss; JSON is the recommended machine-readable format
- For M. tuberculosis predictions, sensitivity is only 82.6% (excludes pyrazinamide) because resistance genetics are poorly understood — analogous typing panels for poorly characterized loci need large training sets
- Contaminated samples (e.g. >40 confident heterozygous SNPs) must be excluded before calling, as in the article's QC (6 training / 20 validation samples removed)

## Evidence

- [abstract] Mykrobe predictor generates a clinician-friendly report within 3 minutes on a laptop: [evidence span withheld — anchor status: not_found]
- [discussion] Freely available predictor software: "The Mykrobe predictor software is freely available (opensource) at www.github.com/iqbal-lab/Mykrobe-predictor for non-commercial academic and research use only"
- [readme] JSON output requirement: "Stores results a .json file instead of a text file. Mykrobe Parser only works with json files."
- [readme] CSV information loss: "(It is possible to output most of the results as CSV, but some information is lost (does not fit easily
in tabular/csv format)"
- [readme] Failed-prediction flagging convention: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
- [discussion] Contamination QC threshold: "We used this threshold of 40 heterozygous sites to determine whether a sample was contaminated by an unrelated strain"
