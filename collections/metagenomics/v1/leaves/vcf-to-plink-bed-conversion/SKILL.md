---
name: vcf-to-plink-bed-conversion
description: Use when downstream population-genomic tools require PLINK binary (BED)
  input but the available genotype data are VCF files split by chromosome and indexed
  with tabix, e.g. before running ADMIXTURE in scalepopgen.
when_to_use_negative:
- Input is already a PLINK binary (BED/BIM/FAM) dataset — specify the files in a plink_input.csv
  sheet with header 'prefix,bed,bim,fam' instead of converting from VCF.
- Data are multi-allelic or non-SNP variants — scalepopgen is designed for biallelic
  SNP data stored in VCF or PLINK binary files.
edam_operation: http://edamontology.org/operation_1815
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2885
tools:
- name: PLINK
  role: Performs the VCF to binary PED (BED) file format conversion and emits the
    plink-generated binary files consumed by downstream analyses.
  repo: https://github.com/Popgen48/scalepopgen
- name: Nextflow
  role: Workflow manager that orchestrates the conversion step and its dependencies
    within the scalepopgen sub-workflows.
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:6b774704a862cfe6e0a4a8ce24846c16f557d891a0fb39ef0cee73fb503e1886
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - Nextflow
  - PLINK
  - Python
  - Vcftools
  edam_operation: http://edamontology.org/operation_1815
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# vcf-to-plink-bed-conversion

## Summary

Convert chromosome-split, tabix-indexed biallelic SNP VCF files into PLINK binary (BED/BIM/FAM) format within the scalepopgen Nextflow pipeline, producing the binary genotype input required by downstream tools such as ADMIXTURE and PCA.

## When to use

Use when downstream population-genomic tools require PLINK binary (BED) input but the available genotype data are VCF files split by chromosome and indexed with tabix, e.g. before running ADMIXTURE in scalepopgen.

## When NOT to use

- Input is already a PLINK binary (BED/BIM/FAM) dataset — specify the files in a plink_input.csv sheet with header 'prefix,bed,bim,fam' instead of converting from VCF.
- Data are multi-allelic or non-SNP variants — scalepopgen is designed for biallelic SNP data stored in VCF or PLINK binary files.

## Inputs

- Chromosome-split, tabix-indexed VCF files (.vcf.gz with .tbi) listed in a vcf_input.csv sheet with header 'chrom,vcf,vcf_idx'
- Sample map file (two tab-delimited columns: individual IDs and population IDs, extension '.map')
- YAML parameter file defining sub-workflow and tool options (generated via the scalepopgen CLI)

## Outputs

- PLINK binary genotype files (.bed, .bim, .fam)
- Converted dataset ready for downstream analyses (e.g., ADMIXTURE, PCA)

## How to apply

Prepare chromosome-split VCFs (.vcf.gz with .tbi indexes) listed in a comma-separated input sheet with header 'chrom,vcf,vcf_idx', ensuring chromosome names contain no punctuation marks. In scalepopgen, this conversion is performed with PLINK as part of the sub-workflow: after (optionally) LD-based filtering recommended before conversion, run the VCF-to-BED conversion step using plink, which emits .bed/.bim/.fam binary files. The filtered/converted dataset is then supplied to downstream analyses (e.g., running ADMIXTURE on the bed file). Validate that the resulting BED file has the same samples as the input sample map (two tab-delimited columns: individual IDs and population IDs, file ending in '.map') and that SNP counts match the filtered VCF.

## Related tools

- **PLINK** (Performs the VCF to binary PED (BED) file format conversion and emits the plink-generated binary files consumed by downstream analyses.) — https://github.com/Popgen48/scalepopgen
- **Nextflow** (Workflow manager that orchestrates the conversion step and its dependencies within the scalepopgen sub-workflows.) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen/ -profile <docker/singularity/conda/mamba> -params-file <path/to/parameters.yml> -qs <maximum number of processes>
```

## Evaluation signals

- The three expected PLINK output files (.bed, .bim, .fam) are produced with sample count matching the input sample map.
- Chromosome names in the input CSV contain no punctuation marks (a stated requirement of the input format).
- The converted dataset is accepted by the downstream tool (e.g., ADMIXTURE runs successfully on the bed file).
- When run in the paper's case study, the full pipeline completed reproducibly in ~1.5 h (~136 CPU hours) with the converted inputs.

## Limitations

- All VCF files must be split by chromosome and indexed with tabix before conversion; unsplit or unindexed VCFs will not work.
- Chromosome names must not contain punctuation marks.
- Users must also supply a correctly formatted sample map (.map) with individual and population IDs; population-level outputs depend on it.

## Evidence

- [intro] conversion of VCF to binary PED bed (BED) file format using plink (Purcell et al. 2007): "conversion of VCF to binary PED bed (BED) file format using plink (Purcell et al. 2007)"
- [intro] filtering based on linkage disequilibrium (LD) (recommended): "(i) filtering based on linkage disequilibrium (LD) (recommended)"
- [readme] All VCF files need to be splitted by the chromosomes and indexed with tabix.: [evidence span withheld — anchor status: external_doc]
- [readme] Please note that the chromosome name must not contain any punctuation marks.: [evidence span withheld — anchor status: external_doc]
- [intro] run ADMIXTURE tool on the bed file: "(iii) run ADMIXTURE tool on the bed file"
