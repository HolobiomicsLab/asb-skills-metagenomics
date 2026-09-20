---
name: mykrobe-output-parsing
description: Use when you have raw Mykrobe predictor . outputs (e.g. from MTBC_A1
  or St_B1 isolates run with --format ) and need per-mutation, per-genetic-region
  tabular data for per-drug error or concordance tables.
when_to_use_negative:
- 'Mykrobe output is in text/CSV format rather than JSON — Mykrobe Parser only works
  with JSON files produced via --format '
- You need a unified parser for species beyond M. tuberculosis / S. aureus — the parser
  targets the Mycobacterium tuberculosis-specific results of Mykrobe
- You need non-tuberculous mycobacteria results retained — the parser removes such
  samples for quality control purposes
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mykrobe predictor
  role: Generates per-sample JSON resistance calls from raw reads with --format  and
    --panel 201901/202010; source of the files parsed by this skill
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe JSON results into output-report.csv and output-data.csv,
    separating mutations into individual genetic regions in tidy format
  repo: https://github.com/phac-nml/mykrobe-parser
- name: Phoenix automated microbiology system (BD Biosciences)
  role: Phenotypic reference method used to validate parsed predictions by flagging
    false negatives and false positives per antibiotic
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:90fc9329f41758e3adbd586d9f12306cbe431826387f9f1e776bd5e9244289e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  - Phoenix automated microbiology system (BD Biosciences)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# mykrobe--output-parsing

## Summary

Parses Mykrobe predictor JSON resistance-call outputs into a tidy per-sample CSV (output-report.csv) with mutations separated into individual genetic regions, using the phac-nml mykrobe-parser R script so downstream error-rate and genotype-concordance analyses can be computed.

## When to use

Use when you have raw Mykrobe predictor . outputs (e.g. from MTBC_A1 or St_B1 isolates run with --format ) and need per-mutation, per-genetic-region tabular data for per-drug error or concordance tables.

## When NOT to use

- Mykrobe output is in text/CSV format rather than JSON — Mykrobe Parser only works with JSON files produced via --format 
- You need a unified parser for species beyond M. tuberculosis / S. aureus — the parser targets the Mycobacterium tuberculosis-specific results of Mykrobe
- You need non-tuberculous mycobacteria results retained — the parser removes such samples for quality control purposes

## Inputs

- Mykrobe predictor JSON output files (.) produced with --format  and panel 201901 or 202010 (Mykrobe v0.7.0 compatible)
- Raw isolate FASTQ read sets (e.g. St_B1, MTBC_A1, Myc_Retro) used to generate the Mykrobe calls
- Phenotypic reference calls (BSAC disc test / Phoenix automated system) for downstream error-rate comparison

## Outputs

- output-report.csv: tidy antimicrobial resistance calls with mutations separated into individual genetic regions
- output-data.csv: speciation data (phylo_group, species, lineage with depths and coverage percentages)
- Downstream derived tables such as st_b1_per_drug_error_table.csv (per-drug FN/FP counts, sensitivity/specificity) or genotype_concordance.csv (per-site MAP genotype calls vs Mykrobe calls)

## How to apply

Run Mykrobe predictor on each isolate's read set with --format  and a compatible panel (201901 or 202010), since Mykrobe Parser only works with JSON files. Install the required R dependencies with 'conda env create -f mykrobe_parser_installation.yml', then execute 'Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe__files' (add -p 2020 for the 202010 panel). The parser emits output-report.csv with mutations split into individual genetic regions in tidy format, plus output-data.csv holding speciation data (phylo_group, species, lineage, depths, coverage percentages). Handle parser QC semantics: samples detected as non-tuberculous mycobacteria are removed, and a failed gene prediction sets all '[antimicrobial]_prediction' columns to 'failed' with genetic regions 'NA'. Optionally record provenance via -D (min_depth, default 5), -c (min_variant_conf, default 10), and -v/--version flags before comparing predictions against phenotypic references (BSAC disc test, Phoenix) or recomputing MAP genotype calls.

## Related tools

- **Mykrobe predictor** (Generates per-sample JSON resistance calls from raw reads with --format  and --panel 201901/202010; source of the files parsed by this skill) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe JSON results into output-report.csv and output-data.csv, separating mutations into individual genetic regions in tidy format) — https://github.com/phac-nml/mykrobe-parser
- **Phoenix automated microbiology system (BD Biosciences)** (Phenotypic reference method used to validate parsed predictions by flagging false negatives and false positives per antibiotic)

## Examples

```
Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe__files
```

## Evaluation signals

- output-report.csv exists and contains '[antimicrobial]_prediction' columns plus per-genetic-region mutation rows in tidy format
- Samples flagged by Mykrobe as non-tuberculous mycobacteria are absent from the parsed output (QC removal applied)
- Failed gene predictions appear as 'failed' in [antimicrobial]_prediction columns with genetic regions set to 'NA'
- On the S. aureus St_B1 validation set, aggregating per-drug FN/FP counts reproduces overall sensitivity/specificity of 99.1%/99.6% across 12 antibiotics (methicillin FPR 0.0%, penicillin FPR 11.7%, ciprofloxacin FNR 4.6%)
- Speciation columns (phylo_group, species, lineage with depths and coverage percentages) are populated in output-data.csv

## Limitations

- Mykrobe Parser is compatible only with Mykrobe v0.7.0 and the 201901 / 202010 panels; other panel versions are not supported
- Parser targets M. tuberculosis-specific Mykrobe results; applying it to other species output is out of scope
- If prediction for a gene fails in Mykrobe, the parser sets predictions to 'failed' and regions to 'NA' rather than imputing or re-calling
- Some Mykrobe results (notably for S. aureus workflows) do not fit tabular/CSV format and information is lost if CSV is requested instead of JSON

## Evidence

- [readme] Mykrobe Parser is compatible with [Mykrobe v0.7.0] and the "201901" / "202010" panels.: "Mykrobe Parser is compatible with [Mykrobe v0.7.0](https://anaconda.org/bioconda/mykrobe/files?version=0.7.0) and the "201901" / "202010" panels."
- [readme] It seperates mutations into indivudal genetic regions and presents them in a tidy data format compatible with reporting.: "It seperates mutations into indivudal genetic regions and presents them in a tidy data format compatible with reporting."
- [methods] Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes.: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [readme] If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed": "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
- [abstract] For S. aureus, the error rates of our method are comparable to gold-standard phenotypic methods, with sensitivity/specificity of 99.1%/99.6% across 12 antibiotics: "For S. aureus, the error rates of our method are comparable to gold-standard phenotypic methods, with sensitivity/specificity of 99.1%/99.6% across 12 antibiotics"
