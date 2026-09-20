---
name: vcf-lineage-mutation-annotation
description: Use when the input is one or more VCF files from SARS-CoV-2 wastewater
  NGS data (e.g., freebayes calls against NC_045512) and the goal is to list detected
  mutations and identify which lineages (e.g., B.1.177, B.1.1.7, B.1.351) are present.
when_to_use_negative:
- Input is raw FASTQ reads rather than called variants — variant calling (mapping
  with Minimap2, freebayes at 0.01 mutation frequency) must happen first
- 'The organism/reference is not SARS-CoV-2 (NC_045512): the Pangolin-rule and outbreak.info
  mutation matching is SARS-CoV-2 specific'
- The goal is absolute lineage abundance quantification rather than detection/monitoring
  of lineages — the method identifies lineages present but the article notes assignment
  assumptions can mis-assign lineages
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: R framework that ingests VCF files, merges mutations, and identifies SARS-CoV-2
    lineages via Pangolin rules / outbreak.info definitions
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: SnpEff
  role: Annotates detected nucleotide mutations with amino acid substitutions using
    the NC_045512.2 (version 5.0) database
- name: Pangolin
  role: Source of lineage-definition rules used to assign detected mutations to SARS-CoV-2
    lineages
- name: freebayes
  role: Produces the input VCFs; run at low mutation frequency 0.01 to capture variants
    below 0.01 frequency
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:ea1909de990d6daadd090e0934d22d2c2c3879c4425b8e6046876ab241f4b7a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GATK Mutect2
  - Minimap2
  - Pangolin
  - R
  - SnpEff
  - freebayes
  - lineagespot
  - mpileup
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# vcf-lineage-mutation-annotation

## Summary

Annotate and merge SARS-CoV-2 VCF files from wastewater sequencing into a combined table of nucleotide mutations, amino acid substitutions, allele frequencies and overlapping genes, then infer lineage-characteristic mutations per lineage using Pangolin rules / outbreak.info definitions via the lineagespot R framework.

## When to use

Use when the input is one or more VCF files from SARS-CoV-2 wastewater NGS data (e.g., freebayes calls against NC_045512) and the goal is to list detected mutations and identify which lineages (e.g., B.1.177, B.1.1.7, B.1.351) are present.

## When NOT to use

- Input is raw FASTQ reads rather than called variants — variant calling (mapping with Minimap2, freebayes at 0.01 mutation frequency) must happen first
- The organism/reference is not SARS-CoV-2 (NC_045512): the Pangolin-rule and outbreak.info mutation matching is SARS-CoV-2 specific
- The goal is absolute lineage abundance quantification rather than detection/monitoring of lineages — the method identifies lineages present but the article notes assignment assumptions can mis-assign lineages

## Inputs

- One or more VCF files from SARS-CoV-2 variant calling (e.g., freebayes output)
- Read alignments against the SARS-CoV-2 Wuhan reference genome NC_045512
- Lineage-characteristic mutation definitions (Pangolin rules or outbreak.info profiles)

## Outputs

- Combined table of detected nucleotide mutations with amino acid substitutions, allele frequency and overlapping gene
- Per-lineage counts of unique nucleotide mutations (Nt) for lineages such as B.1.177, B.1.1.7 and B.1.351
- Collapsed gene/amino-acid-substitution level mutation table

## How to apply

Start from VCF files produced by variant calling (freebayes with a low mutation frequency parameter of 0.01 is the article's recommended caller for detecting variants below 0.01 frequency) on reads mapped to the Wuhan reference genome NC_045512 with Minimap2 (minimal chaining score 40). Install lineagespot from GitHub (devtools::install_github('BiodataAnalysisGroup/lineagespot')) and run it on a single VCF or a list of VCFs. Merge all VCFs into a combined table storing detected nucleotide mutations with their corresponding amino acid substitutions, allele frequency and overlapping gene, annotated with SnpEff against the NC_045512.2 (version 5.0) database. Match mutations to lineage definitions retrieved either as a Pangolin rule or through the outbreak.info definition to count unique lineage-characteristic nucleotide mutations (Nt) per lineage. Collapse results at the gene and amino-acid substitution level to reduce noise from nucleotide mutations mapping to the same amino acid change.

## Related tools

- **lineagespot** (R framework that ingests VCF files, merges mutations, and identifies SARS-CoV-2 lineages via Pangolin rules / outbreak.info definitions) — github.com/BiodataAnalysisGroup/lineagespot
- **SnpEff** (Annotates detected nucleotide mutations with amino acid substitutions using the NC_045512.2 (version 5.0) database)
- **Pangolin** (Source of lineage-definition rules used to assign detected mutations to SARS-CoV-2 lineages)
- **freebayes** (Produces the input VCFs; run at low mutation frequency 0.01 to capture variants below 0.01 frequency)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); lineagespot(vcf = c("freebayes_0.01.vcf"), ref = "NC_045512")
```

## Evaluation signals

- The combined output table contains, for every VCF, the nucleotide mutation with its amino acid substitution, allele frequency and overlapping gene
- Per-caller Nt counts for B.1.177, B.1.1.7 and B.1.351 reproduce the paper's variant-caller comparison table for the 05–11 February 2021 Thessaloniki sample
- The freebayes (mutation frequency 0.01) VCF yields the highest and most informative Nt counts among the three callers, supporting the paper's conclusion
- Lineage assignments agree with Pangolin rules or outbreak.info lineage-characteristic mutation profiles
- Collapsing at gene and amino-acid substitution level reduces, without losing distinct amino acid changes, noise from nucleotide mutations that correspond to the same amino acid change

## Limitations

- Pangolin-based lineage assignment assumes reads matching any single rule of lineage L imply lineage L; when different reads satisfy different rules of the same lineage, lineage L can be incorrectly assigned
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across the study: for four samples between 19 March 2021 and 15 April 2021, primer trimming and remapping to the reference genome were not applied
- Software versions for Minimap2, iVar, freebayes and SnpEff are not reported in the article

## Evidence

- [readme] lineagespot identifies SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s): "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
- [results] VCFs merged into a combined mutation table: "all VCF files were merged into a combined table in which all the detected nucleotide mutations along with the corresponding amino acid substitutions"
- [results] lineage definitions from Pangolin or outbreak.info: "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [results] freebayes recommended as most productive variant caller: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] Lineage mis-assignment caveat: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
