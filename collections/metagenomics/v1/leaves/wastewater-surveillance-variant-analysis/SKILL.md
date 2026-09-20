---
name: wastewater-surveillance-variant-analysis
description: Use when analyzing NGS wastewater sequencing reads to detect SARS-CoV-2
  mutations below 0.01 frequency and infer the distribution of lineages (e.g., Alpha/B.1.1.7)
  across sampling periods.
when_to_use_negative:
- When input is clinical isolate consensus genomes already assigned by Pangolin rather
  than low-frequency wastewater VCFs
- When the goal is de novo assembly or consensus-genome calling rather than detecting
  mutations below 0.01 allele frequency in mixed samples
- When no reference genome/annotation is available to match nucleotide mutations to
  genes and amino-acid substitutions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: lineagespot
  role: R framework that matches called VCF variants against lineage-characteristic
    mutation profiles and returns variant, lineage-hit, and lineage-report tables
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Variant caller chosen as the most productive and informative for detecting
    mutations below 0.01 frequency in wastewater samples
- name: Minimap2
  role: Maps cleaned wastewater reads to NC_045512 with a minimal chaining score of
    40
- name: Pangolin
  role: Source of lineage assignment rules used to define lineage-characteristic mutation
    profiles
- name: outbreak.info
  role: Alternative source of pre-calculated lineage-characteristic mutation profiles
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:80b5efa3c855a1056273c09d55c2c504dfc608a37358c2a84a03ff8b529f732c
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# wastewater-surveillance-variant-analysis

## Summary

A pipeline for detecting SARS-CoV-2 mutations and inferring lineage abundance in wastewater NGS data, built around the lineagespot R framework that matches freebayes-called low-frequency variants against lineage-characteristic mutation profiles.

## When to use

Use when analyzing NGS wastewater sequencing reads to detect SARS-CoV-2 mutations below 0.01 frequency and infer the distribution of lineages (e.g., Alpha/B.1.1.7) across sampling periods.

## When NOT to use

- When input is clinical isolate consensus genomes already assigned by Pangolin rather than low-frequency wastewater VCFs
- When the goal is de novo assembly or consensus-genome calling rather than detecting mutations below 0.01 allele frequency in mixed samples
- When no reference genome/annotation is available to match nucleotide mutations to genes and amino-acid substitutions

## Inputs

- Raw wastewater NGS FASTQ reads (paired-end)
- SARS-CoV-2 Wuhan reference genome NC_045512 (FASTA)
- NC_045512.2 GFF3 gene annotation
- Per-sample VCF files from low-frequency variant calling (freebayes)
- Lineage-characteristic mutation profiles (Pangolin rules or outbreak.info definitions)

## Outputs

- Overall variant table of detected nucleotide mutations with amino acid substitutions and allele frequencies
- Lineage hits table matching mutations to SARS-CoV-2 lineages
- Lineage report with per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts/proportions
- CSV files of the three tables; optional clustered heatmap of normalized per-gene amino-acid substitution counts

## How to apply

Retrieve raw wastewater FASTQ reads (e.g., from ENA study PRJEB44141/ERP128154) and map cleaned reads to the SARS-CoV-2 Wuhan reference NC_045512 with Minimap2 using a minimal chaining score of 40; retain only paired-end sequences with a minimum trimmed read length of 200 nt and sliding-window quality threshold of 15. Call low-frequency variants with freebayes, which the article identified as the most productive and informative caller among freebayes, mpileup, and GATK Mutect2 (cancer-only mode), producing per-sample VCF files. Annotate variants (e.g., SnpEff with the NC_045512.2 version 5.0 database) and run the single lineagespot() function, supplying the VCF folder, the NC_045512.2 GFF3 annotation, and the reference genome so detected mutations are matched against lineage-characteristic mutation profiles defined via Pangolin rules or outbreak.info. Collect the returned list of three tables — overall variant table, lineage hits, and lineage report with mean AF, mean unique AF, non-zero min AF, and variant counts/proportions per lineage — and export them as CSV. Downstream analyses include collapsing mutations at the gene/amino-acid-substitution level to reduce noise and hierarchical clustering (Euclidean distance, ward.D) of normalized per-gene mutation counts.

## Related tools

- **lineagespot** (R framework that matches called VCF variants against lineage-characteristic mutation profiles and returns variant, lineage-hit, and lineage-report tables) — https://github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Variant caller chosen as the most productive and informative for detecting mutations below 0.01 frequency in wastewater samples)
- **Minimap2** (Maps cleaned wastewater reads to NC_045512 with a minimal chaining score of 40)
- **Pangolin** (Source of lineage assignment rules used to define lineage-characteristic mutation profiles)
- **outbreak.info** (Alternative source of pre-calculated lineage-characteristic mutation profiles)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); res <- lineagespot(vcf_folder = "vcf/", gff3_file = "NC_045512.2.gff3", ref_genome = "NC_045512.2.fa")
```

## Evaluation signals

- All three lineagespot output tables exist, are non-empty, and the lineage report contains per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts/proportions
- Computed non-zero minimum AF of B.1.1.7-unique mutations reproduces reported values, e.g., 16.67% for 05–11 Feb 2021 and 76.71% for 05–11 Mar 2021 within numerical tolerance
- Per-sample VCFs contain variants below 0.01 allele frequency, consistent with freebayes being chosen for low-frequency detection
- Detected mutations map to expected genes and amino-acid substitutions relative to the NC_045512.2 annotation
- Lineage assignment trends (e.g., rapid domination of Alpha/B.1.1.7) qualitatively agree with clinical samples from the same area and time periods

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Preprocessing was inconsistent across samples: primer trimming and remapping to the reference genome were omitted for four samples between 19 March 2021 and 15 April 2021
- No software versions were reported for Minimap2, iVar, freebayes, or SnpEff

## Evidence

- [abstract] we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS): "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [results] we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score: "mapped to the SARS-CoV-2 reference genome (Wuhan variant, NC_045512), using Minimap2 tool19 with a minimal chaining score"
- [discussion] a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality: "a minimum of 200 length in nucleotides for a read to be retained after trimming, and a minimum threshold for sliding window of 15 quality"
- [readme] lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools: "Lineagespot is a framework written in [R](https://www.r-project.org/), and aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
