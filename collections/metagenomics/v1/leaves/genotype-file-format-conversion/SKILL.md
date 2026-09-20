---
name: genotype-file-format-conversion
description: Use when input genotype data are VCF files (chromosome-split and tabix-indexed)
  but the target analysis requires PLINK binary BED format, or when chromosome IDs
  must be recoded to integers for tools like ADMIXTURE and Eigensoft smartpca.
when_to_use_negative:
- Input is already a PLINK binary (bed/bim/fam) set listed in a .p.csv sheet — no
  VCF-to-BED conversion is needed
- Data are not biallelic SNPs; scalepopgen is scoped to biallelic SNP data in VCF
  or PLINK binary format
- Chromosome names contain punctuation marks — fix the VCF naming first since the
  pipeline requires chromosome names without punctuation
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: PLINK
  role: performs VCF to binary PED (BED) format conversion and recodes non-integer
    chromosome IDs to integers
  repo: https://github.com/Popgen48/scalepopgen
- name: Nextflow
  role: orchestrates the conversion as the first step of the scalepopgen workflow
    before filtering and structure analyses
  repo: https://github.com/Popgen48/scalepopgen
- name: tabix
  role: indexes chromosome-split VCF files prior to conversion and downstream processing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:4df7b32bae0c22f02a54ac94b1da27856c4431f7619461a15e2a50f0b4937672
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Bokeh
  - Eigensoft
  - Nextflow
  - PLINK
  - PONG
  - Python
  - tabix
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Genotype file format conversion

## Summary

Converts biallelic SNP genotype data between VCF and PLINK binary (BED/BIM/FAM) formats so downstream population-genomic tools (ADMIXTURE, smartpca) that require nonstandard formats can consume the data.

## When to use

Use when input genotype data are VCF files (chromosome-split and tabix-indexed) but the target analysis requires PLINK binary BED format, or when chromosome IDs must be recoded to integers for tools like ADMIXTURE and Eigensoft smartpca.

## When NOT to use

- Input is already a PLINK binary (bed/bim/fam) set listed in a .p.csv sheet — no VCF-to-BED conversion is needed
- Data are not biallelic SNPs; scalepopgen is scoped to biallelic SNP data in VCF or PLINK binary format
- Chromosome names contain punctuation marks — fix the VCF naming first since the pipeline requires chromosome names without punctuation

## Inputs

- VCF files split by chromosome and indexed with tabix (.vcf.gz + .tbi), listed in a vcf_input.csv sheet (chrom,vcf,vcf_idx header)
- PLINK binary files (bed/bim/fam) listed in a .p.csv sheet (prefix,bed,bim,fam header)
- Tab-delimited sample map file ending in '.map' with individual IDs and population IDs, no header

## Outputs

- PLINK binary BED/BIM/FAM genotype files (biallelic SNP data)
- Genotype files with integer-recoded chromosome IDs suitable for ADMIXTURE and smartpca

## How to apply

First stage the raw data: split VCFs by chromosome and index with tabix, then list them in a comma-separated sheet (vcf_input.csv with chrom,vcf,vcf_idx columns), ensuring chromosome names contain no punctuation marks. If starting from PLINK data, list the prefix,bed,bim,fam in a .p.csv sheet and provide a tab-delimited, header-less sample map mapping individual IDs to population IDs. Convert VCF to binary PED (BED) format with plink (Purcell et al. 2007), which is the recommended intermediate format for structure analyses. If chromosome IDs are non-integer, recode them to integers (e.g. with PLINK) because tools expecting integer chromosome IDs such as ADMIXTURE (v 7.0.2) and Eigensoft smartpca (v 8.0.0) cannot read non-integer IDs. In scalepopgen this conversion is an upstream step before filtering, LD pruning (plink2, user-defined parameters), PCA, and ADMIXTURE.

## Related tools

- **PLINK** (performs VCF to binary PED (BED) format conversion and recodes non-integer chromosome IDs to integers) — https://github.com/Popgen48/scalepopgen
- **Nextflow** (orchestrates the conversion as the first step of the scalepopgen workflow before filtering and structure analyses) — https://github.com/Popgen48/scalepopgen
- **tabix** (indexes chromosome-split VCF files prior to conversion and downstream processing)

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs 10
```

## Evaluation signals

- The output directory contains the complete PLINK binary trio (.bed, .bim, .fam) with the prefix specified in the .p.csv input sheet
- All chromosome IDs in the converted .bim file are integers, so ADMIXTURE and smartpca can read the data
- Sample count in the .fam file matches the individual IDs in the sample.map file, and population assignments are consistent
- Downstream LD pruning (plink2, user-defined parameters) runs without format errors, confirming the converted files are valid input

## Limitations

- VCF files must be pre-split by chromosome and indexed with tabix before the pipeline can consume them
- Chromosome names must not contain punctuation marks; non-integer chromosome IDs additionally require recoding before structure analyses
- The pipeline handles only biallelic SNP data — other variant types are out of scope

## Evidence

- [intro] VCF to binary PED (BED) format conversion using plink: "conversion of VCF to binary PED bed (BED) file format using plink (Purcell et al. 2007)"
- [other] Non-integer chromosome IDs must be recoded: [evidence span withheld — anchor status: not_found]
- [readme] VCF input sheet requirements: [evidence span withheld — anchor status: external_doc]
- [readme] Chromosome name restriction: [evidence span withheld — anchor status: external_doc]
- [abstract] Input format constraint of the pipeline: [evidence span withheld — anchor status: not_found]
