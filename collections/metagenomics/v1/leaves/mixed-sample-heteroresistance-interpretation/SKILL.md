---
name: mixed-sample-heteroresistance-interpretation
description: Use when calling resistance in M. tuberculosis samples with minor resistant
  allele populations, especially for capreomycin, amikacin or ofloxacin where major-clone-only
  calls miss phenotypic resistance.
when_to_use_negative:
- The sample is a first-line drug call (e.g. pyrazinamide), which is excluded because
  known mutations are poorly predictive
- The data set is large enough to estimate per-drug allele-frequency thresholds —
  a uniform 10% threshold is a compromise forced by limited M. tuberculosis training
  data
- The goal is direct-from-sputum analysis requiring a general mixture model; the current
  implementation models only major/minor clones
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mykrobe predictor
  role: Predicts antimicrobial resistance from raw reads via de Bruijn graph representation;
    run twice with minor-allele detection disabled vs enabled (10% allele-frequency
    threshold) and with --format  output
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe's M. tuberculosis-specific JSON results into
    tidy CSV, marking failed gene predictions and removing samples where non-tuberculous
    mycobacteria are detected for quality control
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:0caec686d755e72361cb72d4693c761db57a178ab417626881391feb560bc058
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# mixed-sample-heteroresistance-interpretation

## Summary

Interprets Mykrobe predictor resistance calls in Mycobacterium tuberculosis samples where minor (low-frequency) resistant alleles indicate mixed populations or heteroresistance, which improves detection of extremely drug-resistant (XDR) strains for second-line drugs.

## When to use

Use when calling resistance in M. tuberculosis samples with minor resistant allele populations, especially for capreomycin, amikacin or ofloxacin where major-clone-only calls miss phenotypic resistance.

## When NOT to use

- The sample is a first-line drug call (e.g. pyrazinamide), which is excluded because known mutations are poorly predictive
- The data set is large enough to estimate per-drug allele-frequency thresholds — a uniform 10% threshold is a compromise forced by limited M. tuberculosis training data
- The goal is direct-from-sputum analysis requiring a general mixture model; the current implementation models only major/minor clones

## Inputs

- Raw M. tuberculosis sequencing reads (ENA/SRA FASTQ, e.g. Illumina HiSeq 2000, mean read length 87 bp after quality trimming)
- Phenotypic drug-susceptibility testing (DST) labels for second-line drugs from Supplementary Data
- H37Rv reference genome (GenBank NC000962.2) used by Mykrobe's panel

## Outputs

- Mykrobe predictor JSON resistance predictions per sample, under major-clone-only and minor-allele-enabled settings
- Per-drug sensitivity values for capreomycin, amikacin and ofloxacin under both settings
- minor_allele_sensitivity_deltas.csv tabulating baseline vs minor-allele sensitivity deltas (+9.1 pp, +11.8 pp, +23.0 pp)

## How to apply

Run Mykrobe predictor on raw sequencing reads twice: once with minor-allele calls disabled, then again with minor-allele calls enabled using a single 10% allele-frequency threshold applied uniformly to all drugs (per-drug thresholds could not be estimated for M. tuberculosis and phenotyping data was known to be imperfect). Emit predictions as JSON ('--format ') so they can be parsed programmatically, e.g. with Mykrobe Parser, which also removes samples in which non-tuberculous mycobacteria are detected for quality control. Compare each run's resistance calls against phenotypic DST labels, computing sensitivity per drug; on MTBC_B this raised sensitivity from 74.5 to 83.6% for capreomycin, 78.0 to 89.8% for amikacin and 38.5 to 61.5% for ofloxacin (N=13). Treat minor-allele findings as requiring replication with larger datasets, since only 18%/44% of training/validation samples were phenotyped for quinolones or injectables.

## Related tools

- **Mykrobe predictor** (Predicts antimicrobial resistance from raw reads via de Bruijn graph representation; run twice with minor-allele detection disabled vs enabled (10% allele-frequency threshold) and with --format  output) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe's M. tuberculosis-specific JSON results into tidy CSV, marking failed gene predictions and removing samples where non-tuberculous mycobacteria are detected for quality control) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- Baseline (major-clone-only) sensitivities reproduce at 74.5% (capreomycin), 78.0% (amikacin) and 38.5% (ofloxacin) on MTBC_B (N=13)
- Minor-allele-enabled sensitivities reproduce at 83.6%, 89.8% and 61.5%, with deltas +9.1 pp, +11.8 pp and +23.0 pp respectively
- Both runs emit JSON ('--format '); any sample flagged for non-tuberculous mycobacteria by Mykrobe is removed for quality control
- The single 10% allele-frequency threshold is applied uniformly to all drugs, not tuned per drug
- Predictions are compared against phenotypic DST labels with sensitivity = true positives / all phenotypically resistant samples

## Limitations

- Only 18%/44% of training/validation samples were phenotyped for at least one quinolone or injectable, so the minor-allele finding needs replication with larger data sets
- Not enough M. tuberculosis data to estimate per-drug thresholds; a single 10% threshold was selected for all drugs despite imperfect phenotyping
- The current implementation models only major/minor clones; a more general mixture model is suspected to be needed for direct-from-sputum M. tuberculosis analysis
- It remains an open question whether lower-frequency alleles have a bearing on patient outcome despite generally failing to cause in vitro resistance

## Evidence

- [abstract] Minor alleles improve detection of extremely drug-resistant strains: "We give evidence that minor alleles improve detection of extremely drug-resistant strains, and demonstrate feasibility of the use of emerging single-molecule"
- [discussion] Single 10% threshold rationale: "For M. tuberculosis, we did not have enough data to estimate per-drug thresholds, and we knew that the phenotyping data was imperfect"
- [discussion] Need for replication with larger datasets: "since only 18%/44% of our training/validation samples were phenotyped for at least one quinolone or injectable, this finding needs replication with larger data sets"
- [readme] QC removal of non-MTB samples by parser: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [methods] JSON output requirement: "Mykrobe Parser only works with  files."
