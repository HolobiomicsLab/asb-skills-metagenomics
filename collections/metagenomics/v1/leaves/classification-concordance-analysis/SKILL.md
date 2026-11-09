---
name: classification-concordance-analysis
description: Use when raw mycobacterial sequencing reads must be classified as MTBC
  vs non-tuberculous mycobacteria and lineage-typed, and the accuracy of those calls
  (vs KvarQ or expected labels) must be quantified.
when_to_use_negative:
- The goal is antibiotic-resistance prediction rather than species/lineage calling
  — Mykrobe's resistance output (sensitivity 82.6%, specificity 98.5% for M. tuberculosis)
  is a separate task, and pyrazinamide predictions are excluded due to poorly predictive
  known mutations
- Input is already a parsed species/lineage table (output-jsondata.csv) — proceed
  directly to cross-tabulation rather than re-running Mykrobe
- Samples are S. aureus/CoNS (St_A1, St_A2, St_B1, St_B2) rather than mycobacteria
  — the tb species/lineage panel and tb-specific parser do not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mykrobe predictor
  role: Classifies raw reads into phylo_group, species, and lineage using the 202010
    probe panel via de Bruijn graph kmer calls; run with --format json and default
    --min_depth 5 / --min_variant_conf 10
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe's M. tuberculosis-specific JSON results and emits
    output-jsondata.csv containing speciation data (phylo_group, species, lineage
    with depths and percent coverage); compatible with Mykrobe v0.7.0 and the 201901/202010
    panels
  repo: https://github.com/phac-nml/mykrobe-parser.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:4f6b69e131513c9dbdbaa687e0f8ff854f7e49dcb17bf1e65db54a45b333441a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# classification-concordance-analysis

## Summary

Evaluates how accurately a genotype-based species/lineage classification panel (Mykrobe predictor's species/lineage probe set, de Bruijn-graph kmer calling) assigns phylo_group, species, and lineage to mycobacterial isolates, and quantifies agreement with an independent caller (KvarQ lineage calls).

## When to use

Use when raw mycobacterial sequencing reads must be classified as MTBC vs non-tuberculous mycobacteria and lineage-typed, and the accuracy of those calls (vs KvarQ or expected labels) must be quantified.

## When NOT to use

- The goal is antibiotic-resistance prediction rather than species/lineage calling — Mykrobe's resistance output (sensitivity 82.6%, specificity 98.5% for M. tuberculosis) is a separate task, and pyrazinamide predictions are excluded due to poorly predictive known mutations
- Input is already a parsed species/lineage table (output-jsondata.csv) — proceed directly to cross-tabulation rather than re-running Mykrobe
- Samples are S. aureus/CoNS (St_A1, St_A2, St_B1, St_B2) rather than mycobacteria — the tb species/lineage panel and tb-specific parser do not apply

## Inputs

- Raw sequencing reads (FASTQ) for MTBC_A2 (1,157 MTBC isolates) and Myco_Retro (147 mycobacterial isolates) from ENA/NCBI SRA
- Precomputed KvarQ lineage calls for MTBC_A2 isolates (independent reference classifications)
- Expected species labels: MTBC for MTBC_A2, non-tuberculous mycobacteria for Myco_Retro

## Outputs

- Per-isolate table of phylo_group, species, and lineage calls with depths and percent coverage (output-jsondata.csv / species_lineage_concordance.csv)
- Concordance summary: MTBC/NTM misclassification count (expected 0) and lineage concordance rate vs KvarQ (expected 100% on MTBC_A2)

## How to apply

Run Mykrobe predictor on each sample in JSON format with the 202010 species/lineage probe panel and default QC thresholds (--min_depth 5, --min_variant_conf 10), then parse the JSON outputs to extract each sample's phylo_group, species, and lineage calls with their associated depths and percent coverage. Cross-tabulate Mykrobe's species calls against expected labels (MTBC for MTBC_A2, non-tuberculous mycobacteria for Myco_Retro) and count MTBC/NTM misclassifications; samples detected as non-tuberculous mycobacteria are removed for quality control purposes. Compare Mykrobe lineage assignments on MTBC_A2 against precomputed KvarQ lineage calls and compute the concordance rate. Write the per-isolate call table and concordance summary (0 misclassifications, 100% lineage concordance) to species_lineage_concordance.csv. Mykrobe Parser (R, phac-nml/mykrobe-parser) can be used downstream to separate mutations into individual genetic regions and present results in a tidy data format, emitting output-jsondata.csv with speciation columns.

## Related tools

- **Mykrobe predictor** (Classifies raw reads into phylo_group, species, and lineage using the 202010 probe panel via de Bruijn graph kmer calls; run with --format json and default --min_depth 5 / --min_variant_conf 10) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe's M. tuberculosis-specific JSON results and emits output-jsondata.csv containing speciation data (phylo_group, species, lineage with depths and percent coverage); compatible with Mykrobe v0.7.0 and the 201901/202010 panels) — https://github.com/phac-nml/mykrobe-parser.git

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- On the MTBC_A2 + Myco_Retro union, count of MTBC/NTM misclassifications must be 0 (only a handful of samples unresolved to species level within MTBC is acceptable)
- Mykrobe lineage calls on MTBC_A2 must be 100% concordant with precomputed KvarQ lineage calls
- Run parameters recorded: --panel 202010, --min_depth 5, --min_variant_conf 10 (defaults per Mykrobe Parser compatibility)
- Mykrobe run must be in JSON format (--format json) since Mykrobe Parser only works with json files
- species_lineage_concordance.csv exists with per-isolate phylo_group/species/lineage calls plus depth and percent-coverage columns

## Limitations

- Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes, so NTM detection doubles as a QC filter rather than a reported call in the parser output
- If prediction for one gene fails in Mykrobe, Mykrobe parser sets all '[antimicrobial]_prediction' columns to 'failed' and all genetic regions to 'NA'; a handful of MTBC samples remained unresolved to species level
- Parser is compatible only with Mykrobe v0.7.0 and the 201901/202010 panels; CSV output of Mykrobe results loses information that does not fit tabular format

## Evidence

- [abstract] de Bruijn graph representation of bacterial diversity can be used to identify species and resistance profiles of clinical isolates: "Here we show how de Bruijn graph representation of bacterial diversity can be used to identify species and resistance profiles of clinical isolates"
- [discussion] Myc_Retro validation set description: "the validation set for mycobacteria included the set Myc_Retro consisting of all mycobacterial (meaning positive MGIT culture) samples sent to the laboratories"
- [readme] Mykrobe Parser speciation output columns: "Stores Mykrobe's speciation data such as "phylo_group", "species", and "lineage" and their associated depths and percentages of coverage."
- [readme] JSON format requirement: "* Stores results a .json file instead of a text file. Mykrobe Parser only works with json files."
- [readme] NTM QC removal: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
