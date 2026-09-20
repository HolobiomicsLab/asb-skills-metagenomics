---
name: antimicrobial-resistance-prediction
description: Use when raw reads (Illumina MiSeq/HiSeq or ONT MinION FASTQ) from S.
  aureus or M. tuberculosis isolates need antibiotic-resistance calls across a drug
  panel, without assembly or phenotype testing.
when_to_use_negative:
- Species or resistance mechanisms are poorly characterized outside S. aureus, M.
  tuberculosis, Shigella, or Salmonella — large training sets with WGS and phenotype
  data are required to build panels
- You need pyrazinamide resistance prediction in M. tuberculosis — known mutations
  are poorly predictive and the drug is excluded
- You require direct-from-sputum M. tuberculosis mixture analysis — the current model
  only handles major/minor clones, not a general mixture model
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_2830
tools:
- name: Mykrobe predictor
  role: Core de Bruijn graph genotype caller that takes raw sequence data and predicts
    species ID and antibiotic resistance
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe JSON output into tidy, per-genetic-region AMR
    and speciation CSV reports
  repo: https://github.com/phac-nml/mykrobe-parser
- name: Illumina MiSeq
  role: Sequencing platform producing the read data input (e.g. 368x mean depth reference
    profiles)
- name: Oxford Nanopore MinION
  role: Single-molecule sequencing platform validated as proof-of-principle input
    for resistance calls
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:25e58c0be72c7d448f7b0e5c511ea31a1726397319e476efac2dca3766f428af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Illumina MiSeq
  - Mykrobe Parser
  - Mykrobe predictor
  - Oxford Nanopore MinION
  - Phoenix automated microbiology system (BD Biosciences)
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# antimicrobial-resistance-prediction

## Summary

Predict antibiotic resistance directly from raw bacterial sequence data using Mykrobe predictor's de Bruijn graph genotype caller, which maps known resistance-causing mutations and presence/absence genes against panels for S. aureus and M. tuberculosis and emits clinician-friendly resistance calls within ~3 minutes.

## When to use

Use when raw reads (Illumina MiSeq/HiSeq or ONT MinION FASTQ) from S. aureus or M. tuberculosis isolates need antibiotic-resistance calls across a drug panel, without assembly or phenotype testing.

## When NOT to use

- Species or resistance mechanisms are poorly characterized outside S. aureus, M. tuberculosis, Shigella, or Salmonella — large training sets with WGS and phenotype data are required to build panels
- You need pyrazinamide resistance prediction in M. tuberculosis — known mutations are poorly predictive and the drug is excluded
- You require direct-from-sputum M. tuberculosis mixture analysis — the current model only handles major/minor clones, not a general mixture model

## Inputs

- Raw sequencing reads (FASTQ) from S. aureus or M. tuberculosis isolates (Illumina MiSeq/HiSeq 2000 or Oxford Nanopore MinION)
- Mykrobe resistance panel selection (e.g. '201901' or '202010')
- Optional: phenotypic reference calls (BSAC disc test, Phoenix, Vitek) for validation

## Outputs

- Per-sample JSON file of species ID and antibiotic-resistance calls
- Clinician-friendly resistance report generated within ~3 minutes on a laptop
- When using Mykrobe Parser: output-report.csv (per-drug predictions and mutations by genetic region, tidy format) and output-speciation_data.csv (phylo_group, species, lineage with depths and coverage)

## How to apply

Run Mykrobe predictor directly on raw read sets (e.g. Illumina at 368x mean depth, or ONT MinION 2D reads at 39x with a 10% error model and base qualities ignored) using its de Bruijn graph representation of bacterial diversity to identify species and resistance profile. For results compatible with downstream parsing, invoke with '--format json' and a resistance panel such as '--panel 201901' (or 202010), producing per-sample JSON resistance calls. Optionally post-process with Mykrobe Parser: create its Conda environment ('conda env create -f mykrobe_parser_installation.yml') and run 'Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe_json_files' to separate mutations into individual genetic regions in tidy output-report.csv. Judge calls against phenotypic references (BSAC disc test, Phoenix, Vitek) by computing per-drug FN/FP counts and aggregating sensitivity/specificity; for S. aureus expect ~99.1%/99.6% across 12 antibiotics, while M. tuberculosis sensitivity is lower (82.6%) because known mutations are poorly predictive (pyrazinamide excluded).

## Related tools

- **Mykrobe predictor** (Core de Bruijn graph genotype caller that takes raw sequence data and predicts species ID and antibiotic resistance) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe JSON output into tidy, per-genetic-region AMR and speciation CSV reports) — https://github.com/phac-nml/mykrobe-parser
- **Illumina MiSeq** (Sequencing platform producing the read data input (e.g. 368x mean depth reference profiles))
- **Oxford Nanopore MinION** (Single-molecule sequencing platform validated as proof-of-principle input for resistance calls)

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out.json --format json --seq reads.fq.gz
```

## Evaluation signals

- S. aureus validation on St_B1 reproduces sensitivity/specificity of 99.1%/99.6% across 12 antibiotics, with methicillin FPR 0.0%, penicillin FPR 11.7%, and ciprofloxacin FNR 4.6%
- MinION vs MiSeq concordance table shows 12/12 identical resistance calls for the same isolate, with gyrA S84L called as the ciprofloxacin determinant in both and no false-positive resistance SNPs
- M. tuberculosis validation (n=1,609) yields sensitivity/specificity of 82.6%/98.5%, matching line-probe assay performance
- Mykrobe Parser output-report.csv contains per-drug '[antimicrobial]_prediction' columns with mutations separated into individual genetic regions; failed gene predictions are marked 'failed' with regions set to 'NA'
- Contaminated samples (>40 confident heterozygous SNPs) are excluded before evaluating error rates

## Limitations

- M. tuberculosis sensitivity is low (82.6%) vs traditional DST and completely excludes pyrazinamide since known mutations are poorly predictive
- MinION result is an N=1 proof-of-principle only
- Mykrobe Parser is compatible only with Mykrobe v0.7.0 and the 201901/202010 panels, and requires --format json output
- Samples detected as non-tuberculous mycobacteria are removed for quality control; contaminated samples (>40 heterozygous SNPs) must be excluded
- Number of resistant-phenotype samples was limiting; false-negative rates were only estimated where >10 resistant samples existed in the validation set

## Evidence

- [abstract] de Bruijn graph genotyping basis: "Here we show how de Bruijn graph representation of bacterial diversity can be used to identify species and resistance profiles of clinical isolates"
- [abstract] runtime and software: "We implement this method for Staphylococcus aureus and Mycobacterium tuberculosis in a software package ('Mykrobe predictor') that takes raw sequence data as input"
- [abstract] S. aureus error rates: "For S. aureus, the error rates of our method are comparable to gold-standard phenotypic methods, with sensitivity/specificity of 99.1%/99.6% across 12 antibiotics"
- [abstract] M. tuberculosis error rates: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [readme] parser installation: "conda env create -f mykrobe_parser_installation.yml"
- [readme] parser invocation: "Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe_json_files"
- [readme] JSON format requirement: "Stores results a .json file instead of a text file. Mykrobe Parser only works with json files."
- [readme] failed predictions: "If prediction for one gene fails in Mykrobe, Mykrobe parser will set all "[antimicrobial]_prediction" columns to "failed" and all genetic regions to "NA"."
