---
name: vcf-variant-table-processing
description: Use when input consists of single or multiple SARS-CoV-2 VCF files (e.g.,
  from freebayes variant calling of wastewater or ARTIC NGS samples) and the goal
  is to detect lineage-characteristic mutations and quantify per-lineage abundance.
when_to_use_negative:
- Input is raw FASTQ sequencing data — variant calling (e.g., freebayes after Minimap2
  mapping and primer trimming) must happen first
- Samples are from non-SARS-CoV-2 organisms or VCFs not aligned to the NC_045512 Wuhan
  reference genome
- A consensus-genome Pangolin-style single-lineage assignment is needed for a pure
  clinical isolate rather than mixed lineage detection in wastewater
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: 'Core R framework: merges VCF inputs, annotates mutations, and detects SARS-CoV-2
    lineages in a single function call'
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language/runtime for the lineagespot framework and downstream
    table export
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Recommended upstream variant caller for detecting low-frequency (< 0.01) mutations
    that populate the input VCFs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:15c273fc05603ef519b8636a6306fa2334c7529564df989578b78b4188f83c05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# vcf-variant-table-processing

## Summary

An R-based skill that ingests one or more SARS-CoV-2 VCF files and produces a merged variant table (nucleotide mutations, amino acid substitutions, allele frequencies) plus lineage hits and a per-lineage abundance report using the lineagespot package.

## When to use

Use when input consists of single or multiple SARS-CoV-2 VCF files (e.g., from freebayes variant calling of wastewater or ARTIC NGS samples) and the goal is to detect lineage-characteristic mutations and quantify per-lineage abundance.

## When NOT to use

- Input is raw FASTQ sequencing data — variant calling (e.g., freebayes after Minimap2 mapping and primer trimming) must happen first
- Samples are from non-SARS-CoV-2 organisms or VCFs not aligned to the NC_045512 Wuhan reference genome
- A consensus-genome Pangolin-style single-lineage assignment is needed for a pure clinical isolate rather than mixed lineage detection in wastewater

## Inputs

- One or more SARS-CoV-2 VCF files (variant calling format) from NGS variant calling (e.g., freebayes)
- NC_045512.2 GFF3 genome annotation
- SARS-CoV-2 Wuhan reference genome FASTA (NC_045512)

## Outputs

- Overall variant table of all detected nucleotide mutations with amino acid substitutions and allele frequencies
- Lineage hits table matching mutations to lineage-characteristic mutation profiles
- Lineage report table with per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts/proportions

## How to apply

Install lineagespot in R via devtools::install_github('BiodataAnalysisGroup/lineagespot'). Supply the folder of VCF files, the NC_045512.2 GFF3 annotation, and the SARS-CoV-2 Wuhan reference FASTA to the single lineagespot() function call. The tool matches detected mutations against lineage-characteristic mutation profiles (e.g., pre-calculated by outbreak.info or defined as Pangolin rules). Variant calling upstream should use freebayes, which the authors judged the most productive and informative caller for low-frequency variants (< 0.01). Collect the returned list of three tables — the overall variant table, lineage hits, and the lineage report — and write each to CSV.

## Related tools

- **lineagespot** (Core R framework: merges VCF inputs, annotates mutations, and detects SARS-CoV-2 lineages in a single function call) — https://github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language/runtime for the lineagespot framework and downstream table export) — https://github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Recommended upstream variant caller for detecting low-frequency (< 0.01) mutations that populate the input VCFs)

## Examples

```
res <- lineagespot(vcf_folder = "vcf_files/", gff3 = "NC_045512.2.gff3", ref_genome = "NC_045512.2.fasta")
```

## Evaluation signals

- The single lineagespot() call returns exactly three tables: overall variant table, lineage hits, and lineage report
- Each output CSV exists and is non-empty after writing to disk
- The lineage report contains the expected per-lineage summary fields: mean AF, mean unique AF, non-zero min AF, variant counts and proportions
- The variant table lists nucleotide mutations with corresponding amino acid substitutions and allele frequencies
- Lineage assignments are consistent with lineage-characteristic mutation profiles (e.g., Alpha/B.1.1.7 should be detected in samples where its defining mutations dominate)

## Limitations

- Pangolin-rule-based assignment can be incorrect when different reads satisfy different rules of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater samples
- Preprocessing inconsistencies (e.g., primer trimming and remapping omitted for four samples between 19 March and 15 April 2021) can affect variant calls feeding the VCFs

## Evidence

- [abstract] lineagespot proposed methodology: "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [readme] lineagespot README purpose: "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s) (i.e., [variant calling format]"
- [results] freebayes recommended: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [results] VCF merging into combined table: "all VCF files were merged into a combined table in which all the detected nucleotide mutations along with the corresponding amino acid substitutions that have been identified are stored"
- [discussion] Pangolin rule limitation: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
