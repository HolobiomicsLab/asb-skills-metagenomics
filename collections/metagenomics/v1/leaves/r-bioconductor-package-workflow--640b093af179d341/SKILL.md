---
name: r-bioconductor-package-workflow
description: Use when you have SARS-CoV-2 VCF files (e.g., from freebayes variant
  calling of ARTIC wastewater NGS data) and need to detect lineage-characteristic
  mutations and infer lineage abundances in R.
when_to_use_negative:
- Input is raw FASTQ reads rather than called variants — first run the raw-data pipeline
  (QC, Minimap2 mapping to NC_045512, freebayes calling) documented in inst/scripts/raw-data-analysis.md
- You need consensus lineage assignment of an isolate-level sample — the article notes
  Pangolin-based rule matching can misassign when different reads satisfy different
  rules of the same lineage
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0769
tools:
- name: lineagespot
  role: R package whose single lineagespot() function generates the variant table,
    lineage hits, and lineage report from VCF, GFF3, and reference inputs
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: R
  role: Implementation language and runtime for the lineagespot package workflow
- name: freebayes
  role: Upstream variant caller producing the low-frequency VCF inputs consumed by
    lineagespot (considered the most productive for variants < 0.01 AF)
- name: Pangolin
  role: Source of lineage definitions/rules that lineagespot matches sample mutations
    against
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:23b0453d3f97e73579adebe4804a61e5fca1b56460ebf8f3bb5a8d10ca3945d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# r-bioconductor-package-workflow

## Summary

Run the lineagespot R package (BiodataAnalysisGroup/lineagespot) end-to-end: a single lineagespot() call on input VCF files, with the NC_045512.2 GFF3 annotation and SARS-CoV-2 Wuhan reference FASTA, returns the overall variant table, lineage hits table, and per-lineage lineage report (mean AF, mean unique AF, non-zero min AF, variant counts/proportions).

## When to use

Use when you have SARS-CoV-2 VCF files (e.g., from freebayes variant calling of ARTIC wastewater NGS data) and need to detect lineage-characteristic mutations and infer lineage abundances in R.

## When NOT to use

- Input is raw FASTQ reads rather than called variants — first run the raw-data pipeline (QC, Minimap2 mapping to NC_045512, freebayes calling) documented in inst/scripts/raw-data-analysis.md
- You need consensus lineage assignment of an isolate-level sample — the article notes Pangolin-based rule matching can misassign when different reads satisfy different rules of the same lineage

## Inputs

- One or more SARS-CoV-2 variant calling format (VCF) files from low-frequency variant calling (e.g., freebayes on ARTIC NGS reads)
- NC_045512.2 GFF3 genome annotation file
- SARS-CoV-2 Wuhan reference genome FASTA (NC_045512)

## Outputs

- Overall variant table: all detected nucleotide mutations with amino acid substitutions and allele frequencies across input VCFs
- Lineage hits table: lineage-characteristic mutations matched in the sample
- Lineage report table: per-lineage abundance metrics (mean AF, mean unique AF, non-zero min AF, variant counts and proportions)

## How to apply

Install lineagespot from GitHub with devtools::install_github("BiodataAnalysisGroup/lineagespot") in R. Gather the input VCF file(s) — in the source study these are produced by a freebayes-based variant calling pipeline applied to Illumina ARTIC wastewater reads mapped to NC_045512 — plus the NC_045512.2 GFF3 annotation and the SARS-CoV-2 Wuhan reference FASTA (examples are in the package extdata folder and Zenodo deposit 10.5281/zenodo.4564182). Call lineagespot() once, supplying the VCF folder/path, the GFF3 file, and the reference genome, so detected nucleotide mutations are matched against lineage-characteristic mutation profiles derived from Pangolin rules/outbreak.info definitions. The single call returns a list of three tables: an overall variant table of nucleotide mutations with amino acid substitutions and allele frequencies, a lineage hits table, and a lineage report computing per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts/proportions. Write each table to disk (e.g., CSV) and validate that all three files exist, are non-empty, and that the lineage report carries the expected per-lineage summary columns.

## Related tools

- **lineagespot** (R package whose single lineagespot() function generates the variant table, lineage hits, and lineage report from VCF, GFF3, and reference inputs) — https://github.com/BiodataAnalysisGroup/lineagespot
- **R** (Implementation language and runtime for the lineagespot package workflow)
- **freebayes** (Upstream variant caller producing the low-frequency VCF inputs consumed by lineagespot (considered the most productive for variants < 0.01 AF))
- **Pangolin** (Source of lineage definitions/rules that lineagespot matches sample mutations against)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot"); res <- lineagespot(vcf_folder = "vcf_dir/", gff3_file = "NC_045512.2.gff3", ref fasta reference genome); write.csv per returned table
```

## Evaluation signals

- The single lineagespot() call returns exactly three output tables: overall variant table, lineage hits table, and lineage report table
- All three output files exist on disk and are non-empty after writing to CSV
- The lineage report contains per-lineage summary fields: mean AF, mean unique AF, non-zero min AF, and variant counts and proportions
- Nucleotide mutations in the variant table carry corresponding amino acid substitutions and allele frequencies, and reference read depth of zero positions were removed upstream
- Variant caller consistency check: freebayes outputs were judged the most productive/informative for low-frequency (< 0.01) variants versus mpileup and GATK Mutect2

## Limitations

- Pangolin-rule-based matching can incorrectly assign lineage L when one group of reads satisfies rule A and another group satisfies rule B of the same lineage
- Mutation identification in wastewater may be hindered by viral adsorption and decay
- Preprocessing was inconsistent in the source study: primer trimming and remapping were omitted for four samples between 19 March 2021 and 15 April 2021, and no software versions were reported for Minimap2, iVar, freebayes, or SnpEff

## Evidence

- [abstract] lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools: "lineagespot is an open-source tool, implemented in R, and is freely available on GitHub and registered on bio.tools"
- [readme] README installation and citation: [evidence span withheld — anchor status: external_doc]
- [results] freebayes as best variant caller: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [discussion] Pangolin rule misassignment caveat: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
