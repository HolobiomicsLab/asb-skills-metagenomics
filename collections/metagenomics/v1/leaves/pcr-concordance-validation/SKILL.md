---
name: pcr-concordance-validation
description: Use when sequence-based variant or gene-presence calls (e.g., Mykrobe
  PVL or resistance calls) must be checked for concordance against an existing PCR
  or phenotypic reference table on a defined isolate panel.
when_to_use_negative:
- The comparison target is antimicrobial resistance phenotype (disc test, Vitek, Phoenix)
  rather than a PCR gene-presence result — use the resistance-prediction validation
  workflow instead
- No reference PCR/phenotype table exists for the isolates, so concordance cannot
  be computed
- 'Samples are M. tuberculosis: the phac-nml mykrobe-parser workflow removes non-tuberculous
  mycobacteria detections and targets resistance genes, not S. aureus virulence gene/PVL
  concordance'
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2830
tools:
- name: Mykrobe predictor
  role: Runs the gene-presence/variant-calling prediction on raw sequence data and
    emits per-probe JSON results (--format ) used for the concordance comparison
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: Optional R script for parsing Mykrobe JSON output into tidy CSV; note it is
    written for M. tuberculosis resistance results, so S. aureus PVL calls typically
    require custom JSON parsing
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:b4925d6cb37ecc6bec821a77b8281d1683b189146e2a34f64477fbb96546bd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# pcr-concordance-validation

## Summary

Validates Mykrobe predictor's in-silico gene-presence calls (e.g., PVL lukPV-S/lukPV-K) against reference PCR phenotypes by running Mykrobe with --format  on clinical isolate read sets and computing concordance. Provides a template for comparing sequence-based predictions to a molecular gold standard.

## When to use

Use when sequence-based variant or gene-presence calls (e.g., Mykrobe PVL or resistance calls) must be checked for concordance against an existing PCR or phenotypic reference table on a defined isolate panel.

## When NOT to use

- The comparison target is antimicrobial resistance phenotype (disc test, Vitek, Phoenix) rather than a PCR gene-presence result — use the resistance-prediction validation workflow instead
- No reference PCR/phenotype table exists for the isolates, so concordance cannot be computed
- Samples are M. tuberculosis: the phac-nml mykrobe-parser workflow removes non-tuberculous mycobacteria detections and targets resistance genes, not S. aureus virulence gene/PVL concordance

## Inputs

- FASTQ read sets for each clinical isolate (e.g., 67 S. aureus outbreak isolates from Supplementary Data 5, downloadable from ENA/NCBI SRA)
- Reference PCR phenotype table mapping each isolate to PVL-positive or PVL-negative status (23 negative, 44 positive)

## Outputs

- pvl_concordance.csv containing per-isolate PVL calls with probe depth and percentage coverage
- Concordance summary: true positive, true negative, and discordant call counts (expected 44 TP, 23 TN, 0 discordant)

## How to apply

Retrieve the isolate read sets (here, 67 S. aureus outbreak isolates from Supplementary Data 5 via ENA/NCBI SRA) and run Mykrobe predictor on each with --format  so per-probe results are machine-readable. Parse each JSON to extract the gene-presence call (lukPV-S/lukPV-K for PVL) together with supporting probe depth and percentage coverage. Join the called status to the reference PCR phenotype table (23 PVL-negative, 44 PVL-positive in this panel) and compute true positives, true negatives, and discordant calls. Validation passes only when in-silico calls agree with PCR status for all isolates — here 100% concordance means 44 correct positives and 23 correct negatives. Write per-isolate calls plus the concordance summary to a tabular output such as pvl_concordance.csv.

## Related tools

- **Mykrobe predictor** (Runs the gene-presence/variant-calling prediction on raw sequence data and emits per-probe JSON results (--format ) used for the concordance comparison) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (Optional R script for parsing Mykrobe JSON output into tidy CSV; note it is written for M. tuberculosis resistance results, so S. aureus PVL calls typically require custom JSON parsing) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample isolate_01 --species staph --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- In-silico PVL calls agree with PCR reference status for all 67 isolates (100% concordance)
- Concordance counts decompose to exactly 44 correct positive calls and 23 correct negative calls with 0 discordant
- Each call is supported by non-trivial probe depth and percentage coverage in the parsed JSON output
- pvl_concordance.csv contains one row per isolate (67 rows) with both the called and reference PVL status

## Limitations

- Mykrobe Parser only works with JSON files and is built for M. tuberculosis resistance panels (201901/202010); if a gene prediction fails, it sets all '[antimicrobial]_prediction' columns to 'failed' and genetic regions to 'NA'
- Concordance is only as good as the PCR reference; the paper notes phenotyping data can be imperfect and resides only in Supplementary Data 1–11
- High concordance on a single 67-isolate outbreak panel may not generalize to diverse populations or novel variants absent from the probe set

## Evidence

- [results] Mykrobe predictor's PVL gene tests were 100% concordant with PCR tests across the 67 outbreak isolates, comprising 23 PVL-negative and 44 PVL-positive samples.: [evidence span withheld — anchor status: not_found]
- [other] Run Mykrobe predictor on each isolate, requesting --format so that per-probe results (including PVL probes) are emitted as machine-readable JSON.: [evidence span withheld — anchor status: not_found]
- [readme] Stores results a . file instead of a text file. Mykrobe Parser only works with  files.: "--format 
        * Stores results a . file instead of a text file. Mykrobe Parser only works with  files."
- [readme] If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA".: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
- [discussion] All the sequencing data is available at the European Nucleotide Archive and National Center for Biotechnology Information's SRA: "All the sequencing data is available at the European Nucleotide Archive and National Center for Biotechnology Information's SRA"
