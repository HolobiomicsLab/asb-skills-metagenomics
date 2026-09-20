---
name: sars-cov-2-lineage-mutation-matching
description: Use when VCF variant calls from SARS-CoV-2 wastewater sequencing must
  be matched to a lineage's characteristic mutation set and summarized by allele-frequency
  metrics (e.g., minimum non-zero AF per time period).
when_to_use_negative:
- Input variants come from a caller other than the validated low-frequency pipeline
  (freebayes), since mpileup and GATK Mutect2 were less productive for mutations below
  0.01 frequency
- Samples lack primer trimming/remapping consistency (e.g., the four samples between
  19 March and 15 April 2021), which can distort per-mutation allele frequencies
- The goal is clinical consensus-level lineage assignment rather than low-frequency
  mutation detection in wastewater mixtures
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3673
tools:
- name: lineagespot
  role: R framework used to identify SARS-CoV-2 mutations and lineages from VCF files
    against lineage profiles
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Variant caller judged most productive and informative for detecting mutations
    below 0.01 frequency; supplies AFs per mutation
- name: Minimap2
  role: Maps cleaned paired-end reads to the NC_045512 reference with a minimal chaining
    score of 40 prior to variant calling
- name: Pangolin
  role: Source of lineage definitions/rules against which detected mutations are matched
- name: outbreak.info
  role: Alternative source of pre-calculated lineage-characteristic mutation profiles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:8fe7c8165d82b02c9010bb6ccaaeb4123e5af355ba8b0e3bd3020370b3edabfc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - Minimap2
  - Pangolin
  - R
  - freebayes
  - lineagespot
  - outbreak.info
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# sars-cov-2-lineage-mutation-matching

## Summary

A method for matching variant calls from wastewater NGS data against SARS-CoV-2 lineage-characteristic mutation profiles (e.g., B.1.1.7/Alpha defined via Pangolin rules or outbreak.info) and computing per-lineage allele-frequency statistics such as the non-zero minimum allele frequency.

## When to use

Use when VCF variant calls from SARS-CoV-2 wastewater sequencing must be matched to a lineage's characteristic mutation set and summarized by allele-frequency metrics (e.g., minimum non-zero AF per time period).

## When NOT to use

- Input variants come from a caller other than the validated low-frequency pipeline (freebayes), since mpileup and GATK Mutect2 were less productive for mutations below 0.01 frequency
- Samples lack primer trimming/remapping consistency (e.g., the four samples between 19 March and 15 April 2021), which can distort per-mutation allele frequencies
- The goal is clinical consensus-level lineage assignment rather than low-frequency mutation detection in wastewater mixtures

## Inputs

- freebayes VCF files of low-frequency variants per wastewater sample
- Lineage-characteristic mutation profile (Pangolin rule or outbreak.info definition, e.g., B.1.1.7-unique mutations)
- FASTQ reads from ENA study PRJEB44141 (ERP128154) and SARS-CoV-2 reference genome NC_045512 (for upstream variant calling)

## Outputs

- Table of detected nucleotide mutations with amino acid substitutions, allele frequency, and overlapping gene per sample
- Per-lineage allele-frequency metrics per time period (e.g., non-zero minimum AF, average AF of all and unique lineage mutations)
- CSV comparing computed vs. reported values (e.g., b117_min_allele_frequencies.csv)

## How to apply

Retrieve cleaned, paired-end reads (minimum 200 nt after trimming, sliding-window quality 15), map them to the SARS-CoV-2 Wuhan reference NC_045512 with Minimap2 using a minimal chaining score of 40, and call low-frequency variants with freebayes, which the paper found most productive and informative for detecting mutations below 0.01 frequency. Annotate the resulting VCFs (SnpEff, NC_045512.2 version 5.0 database) and match variants against the lineage-characteristic mutation profile retrieved either as a Pangolin rule or through the outbreak.info definition. For each lineage-unique mutation detected in a sample, extract its allele frequency from the freebayes VCF and compute the non-zero minimum allele frequency across mutations. Assemble per-period values and validate against reported metrics, e.g., 16.67% non-zero minimum AF for B.1.1.7 in 5–11 Feb 2021 and 76.71% for 5–11 Mar 2021, alongside average AFs of 32.71%/35.15% (Feb) and 95.54%/96.28% (Mar).

## Related tools

- **lineagespot** (R framework used to identify SARS-CoV-2 mutations and lineages from VCF files against lineage profiles) — https://github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Variant caller judged most productive and informative for detecting mutations below 0.01 frequency; supplies AFs per mutation)
- **Minimap2** (Maps cleaned paired-end reads to the NC_045512 reference with a minimal chaining score of 40 prior to variant calling)
- **Pangolin** (Source of lineage definitions/rules against which detected mutations are matched)
- **outbreak.info** (Alternative source of pre-calculated lineage-characteristic mutation profiles)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); lineagespot(vcf = c("feb_2021.vcf", "mar_2021.vcf"), af_threshold = 0.01)
```

## Evaluation signals

- Computed non-zero minimum AF of B.1.1.7-unique mutations matches reported Table 5 values within numerical tolerance: 16.67% for 5–11 Feb 2021 and 76.71% for 5–11 Mar 2021
- Average AF of all B.1.1.7 mutations reproduces 32.71% (Feb) and 95.54% (Mar); average AF of unique mutations reproduces 35.15% (Feb) and 96.28% (Mar)
- Every output row carries a gene, amino-acid substitution, and allele frequency, matching the merged VCF table schema
- Only mutations present in the lineage-characteristic profile (Pangolin rule / outbreak.info) are counted as lineage-unique; positions with reference read depth zero are excluded

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were not applied for the four samples between 19 March 2021 and 15 April 2021

## Evidence

- [results] freebayes most productive: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [results] lineage definition source: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [discussion] mapping parameters: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [discussion] read filtering: "a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality"
- [readme] lineagespot scope: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
