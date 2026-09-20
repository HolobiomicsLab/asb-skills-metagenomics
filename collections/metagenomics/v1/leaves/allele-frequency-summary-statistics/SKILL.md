---
name: allele-frequency-summary-statistics
description: Use when you have VCF files of SARS-CoV-2 variants (e.g., from freebayes
  on wastewater NGS data) and need per-lineage abundance metrics such as mean allele
  frequency and variant counts.
when_to_use_negative:
- Input VCFs were not called against the SARS-CoV-2 NC_045512 reference, since mutations
  are matched against NC_045512-based lineage profiles
- You need consensus/whole-lineage Pangolin assignment of a single clinical isolate
  rather than multi-lineage abundance estimation in a mixed sample
- Positions with reference read depth equal to zero have not been removed, which distorts
  low-frequency allele-frequency estimates
tools:
- name: lineagespot
  role: R framework that matches VCF variants against lineage-characteristic mutation
    profiles and computes per-lineage allele-frequency summary statistics in the lineage
    report
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Variant caller chosen as most productive and informative for producing low-frequency
    variant VCFs that feed the allele-frequency summaries
- name: R
  role: Implementation language; lineagespot() is a single R function returning the
    three output tables
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:5127710e8414e97cb7263732e90b938a7b97b3638f508f757612f6ca8f198dc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - R
  - freebayes
  - lineagespot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# allele-frequency-summary-statistics

## Summary

Compute per-lineage allele-frequency summary statistics (mean AF, mean unique AF, non-zero minimum AF, variant counts and proportions) from SARS-CoV-2 VCF variant tables using the lineagespot R package's lineage-report output, enabling lineage abundance and mutational load estimation in wastewater samples.

## When to use

Use when you have VCF files of SARS-CoV-2 variants (e.g., from freebayes on wastewater NGS data) and need per-lineage abundance metrics such as mean allele frequency and variant counts.

## When NOT to use

- Input VCFs were not called against the SARS-CoV-2 NC_045512 reference, since mutations are matched against NC_045512-based lineage profiles
- You need consensus/whole-lineage Pangolin assignment of a single clinical isolate rather than multi-lineage abundance estimation in a mixed sample
- Positions with reference read depth equal to zero have not been removed, which distorts low-frequency allele-frequency estimates

## Inputs

- One or more VCF files of SARS-CoV-2 variants (e.g., freebayes output with allele frequencies)
- NC_045512.2 GFF3 genome annotation
- SARS-CoV-2 Wuhan-Hu-1 reference genome FASTA (NC_045512.2)

## Outputs

- Overall variant table of all detected nucleotide mutations with amino acid substitutions, allele frequencies, and overlapping gene
- Table of lineage hits matched against lineage-characteristic mutation profiles
- Lineage report table with per-lineage mean AF, mean unique AF, non-zero min AF, and variant counts and proportions (exportable as CSV)

## How to apply

Run variant calling on ARTIC-based NGS data mapped to the SARS-CoV-2 Wuhan reference NC_045512.2 (freebayes was judged the most productive and informative caller for low-frequency variants), then merge VCFs into a combined table of nucleotide mutations, amino acid substitutions, and allele frequencies. Call the single lineagespot() function, supplying the VCF folder, the NC_045512.2 GFF3 annotation, and the reference genome, so detected mutations are matched against lineage-characteristic mutation profiles (Pangolin rules / outbreak.info definitions). Collect the returned list of three tables — overall variant table, lineage hits, and lineage report — and export the lineage report (per-lineage mean AF, mean unique AF, non-zero min AF, variant counts and proportions) as CSV. Optionally collapse the variant table at the gene and amino-acid substitution level to reduce noise from nucleotide mutations that correspond to the same amino-acid change.

## Related tools

- **lineagespot** (R framework that matches VCF variants against lineage-characteristic mutation profiles and computes per-lineage allele-frequency summary statistics in the lineage report) — https://github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Variant caller chosen as most productive and informative for producing low-frequency variant VCFs that feed the allele-frequency summaries)
- **R** (Implementation language; lineagespot() is a single R function returning the three output tables) — https://github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
res <- lineagespot(vcf_folder = "extdata/vcf/", gff3 = "NC_045512.2.gff3", reference = "NC_045512.2.fasta")
```

## Evaluation signals

- All three output tables (overall variant table, lineage hits, lineage report) exist and are non-empty
- Lineage report contains the expected per-lineage summary fields: mean AF, mean unique AF, non-zero min AF, variant counts and proportions
- Allele frequencies in the merged variant table fall within [0, 1] and match the input VCF values
- Lineage hits are consistent with the mutations present in the input VCFs and the GFF3-annotated amino-acid substitutions
- Regression test against the deposited example VCFs (Zenodo 10.5281/zenodo.4564182) reproduces the reported lineage abundances (e.g., Alpha B.1.1.7 dominance)

## Limitations

- Pangolin-based lineage assignment can incorrectly assign lineage L when different reads satisfy different rules (A and B) of the same lineage
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Positions with reference read depth equal to zero must be removed from the input vectors before summarization

## Evidence

- [other] the lineage report table computing per-lineage abundance metrics (mean AF, mean unique AF, non-zero min AF, variant counts and proportions): [evidence span withheld — anchor status: not_found]
- [abstract] we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS): "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [results] we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [results] Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise: "Table 4 was collapsed at the gene and amino acid substitution level, therefore reducing any data noise that is introduced by nucleotide mutations that correspond to the same amino acid change"
- [discussion] remove positions with reference read depth equal to zero: "positions with reference read depth equal to zero are removed from the first vector"
