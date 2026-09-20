---
name: composite-likelihood-ratio-sweep-detection
description: Use when phased, filtered biallelic SNP VCFs split by chromosome are
  available and you need single-population selective-sweep scans that complement haplotype-based
  iHS/XP-EHH analyses.
when_to_use_negative:
- The goal is pair-wise population-comparison-based sweep detection (XP-EHH) rather
  than single-population scans — XP-EHH is computed with selscan instead
- Data are unphased multiallelic variants — the sub-workflow requires biallelic SNPs
  phased with Beagle/SHAPEIT5 before the selection scans
- No population structure/mapping is available — SweepFinder2 is run per population,
  so a sample.map of individual-to-population assignments is required
edam_operation: http://edamontology.org/operation_0486
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3056
tools:
- name: SweepFinder2
  role: Computes the composite likelihood ratio (Kim and Stephan 2002) along the genome
    for each population on prepared SweepFinder2 input files
- name: est-sfs
  role: Optionally infers ancestral alleles (v2.04) when an outgroup is present, used
    to prepare SweepFinder2 input files
- name: BEAGLE
  role: Phases filtered VCF files (v5.2) as a prerequisite for the selection scans
- name: SHAPEIT5
  role: Alternative phasing tool (v5.1.1) required before selscan runs
- name: Nextflow
  role: Workflow manager that launches and coordinates the signatures-of-selection
    sub-workflow with a YAML parameters file
  repo: https://github.com/Popgen48/scalepopgen
- name: Python
  role: Plotly-based scripts that produce interactive Manhattan plots of CLR scan
    results per population
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:d3378c0cb72a471c13b886fa6d14b3fa05a4c54a2c482918c2ba748b1cd561af
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
  edam_operation: http://edamontology.org/operation_0486
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3056
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# composite-likelihood-ratio-sweep-detection

## Summary

Run SweepFinder2 v1.0.0 per population within the scalepopgen signatures-of-selection sub-workflow to compute the composite likelihood ratio (Kim and Stephan 2002) along the genome, detecting recent selective sweeps from biallelic SNP data.

## When to use

Use when phased, filtered biallelic SNP VCFs split by chromosome are available and you need single-population selective-sweep scans that complement haplotype-based iHS/XP-EHH analyses.

## When NOT to use

- The goal is pair-wise population-comparison-based sweep detection (XP-EHH) rather than single-population scans — XP-EHH is computed with selscan instead
- Data are unphased multiallelic variants — the sub-workflow requires biallelic SNPs phased with Beagle/SHAPEIT5 before the selection scans
- No population structure/mapping is available — SweepFinder2 is run per population, so a sample.map of individual-to-population assignments is required

## Inputs

- Phased biallelic SNP VCF files, split by chromosome and tabix-indexed (listed in vcf_input.csv with chrom,vcf,vcf_idx header)
- Tab-delimited sample map (.map) of individual IDs and population IDs without a header line
- YAML parameters file specifying sub-workflow options (e.g. test_full.yaml)
- Optionally: VCF including an outgroup sample for est-sfs ancestral allele inference used to prepare SweepFinder2 inputs

## Outputs

- SweepFinder2 composite likelihood ratio (CLR) result files for each population
- Interactive Manhattan plots (Plotly) per population with Ensembl location links for regions above the user-defined selection threshold

## How to apply

Within the scalepopgen signatures-of-selection sub-workflow launched via Nextflow (e.g. 'nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume'), first filter the VCF by sample and site criteria and phase haplotypes with Beagle v5.2 or SHAPEIT5 v5.1.1, since SweepFinder2 runs downstream of phasing. Optionally infer ancestral alleles with est-sfs v2.04 when an outgroup is present in the VCF files, and use this information to prepare SweepFinder2 input files, since the CLR depends on an ancestral-state polarization. Then run SweepFinder2 v1.0.0 once for each population (populations defined in the tab-delimited sample.map file) to compute the composite likelihood ratio along the genome. Collect the per-population scan results and produce interactive Manhattan plots with the workflow's Plotly-based Python scripts, linking regions above the user-defined selection threshold to Ensembl locations when species annotation is available.

## Related tools

- **SweepFinder2** (Computes the composite likelihood ratio (Kim and Stephan 2002) along the genome for each population on prepared SweepFinder2 input files)
- **est-sfs** (Optionally infers ancestral alleles (v2.04) when an outgroup is present, used to prepare SweepFinder2 input files)
- **BEAGLE** (Phases filtered VCF files (v5.2) as a prerequisite for the selection scans)
- **SHAPEIT5** (Alternative phasing tool (v5.1.1) required before selscan runs)
- **Nextflow** (Workflow manager that launches and coordinates the signatures-of-selection sub-workflow with a YAML parameters file) — https://github.com/Popgen48/scalepopgen
- **Python** (Plotly-based scripts that produce interactive Manhattan plots of CLR scan results per population)

## Examples

```
nextflow run scalepopgen/ -params-file test_full.yaml -profile singularity -resume
```

## Evaluation signals

- SweepFinder2 CLR result files exist for each population defined in the sample.map
- Phased VCFs and, where an outgroup was available, est-sfs ancestral allele outputs were generated before the SweepFinder2 step
- Interactive Manhattan plots are produced for each single-population analysis and each population (XP-EHH is excluded from per-population plotting)
- Regions above the user-defined selection threshold are flagged and carry Ensembl location links when species annotation is available in the Ensembl database

## Limitations

- CLR computation requires phased biallelic SNP data; the workflow operates on biallelic SNP data only
- Ancestral allele inference with est-sfs is only possible when an outgroup is present in the VCF files; SweepFinder2 input preparation depends on this
- Ensembl-based annotation of candidate regions is only available if annotation resources for the species under investigation are present in the Ensembl database

## Evidence

- [methods] CLR per population: [evidence span withheld — anchor status: not_found]
- [methods] ancestral alleles: [evidence span withheld — anchor status: not_found]
- [methods] Manhattan plots: [evidence span withheld — anchor status: not_found]
- [methods] Ensembl annotation: "if the annotation resources for the species under investigation are present in the Ensembl database"
- [methods] phasing prerequisite: [evidence span withheld — anchor status: not_found]
- [abstract] pipeline scope: "(iv) implementing single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection"
