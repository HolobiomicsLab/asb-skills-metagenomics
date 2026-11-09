---
name: haplotype-phasing-snp-vcf
description: Use when downstream EHH-based scans (iHS, XP-EHH, SweepFinder2 inputs)
  require phased haplotypes but the input is unphased chromosome-split VCF files from
  biallelic SNP data.
when_to_use_negative:
- Input data are already phased (e.g. the cattle Animal_SNPAtlas data set of phased
  and imputed autosomal SNP genotypes) and the scan tool accepts phased haplotypes
  directly
- Analysis goal is genetic structure or phylogeny (PCA, ADMIXTURE, TreeMix), which
  do not require haplotype phasing
- Data are multiallelic variants or structural variants rather than biallelic SNPs,
  which the scalepopgen workflows are not designed for
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: BEAGLE
  role: Phasing tool (v5.2) used to phase filtered per-chromosome VCFs before selscan
    runs
  repo: https://github.com/Popgen48/scalepopgen
- name: SHAPEIT5
  role: Alternative phasing tool (v5.1.1) for producing phased haplotypes
- name: selscan
  role: Downstream consumer of phased haplotypes; computes iHS per population and
    XP-EHH per population pair (v2.0.2)
- name: Nextflow
  role: Workflow manager that orchestrates the phasing step and resolves tool dependencies
    automatically
  repo: https://github.com/Popgen48/scalepopgen
- name: est-sfs
  role: Optional post-phasing step (v2.04) inferring ancestral alleles when an outgroup
    is present, for SweepFinder2 input preparation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:1d5a0998c3c012410bac8e699e22821efb4811a93f1417f5a670e0b0c785017c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Popgen48/scalepopgen
  tools:
  - BEAGLE
  - Conda
  - Nextflow
  - Plotly
  - Python
  - SHAPEIT5
  - Singularity
  - SweepFinder2
  - est-sfs
  - selscan
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# haplotype-phasing-snp-vcf

## Summary

Phase biallelic SNP VCFs (chromosome-split, tabix-indexed) with Beagle v5.2 or SHAPEIT5 v5.1.1 so that EHH-based selection scans such as selscan (iHS, XP-EHH) can be run in scalepopgen's signatures-of-selection sub-workflow.

## When to use

Use when downstream EHH-based scans (iHS, XP-EHH, SweepFinder2 inputs) require phased haplotypes but the input is unphased chromosome-split VCF files from biallelic SNP data.

## When NOT to use

- Input data are already phased (e.g. the cattle Animal_SNPAtlas data set of phased and imputed autosomal SNP genotypes) and the scan tool accepts phased haplotypes directly
- Analysis goal is genetic structure or phylogeny (PCA, ADMIXTURE, TreeMix), which do not require haplotype phasing
- Data are multiallelic variants or structural variants rather than biallelic SNPs, which the scalepopgen workflows are not designed for

## Inputs

- Biallelic SNP VCF files split by chromosome and tabix-indexed (.vcf.gz + .tbi)
- vcf_input.csv sheet with header chrom,vcf,vcf_idx
- Tab-delimited sample map (.map) of individual IDs to population IDs
- YAML parameters file specifying phasing tool and selection-scan options

## Outputs

- Phased per-chromosome VCF files suitable for selscan input
- Optional est-sfs ancestral-allele information for SweepFinder2 input preparation
- iHS results per population and XP-EHH results per population pair (downstream of phasing)
- Interactive Plotly Manhattan plots per analysis (except XP-EHH) and population

## How to apply

Split VCFs by chromosome, index with tabix, and list them in a comma-separated vcf_input.csv sheet (columns chrom,vcf,vcf_idx; chromosome names must contain no punctuation). Launch the scalepopgen signatures-of-selection sub-workflow with a YAML parameters file, e.g. 'nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume', so Nextflow (with Singularity/Conda/Docker) resolves all dependencies. Phase the filtered VCFs with Beagle v5.2 or SHAPEIT5 v5.1.1; phasing is required before selscan v2.0.2 runs. Optionally, when an outgroup is present, infer ancestral alleles with est-sfs v2.04 to prepare SweepFinder2 input files. Validate by checking that phased VCFs exist per chromosome and that downstream iHS/XP-EHH and SweepFinder2 result files plus Plotly Manhattan plots are generated.

## Related tools

- **BEAGLE** (Phasing tool (v5.2) used to phase filtered per-chromosome VCFs before selscan runs) — https://github.com/Popgen48/scalepopgen
- **SHAPEIT5** (Alternative phasing tool (v5.1.1) for producing phased haplotypes)
- **selscan** (Downstream consumer of phased haplotypes; computes iHS per population and XP-EHH per population pair (v2.0.2))
- **Nextflow** (Workflow manager that orchestrates the phasing step and resolves tool dependencies automatically) — https://github.com/Popgen48/scalepopgen
- **est-sfs** (Optional post-phasing step (v2.04) inferring ancestral alleles when an outgroup is present, for SweepFinder2 input preparation)

## Examples

```
nextflow run scalepopgen/ -profile singularity -params-file test_full.yaml -resume
```

## Evaluation signals

- Phased per-chromosome VCF files are produced for every entry in vcf_input.csv
- iHS result files exist for each population and XP-EHH files for each population pair, confirming phasing was accepted by selscan
- Interactive Plotly Manhattan plots are generated for each analysis (except XP-EHH) and each population
- Regions flagged as under selection match the user-defined selection threshold
- In the case study, the phased data set of ~25 million SNPs in 442 samples (30 breeds) reduced to 269 samples (17 breeds) and ~23.4 million SNPs after sample- and site-filtering before scanning

## Limitations

- VCF chromosome names must not contain punctuation marks and all VCFs must be chromosome-split and tabix-indexed before the workflow
- Ancestral-allele inference with est-sfs is only possible when an outgroup is present in the VCF files
- Phasing is not needed for XP-EHH visualization of Manhattan plots — the workflow produces interactive plots for each analysis except XP-EHH

## Evidence

- [methods] Phase the filtered VCF files with Beagle v5.2 or SHAPEIT5 v5.1.1, as required before selscan runs.: [evidence span withheld — anchor status: not_found]
- [readme] All VCF files need to be splitted by the chromosomes and indexed with tabix.: [evidence span withheld — anchor status: external_doc]
- [methods] Run selscan v2.0.2 to compute iHS for each population and XP-EHH for each possible pair of populations on the phased haplotypes.: [evidence span withheld — anchor status: not_found]
- [methods] Optionally infer ancestral alleles with est-sfs v2.04 when an outgroup is present in the VCF files.: [evidence span withheld — anchor status: not_found]
- [readme] Please note that the chromosome name must not contain any punctuation marks.: [evidence span withheld — anchor status: external_doc]
