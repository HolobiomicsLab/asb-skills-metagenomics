---
name: mycobacterial-drug-resistance-prediction
description: Use when raw FASTQ reads from MTBC isolates need rapid per-drug resistance
  calls (e.g. rifampicin, isoniazid, ethambutol) without assembling genomes or variant
  calling, or when benchmarking against phenotypic DST or KvarQ.
when_to_use_negative:
- The sample may contain non-tuberculous mycobacteria or mixed strains — such samples
  are removed for QC or flagged by the >40 heterozygous SNP contamination threshold,
  so predictions on them are invalid
- You need pyrazinamide predictions — known pyrazinamide mutations are poorly predictive
  and the drug is completely excluded from the panel
- The organism is not one of the supported species (M. tuberculosis, S. aureus, S.
  sonnei, S. typhi, S. enterica Paratyphi B); the tb panel will not apply
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2830
- http://edamontology.org/topic_3673
tools:
- name: Mykrobe predictor
  role: 'Core engine: predicts resistance and species directly from raw sequence reads
    via de Bruijn graph genotyping against a curated TB resistance panel; run with
    --species tb --format '
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses M. tuberculosis-specific Mykrobe JSON results into tidy
    per-genetic-region CSV reports and removes non-tuberculous mycobacteria samples
    for QC
  repo: https://github.com/phac-nml/mykrobe-parser
- name: KvarQ
  role: 'Comparator tool: reported per-drug first-line sensitivities of 90.8% (rifampicin),
    83.2% (isoniazid), 76.3% (ethambutol) against which Mykrobe calls are benchmarked'
- name: Cortex
  role: Independent whole-genome variant calling workflow (v1.0.5.21, k=31, ploidy=2)
    used for validation analyses rather than prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:8eb71092cb8b5db4a650464f35122d87f986b198cf53f628b1df42417c06b38f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Cortex
  - KvarQ
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2830
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mycobacterial-drug-resistance-prediction

## Summary

Predict M. tuberculosis antibiotic resistance directly from raw sequencing reads using Mykrobe predictor's de Bruijn graph approach, then parse the JSON output into per-drug resistant/susceptible calls with the PHAC Mykrobe Parser. On the n=1,609 MTBC_B validation set this yields overall sensitivity/specificity of 82.6%/98.5%.

## When to use

Use when raw FASTQ reads from MTBC isolates need rapid per-drug resistance calls (e.g. rifampicin, isoniazid, ethambutol) without assembling genomes or variant calling, or when benchmarking against phenotypic DST or KvarQ.

## When NOT to use

- The sample may contain non-tuberculous mycobacteria or mixed strains — such samples are removed for QC or flagged by the >40 heterozygous SNP contamination threshold, so predictions on them are invalid
- You need pyrazinamide predictions — known pyrazinamide mutations are poorly predictive and the drug is completely excluded from the panel
- The organism is not one of the supported species (M. tuberculosis, S. aureus, S. sonnei, S. typhi, S. enterica Paratyphi B); the tb panel will not apply

## Inputs

- Raw Illumina sequencing reads (FASTQ, optionally gzipped) per MTBC isolate, e.g. from ENA/NCBI SRA accessions listed in Supplementary Data
- Phenotypic drug-susceptibility testing (DST) resistance labels per isolate and drug (Supplementary Data 1-10)
- H37Rv reference context (GenBank NC000962.2) implicit in the resistance panel

## Outputs

- Per-isolate JSON resistance predictions (--format )
- output-report.csv with per-antimicrobial predictions and genetic regions (via Mykrobe Parser)
- output-speciation_data.csv with phylo_group, species and lineage (QC for non-tuberculous mycobacteria)
- Per-drug confusion matrices and validation metrics (e.g. validation_metrics.csv: overall 82.6%/98.5%; rifampicin 93.7%, isoniazid 84.3%, ethambutol 71.6% sensitivity)

## How to apply

Run `mykrobe predict --species tb --format  --seq reads.fq.gz` per isolate using the M. tuberculosis resistance panel (Mykrobe Parser is compatible with v0.7.0 and the '201901'/'202010' panels, e.g. `--panel 201901`). Apply quality control: remove samples in which Mykrobe detects non-tuberculous mycobacteria, and exclude samples contaminated by an unrelated strain using the threshold of 40 confident heterozygous SNPs. Parse each isolate's JSON output with Mykrobe Parser (Rscript mykrobe_parser.R -d <_dir>), which separates mutations into individual genetic regions, sets '[antimicrobial]_prediction' columns to 'failed' with genetic regions 'NA' for failed genes, and emits output-report.csv. Tabulate per-drug predictions against phenotypic DST labels from Supplementary Data to build confusion matrices and compute sensitivity/specificity; estimate false-negative rates only where >10 resistant samples exist. Rationale: the de Bruijn graph method avoids assembly and matches line-probe assay performance because its resistance panel was built from known mutations, which is why pyrazinamide is excluded and sensitivity (82.6%) trails gold-standard DST.

## Related tools

- **Mykrobe predictor** (Core engine: predicts resistance and species directly from raw sequence reads via de Bruijn graph genotyping against a curated TB resistance panel; run with --species tb --format ) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses M. tuberculosis-specific Mykrobe JSON results into tidy per-genetic-region CSV reports and removes non-tuberculous mycobacteria samples for QC) — https://github.com/phac-nml/mykrobe-parser
- **KvarQ** (Comparator tool: reported per-drug first-line sensitivities of 90.8% (rifampicin), 83.2% (isoniazid), 76.3% (ethambutol) against which Mykrobe calls are benchmarked)
- **Cortex** (Independent whole-genome variant calling workflow (v1.0.5.21, k=31, ploidy=2) used for validation analyses rather than prediction)

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --panel 201901 --seq reads.fq.gz; Rscript PATH/TO/R/mykrobe_parser.R -d PATH/TO/Mykrobe__files
```

## Evaluation signals

- Reproduced overall sensitivity/specificity on the n=1,609 MTBC_B validation set equals the reported 82.6%/98.5%
- Per-drug sensitivities match the reported values: rifampicin 93.7%, isoniazid 84.3%, ethambutol 71.6%
- Mykrobe is run with --format  (Mykrobe Parser only works with JSON files) and a compatible panel ('201901'/'202010') with Mykrobe v0.7.0
- QC is applied: samples flagged as non-tuberculous mycobacteria and samples with >40 confident heterozygous SNPs are excluded (20 removed from the validation set)
- Failed gene predictions appear as 'failed' in [antimicrobial]_prediction columns with genetic regions set to 'NA' rather than silent drops

## Limitations

- Sensitivity for M. tuberculosis is low (82.6%) compared with traditional DST because of limited understanding of underlying genetic mechanisms, and pyrazinamide is excluded since known mutations are poorly predictive
- Not enough M. tuberculosis data existed to estimate per-drug thresholds, so a single 10% threshold was used for all drugs; false-negative rates were only estimated where >10 resistant samples existed
- The mixture model only handles major/minor clones, which may be inadequate for direct-from-sputum M. tuberculosis analysis; the low-frequency minor allele finding needs replication in larger datasets
- Mykrobe Parser is version- and panel-locked: compatible only with Mykrobe v0.7.0 and the 201901/202010 panels; sample-level accession and phenotyping data are only in Supplementary Data 1-11

## Evidence

- [abstract] Mtb validation metrics: "For M. tuberculosis, our method predicts resistance with sensitivity/specificity of 82.6%/98.5% (independent validation set, n=1,609)"
- [readme] mykrobe predict JSON invocation: [evidence span withheld — anchor status: not_found]
- [readme] Mykrobe Parser QC and failure handling: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [methods] Mykrobe Parser JSON-only compatibility: "--format 
        * Stores results a . file instead of a text file. Mykrobe Parser only works with  files."
- [discussion] Contamination threshold: "We used this threshold of 40 heterozygous sites to determine whether a sample was contaminated by an unrelated strain"
- [discussion] Sensitivity limitation rationale: "sensitivity is lower here, probably because of limited understanding of the underlying genetic mechanisms"
