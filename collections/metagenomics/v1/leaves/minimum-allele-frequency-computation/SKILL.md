---
name: minimum-allele-frequency-computation
description: Use when you have freebayes VCFs from wastewater NGS samples mapped to
  SARS-CoV-2 NC_045512 and need to reproduce or recompute lineage-specific AF statistics
  such as the non-zero minimum AF of B.1.1.7-unique mutations.
when_to_use_negative:
- Samples lacking freebayes VCF calls (e.g., if another caller such as mpileup or
  GATK Mutect2 was used, AF values may not reproduce the reported Table 5 statistics)
- Samples for which primer trimming and remapping were skipped (e.g., the four samples
  between 19 March and 15 April 2021), making AF comparability questionable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: freebayes
  role: low-frequency variant caller producing the VCF allele frequencies used in
    the non-zero minimum AF computation
  repo: https://github.com/BiodataAnalysisGroup/lineagespot/blob/master/inst/scripts/raw-data-analysis.md
- name: Minimap2
  role: maps cleaned reads to SARS-CoV-2 NC_045512 with minimal chaining score 40
    prior to variant calling
- name: lineagespot
  role: R framework that merges VCFs and identifies SARS-CoV-2 mutations/lineages
    against the B.1.1.7 profile
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: source of the B.1.1.7 lineage definition used to select unique mutations
- name: outbreak.info
  role: alternative source of pre-calculated lineage-characteristic mutation profiles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:ec8dc71422513c5f9e9d7970737ce22028c9758cfc18d14c348c6ffc9199007e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Minimap2
  - Pangolin
  - R
  - freebayes
  - lineagespot
  - outbreak.info
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# minimum-allele-frequency-computation

## Summary

Recomputes the reported per-period allele frequency statistics for B.1.1.7 (Alpha)-unique mutations in Thessaloniki wastewater samples by extracting allele frequencies from freebayes VCFs and taking the non-zero minimum (e.g., 16.67% for 5–11 Feb 2021; 76.71% for 5–11 Mar 2021).

## When to use

Use when you have freebayes VCFs from wastewater NGS samples mapped to SARS-CoV-2 NC_045512 and need to reproduce or recompute lineage-specific AF statistics such as the non-zero minimum AF of B.1.1.7-unique mutations.

## When NOT to use

- Samples lacking freebayes VCF calls (e.g., if another caller such as mpileup or GATK Mutect2 was used, AF values may not reproduce the reported Table 5 statistics)
- Samples for which primer trimming and remapping were skipped (e.g., the four samples between 19 March and 15 April 2021), making AF comparability questionable

## Inputs

- Raw wastewater NGS FASTQ reads (paired-end) from ENA study PRJEB44141 (ERP128154)
- SARS-CoV-2 Wuhan reference genome NC_045512
- freebayes VCF files per wastewater sample
- B.1.1.7-unique lineage-characteristic mutation profile from Pangolin rules or outbreak.info

## Outputs

- Per-mutation allele frequency table for B.1.1.7-unique mutations per sample (b117_min_allele_frequencies.csv)
- Computed non-zero minimum AF, average AF of all B.1.1.7 mutations, and average AF of unique mutations per time period

## How to apply

Retrieve the raw reads from ENA study PRJEB44141 (ERP128154), clean and trim them (paired-end only, min trimmed length 200 nt, sliding-window quality 15) and map to the Wuhan reference NC_045512 with Minimap2 using a minimal chaining score of 40. Call low-frequency variants with freebayes — the caller the paper judged most productive and informative for mutations below 0.01 frequency — producing per-sample VCF files. Define the B.1.1.7-unique mutation profile via Pangolin rules or outbreak.info definitions, match annotated VCF variants against it, extract each mutation's allele frequency, and compute the non-zero minimum AF across B.1.1.7-unique mutations per sample. Compare computed values against Table 5 (32.71% average AF of all B.1.1.7 mutations, 35.15% of unique, 16.67% minimum for 5–11 Feb 2021; 95.54%, 96.28%, 76.71% for 5–11 Mar 2021) within numerical tolerance.

## Related tools

- **freebayes** (low-frequency variant caller producing the VCF allele frequencies used in the non-zero minimum AF computation) — https://github.com/BiodataAnalysisGroup/lineagespot/blob/master/inst/scripts/raw-data-analysis.md
- **Minimap2** (maps cleaned reads to SARS-CoV-2 NC_045512 with minimal chaining score 40 prior to variant calling)
- **lineagespot** (R framework that merges VCFs and identifies SARS-CoV-2 mutations/lineages against the B.1.1.7 profile) — github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (source of the B.1.1.7 lineage definition used to select unique mutations)
- **outbreak.info** (alternative source of pre-calculated lineage-characteristic mutation profiles)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Computed non-zero minimum AF of B.1.1.7-unique mutations equals 16.67% for the 5–11 Feb 2021 sample and 76.71% for the 5–11 Mar 2021 sample within numerical tolerance
- Average AF of all B.1.1.7 mutations reproduces 32.71% (Feb) and 95.54% (Mar); average AF of unique mutations reproduces 35.15% (Feb) and 96.28% (Mar)
- Positions with reference read depth equal to zero are removed before AF extraction
- All B.1.1.7-unique mutations matched against the Pangolin/outbreak.info profile are represented in the output table, and output CSV b117_min_allele_frequencies.csv is produced

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping were omitted for the four samples between 19 March 2021 and 15 April 2021

## Evidence

- [results] freebayes most productive caller: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] minimap2 mapping parameters: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [discussion] read depth filter: "positions with reference read depth equal to zero are removed from the first vector"
- [abstract] lineagespot purpose: "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [readme] lineagespot README description: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
