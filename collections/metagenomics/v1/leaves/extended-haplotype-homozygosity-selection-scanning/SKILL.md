---
name: extended-haplotype-homozygosity-selection-scanning
description: Use when scanning phased biallelic SNP VCF (or PLINK binary) data, split
  by chromosome and indexed with tabix, for single-population (iHS) and pairwise population-comparison
  (XP-EHH) haplotype-based signals of selection.
when_to_use_negative:
- Input VCFs are not biallelic SNP data split by chromosome and tabix-indexed (chromosome
  names must not contain punctuation marks), or only PLINK non-binary formats are
  available
- The analysis goal is genetic structure (PCA, ADMIXTURE with optimal K via CV error)
  or phylogeny (TreeMix), not signatures of selection — use the corresponding scalepopgen
  sub-workflows instead
- A quick single-tool run is desired without intermediate transformations; the scalepopgen
  framework exists because many tools require nonstandard input formats and multistep
  transformations
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: Nextflow
  role: Workflow management system orchestrating the selection sub-workflow and its
    dependencies
  repo: https://github.com/Popgen48/scalepopgen
- name: BEAGLE
  role: Phasing (v5.2) of the filtered VCF files as required before selscan runs
- name: SHAPEIT5
  role: Alternative phasing tool (v5.1.1) used before selscan
- name: selscan
  role: Computes iHS for each population and XP-EHH for each pair of populations on
    the phased haplotypes (v2.0.2)
- name: SweepFinder2
  role: Per-population composite likelihood ratio scan along the genome (v1.0.0),
    using est-sfs ancestral alleles when an outgroup is present
- name: est-sfs
  role: Optionally infers ancestral alleles (v2.04) when an outgroup is present in
    the VCF files, to prepare SweepFinder2 inputs
- name: Plotly
  role: Python-script plotting of interactive Manhattan plots per analysis (except
    XP-EHH) and per population, with Ensembl location links for regions above threshold
- name: Singularity
  role: Container profile used to run the sub-workflow reproducibly (as in the case
    study command)
- name: scalepopgen-cli
  role: Command-Line Interface that guides parameter specification and validates input
    formats to generate the YAML parameters file
  repo: https://github.com/Popgen48/scalepopgen-cli.git
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:14f0fc9160be0110a2f9877e0eedbd6506a0a35e945dea2e0f7b7c46553d51c2
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
  - scalepopgen-cli
  - selscan
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# Extended-haplotype-homozygosity selection scanning (iHS/XP-EHH) with scalepopgen

## Summary

Runs the scalepopgen 'signatures of selection' sub-workflow to detect genomic signatures of selection on biallelic SNP data by phasing chromosome-split VCFs (Beagle v5.2 or SHAPEIT5 v5.1.1) and computing iHS per population and XP-EHH per population-pair with selscan v2.0.2, plus SweepFinder2 CLR scans and interactive Manhattan plots.

## When to use

Use when scanning phased biallelic SNP VCF (or PLINK binary) data, split by chromosome and indexed with tabix, for single-population (iHS) and pairwise population-comparison (XP-EHH) haplotype-based signals of selection.

## When NOT to use

- Input VCFs are not biallelic SNP data split by chromosome and tabix-indexed (chromosome names must not contain punctuation marks), or only PLINK non-binary formats are available
- The analysis goal is genetic structure (PCA, ADMIXTURE with optimal K via CV error) or phylogeny (TreeMix), not signatures of selection — use the corresponding scalepopgen sub-workflows instead
- A quick single-tool run is desired without intermediate transformations; the scalepopgen framework exists because many tools require nonstandard input formats and multistep transformations

## Inputs

- Biallelic SNP VCF files split by chromosome and indexed with tabix, listed in a comma-separated vcf_input.csv sheet (chrom,vcf,vcf_idx columns)
- PLINK binary files (bed/bim/fam) listed in a .p.csv input sheet (prefix,bed,bim,fam)
- Tab-delimited sample map (.map) with individual IDs in column 1 and population IDs in column 2, no header
- YAML parameters file (e.g. test_full.yaml) specifying sub-workflow options, including the user-defined selection threshold

## Outputs

- Phased VCF files per chromosome produced by Beagle v5.2 or SHAPEIT5 v5.1.1
- selscan v2.0.2 iHS results per population and XP-EHH results per population-pair on phased haplotypes
- SweepFinder2 v1.0.0 per-population composite likelihood ratio results along the genome
- Interactive Manhattan plots (Plotly) for each analysis except XP-EHH and each population, with Ensembl location links for regions above the user-defined selection threshold

## How to apply

Launch the scalepopgen selection sub-workflow via Nextflow with a YAML parameters file (e.g. test_full.yaml) under a container profile such as singularity, as in the case study command 'nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume'. First phase the filtered, chromosome-split VCFs with Beagle v5.2 or SHAPEIT5 v5.1.1, since selscan requires phased haplotypes. Optionally infer ancestral alleles with est-sfs v2.04 when an outgroup is present in the VCFs, using that information to prepare SweepFinder2 input files. Then run selscan v2.0.2 to compute iHS for each population and XP-EHH for each possible pair of populations on the phased haplotypes, and SweepFinder2 v1.0.0 per population to compute the composite likelihood ratio (Kim and Stephan 2002) along the genome. Finally, collect the scan results and generate interactive Manhattan plots (Plotly) for each analysis except XP-EHH and each population, with Ensembl location links for regions above the user-defined selection threshold when species annotation is available in the Ensembl database.

## Related tools

- **Nextflow** (Workflow management system orchestrating the selection sub-workflow and its dependencies) — https://github.com/Popgen48/scalepopgen
- **BEAGLE** (Phasing (v5.2) of the filtered VCF files as required before selscan runs)
- **SHAPEIT5** (Alternative phasing tool (v5.1.1) used before selscan)
- **selscan** (Computes iHS for each population and XP-EHH for each pair of populations on the phased haplotypes (v2.0.2))
- **SweepFinder2** (Per-population composite likelihood ratio scan along the genome (v1.0.0), using est-sfs ancestral alleles when an outgroup is present)
- **est-sfs** (Optionally infers ancestral alleles (v2.04) when an outgroup is present in the VCF files, to prepare SweepFinder2 inputs)
- **Plotly** (Python-script plotting of interactive Manhattan plots per analysis (except XP-EHH) and per population, with Ensembl location links for regions above threshold)
- **Singularity** (Container profile used to run the sub-workflow reproducibly (as in the case study command))
- **scalepopgen-cli** (Command-Line Interface that guides parameter specification and validates input formats to generate the YAML parameters file) — https://github.com/Popgen48/scalepopgen-cli.git

## Examples

```
nextflow run scalepopgen/ -profile test_full,singularity -params-file test_full.yaml -qs 10
```

## Evaluation signals

- Phased VCF files are generated for every chromosome before any selscan run, confirming the Beagle/SHAPEIT5 step completed
- iHS result files exist for every population and XP-EHH result files exist for every possible population pair
- SweepFinder2 CLR result files are produced for each population
- Interactive Manhattan plots are generated for each analysis except XP-EHH and each population, with Ensembl location links attached to regions above the user-defined selection threshold
- Regions flagged as under selection in the plots correspond to the user-defined threshold applied to the selscan/SweepFinder2 scan statistics

## Limitations

- XP-EHH does not receive an interactive Manhattan plot — plots cover each analysis except XP-EHH
- selscan requires phased input, so the scan cannot run without a successful Beagle or SHAPEIT5 phasing step
- Ancestral-allele inference via est-sfs for SweepFinder2 is only possible when an outgroup is present in the VCF files
- Chromosome names in the VCF inputs must not contain punctuation marks, and all VCFs must be split by chromosome and tabix-indexed
- Ensembl location links for candidate regions are only available when species annotation resources are present in the Ensembl database

## Evidence

- [abstract] Single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection: "(iv) implementing single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection"
- [methods] Run selscan v2.0.2 to compute iHS for each population and XP-EHH for each possible pair of populations on the phased haplotypes.: [evidence span withheld — anchor status: not_found]
- [methods] Phase the filtered VCF files with Beagle v5.2 or SHAPEIT5 v5.1.1, as required before selscan runs.: [evidence span withheld — anchor status: not_found]
- [methods] produce interactive Manhattan plots with the workflow's Python scripts (Plotly) for each analysis except XP-EHH and each population, with Ensembl location links for regions above the user-defined selection threshold when species annotation is available: [evidence span withheld — anchor status: not_found]
- [readme] The sub-workflows can be used separately or in combination with each other.: "The sub-workflows can be used separately or in combination with each other."
