---
name: pairwise-population-xpehh-comparison
description: Use when detecting recent selective sweeps by contrasting haplotype lengths
  between two populations from chromosome-split, tabix-indexed biallelic VCFs plus
  a sample-to-population .map file.
when_to_use_negative:
- Input VCFs are not phased or not split/indexed per chromosome — phase with Beagle
  or SHAPEIT5 first, as selscan requires phased haplotypes
- Only a single population is present — XP-EHH is a between-population comparison;
  use a within-population scan such as iHS or SweepFinder2 instead
- Data are multi-allelic or non-SNP variants — scalepopgen's selection sub-workflow
  targets biallelic SNP data only
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3053
tools:
- name: selscan
  role: v2.0.2 EHH-based scan tool that computes XP-EHH for each possible pair of
    populations on the phased haplotypes
- name: BEAGLE
  role: v5.2 phasing tool preparing filtered VCFs for selscan XP-EHH input
- name: SHAPEIT5
  role: v5.1.1 alternative phasing tool before running selscan
- name: Nextflow
  role: orchestrates the signatures-of-selection sub-workflow including phasing, XP-EHH,
    and result collection
  repo: https://github.com/Popgen48/scalepopgen
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/molbev/msae057
    title: 'scalepopgen: Bioinformatic Workflow Resources Implemented in Nextflow
      for Comprehensive Population Genomic Analyses'
  source_content_hash: sha256:b870323ba682a8dda9e47d4b30da3f7284dccb54ceaa28899972913732033786
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
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Pairwise population XP-EHH comparison

## Summary

Computes XP-EHH (cross-population extended haplotype homozygosity) between each possible pair of populations on biallelic SNP data, as implemented in the scalepopgen Nextflow signatures-of-selection sub-workflow using selscan on Beagle/SHAPEIT5-phased haplotypes.

## When to use

Use when detecting recent selective sweeps by contrasting haplotype lengths between two populations from chromosome-split, tabix-indexed biallelic VCFs plus a sample-to-population .map file.

## When NOT to use

- Input VCFs are not phased or not split/indexed per chromosome — phase with Beagle or SHAPEIT5 first, as selscan requires phased haplotypes
- Only a single population is present — XP-EHH is a between-population comparison; use a within-population scan such as iHS or SweepFinder2 instead
- Data are multi-allelic or non-SNP variants — scalepopgen's selection sub-workflow targets biallelic SNP data only

## Inputs

- Filtered biallelic SNP VCF files split by chromosome and indexed with tabix (listed in a vcf_input.csv with chrom,vcf,vcf_idx header)
- Tab-delimited sample map file (individual IDs and population IDs, no header, .map extension)
- YAML parameters file specifying XP-EHH analysis options and the selection threshold

## Outputs

- Phased VCF/haplotype files per chromosome per population pair
- selscan XP-EHH result files for each possible pair of populations
- Collected scan results flagged against the user-defined selection threshold

## How to apply

Launch the scalepopgen signatures-of-selection sub-workflow via Nextflow with a YAML parameters file (e.g. test_full.yaml) under a container profile such as singularity. First phase the filtered, chromosome-split VCF files with Beagle v5.2 or SHAPEIT5 v5.1.1, since EHH-based scans in selscan v2.0.2 require phased haplotypes. Then run selscan to compute XP-EHH for each possible pair of populations defined in the sample map (complementing per-population iHS). Regions exceeding the user-defined selection threshold are reported as candidate sweep loci; when Ensembl species annotation is available, candidate regions can be linked to Ensembl locations. Note that scalepopgen's interactive Plotly Manhattan plots are produced for each selection analysis except XP-EHH, so XP-EHH results are inspected from the raw result files.

## Related tools

- **selscan** (v2.0.2 EHH-based scan tool that computes XP-EHH for each possible pair of populations on the phased haplotypes)
- **BEAGLE** (v5.2 phasing tool preparing filtered VCFs for selscan XP-EHH input)
- **SHAPEIT5** (v5.1.1 alternative phasing tool before running selscan)
- **Nextflow** (orchestrates the signatures-of-selection sub-workflow including phasing, XP-EHH, and result collection) — https://github.com/Popgen48/scalepopgen

## Examples

```
nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume
```

## Evaluation signals

- Phased VCF files exist for every chromosome listed in vcf_input.csv before selscan runs
- XP-EHH result files are generated for every possible pair of populations defined in the sample map
- Candidate regions are flagged only above the user-defined selection threshold
- Pipeline completes under 'nextflow run scalepopgen -params-file test_full.yaml -profile singularity -resume' with resumable caching
- Interactive Manhattan plots are expected for each selection analysis except XP-EHH, so their absence for XP-EHH is expected, not a failure

## Limitations

- XP-EHH is excluded from scalepopgen's interactive Plotly Manhattan plotting; results must be inspected from raw output files
- Requires phased haplotypes — unphased VCFs must first pass Beagle or SHAPEIT5 phasing
- Pipeline depends on Nextflow, Java, and a container/environment system (conda, mamba, singularity, or docker); other tools are auto-downloaded by Nextflow

## Evidence

- [methods] XP-EHH computed for each population pair on phased haplotypes by selscan: [evidence span withheld — anchor status: not_found]
- [methods] Phasing prerequisite with Beagle or SHAPEIT5 before selscan: [evidence span withheld — anchor status: not_found]
- [abstract] scalepopgen implements single-population and pairwise-comparison selection scans: "(iv) implementing single-population and pair-wise population comparison-based procedures to identify genomic signatures of selection"
- [methods] Manhattan plots are produced for each analysis except XP-EHH: [evidence span withheld — anchor status: not_found]
- [readme] signatures of selection is one of the four scalepopgen sub-workflows: "signatures of selection"
