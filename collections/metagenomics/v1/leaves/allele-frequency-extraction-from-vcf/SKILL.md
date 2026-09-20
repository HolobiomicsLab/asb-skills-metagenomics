---
name: allele-frequency-extraction-from-vcf
description: Use when VCF files from low-frequency variant calling on wastewater NGS
  samples must be matched against a lineage mutation profile (e.g., B.1.1.7 unique
  mutations via Pangolin rules or outbreak.info) and per-mutation allele frequencies
  or a non-zero minimum AF are needed.
when_to_use_negative:
- Variant files are not VCF format or were not produced by a low-frequency-aware caller
  (mpileup, GATK Mutect2 were assessed but freebayes was preferred for sub-0.01 frequencies)
- The target mutations are not lineage-characteristic for the queried lineage — the
  skill matches a specific Pangolin/outbreak.info mutation profile, not arbitrary
  variant lists
- Positions with reference read depth equal to zero have not been removed from the
  analysis vectors
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: freebayes
  role: Variant caller producing the per-sample VCF files from which allele frequencies
    are extracted; identified as most productive for detecting mutations below 0.01
    frequency
- name: lineagespot
  role: R framework used to identify SARS-CoV-2 mutations from VCF files and match
    them against lineage-characteristic profiles
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: Minimap2
  role: Maps cleaned paired-end reads to the NC_045512 Wuhan reference genome with
    minimal chaining score 40, upstream of variant calling
- name: Pangolin
  role: Source of lineage-definition rules used to define the B.1.1.7-unique mutation
    set
- name: outbreak.info
  role: Alternative source of pre-calculated lineage-characteristic mutation profiles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:c36f7887b35216ad897c1431443b026424163b12307c8949cfb63993a490740d
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# allele-frequency-extraction-from-vcf

## Summary

Extracts allele frequencies of lineage-characteristic SARS-CoV-2 mutations (e.g., B.1.1.7/Alpha-unique mutations) from freebayes-generated VCF files of wastewater NGS samples and computes summary statistics such as the non-zero minimum AF, reproducing reported Table 5 values for Thessaloniki wastewater samples.

## When to use

Use when VCF files from low-frequency variant calling on wastewater NGS samples must be matched against a lineage mutation profile (e.g., B.1.1.7 unique mutations via Pangolin rules or outbreak.info) and per-mutation allele frequencies or a non-zero minimum AF are needed.

## When NOT to use

- Variant files are not VCF format or were not produced by a low-frequency-aware caller (mpileup, GATK Mutect2 were assessed but freebayes was preferred for sub-0.01 frequencies)
- The target mutations are not lineage-characteristic for the queried lineage — the skill matches a specific Pangolin/outbreak.info mutation profile, not arbitrary variant lists
- Positions with reference read depth equal to zero have not been removed from the analysis vectors

## Inputs

- freebayes VCF files per wastewater sample (variants called against NC_045512)
- B.1.1.7 (Alpha)-unique lineage-characteristic mutation list from Pangolin rules or outbreak.info
- Cleaned paired-end FASTQ reads mapped with Minimap2 (minimal chaining score 40) to NC_045512

## Outputs

- Per-mutation allele frequencies for B.1.1.7-unique mutations per sample
- Non-zero minimum allele frequency per sample (e.g., 16.67% Feb, 76.71% Mar 2021)
- Average AF of all B.1.1.7 mutations and of B.1.1.7-unique mutations per sample
- Comparison table (e.g., b117_min_allele_frequencies.csv) of computed vs. reported values

## How to apply

Call low-frequency variants with freebayes on cleaned reads mapped to the SARS-CoV-2 Wuhan reference genome NC_045512, since freebayes was identified as the most productive and informative caller for mutations below 0.01 frequency. Define the lineage-characteristic mutation profile (e.g., B.1.1.7 unique mutations) via Pangolin rules or outbreak.info. For each profile mutation detected in the sample's VCF, extract its allele frequency (AF), discard zero-frequency entries, and compute the non-zero minimum AF (e.g., 16.67% for 5–11 Feb 2021; 76.71% for 5–11 Mar 2021). Optionally also compute the average AF over all B.1.1.7 mutations and over B.1.1.7-unique mutations (32.71%/35.15% for February; 95.54%/96.28% for March) and compare computed values to reported values within numerical tolerance.

## Related tools

- **freebayes** (Variant caller producing the per-sample VCF files from which allele frequencies are extracted; identified as most productive for detecting mutations below 0.01 frequency)
- **lineagespot** (R framework used to identify SARS-CoV-2 mutations from VCF files and match them against lineage-characteristic profiles) — https://github.com/BiodataAnalysisGroup/lineagespot
- **Minimap2** (Maps cleaned paired-end reads to the NC_045512 Wuhan reference genome with minimal chaining score 40, upstream of variant calling)
- **Pangolin** (Source of lineage-definition rules used to define the B.1.1.7-unique mutation set)
- **outbreak.info** (Alternative source of pre-calculated lineage-characteristic mutation profiles)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Computed non-zero minimum AF of B.1.1.7-unique mutations equals the reported Table 5 values: 16.67% for 5–11 Feb 2021 and 76.71% for 5–11 Mar 2021 (within numerical tolerance)
- Average AF of all B.1.1.7 mutations and of unique B.1.1.7 mutations reproduces reported values (32.71/35.15 Feb; 95.54/96.28 Mar)
- Extracted AFs correspond only to mutations present in the VCF with non-zero frequency
- Every extracted AF can be traced to a B.1.1.7-unique mutation in the Pangolin/outbreak.info profile
- Upstream mapping used only paired-end reads with ≥200 nt after trimming (sliding window quality 15) and removed unmatched/multiple mappings

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping were omitted for the four samples between 19 March and 15 April 2021
- Software versions for Minimap2, iVar, freebayes, and SnpEff were not reported

## Evidence

- [results] freebayes preferred for low-frequency AF extraction: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [abstract] lineagespot purpose: "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples"
- [discussion] mapping parameters: "using Minimap2 tool19 with a minimal chaining score (matching bases minus log gap penalty) equal to 40"
- [discussion] read filtering thresholds: "a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality"
- [readme] lineagespot installation from README: [evidence span withheld — anchor status: external_doc]
