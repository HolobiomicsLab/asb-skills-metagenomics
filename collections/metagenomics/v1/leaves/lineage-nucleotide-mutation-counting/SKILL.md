---
name: lineage-nucleotide-mutation-counting
description: Use when a FASTQ wastewater (or similar) SARS-CoV-2 sample has been mapped
  and multiple variant-caller VCFs exist, and you must quantify Nt mutations per lineage
  to compare caller sensitivity or infer lineage composition.
when_to_use_negative:
- The input is already a per-lineage mutation count table or collapsed amino-acid
  substitution table — this skill operates on raw caller VCFs.
- You need amino-acid-level collapsed results (gene + amino acid substitution) for
  clustering rather than per-caller nucleotide mutation counting.
- The sequencing data are clinical isolates rather than low-frequency mixed wastewater
  populations, where a single standard caller suffices and caller comparison is unnecessary.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: lineagespot
  role: R framework run on each caller's VCF to identify SARS-CoV-2 lineages and count
    unique nucleotide mutations (Nt) per lineage
  repo: github.com/BiodataAnalysisGroup/lineagespot
- name: freebayes
  role: Variant caller run with a low mutation frequency of 0.01 to capture variants
    below 1% frequency; judged the most productive and informative
- name: GATK Mutect2
  role: Variant caller run in cancer-only mode with default parameters for comparison
    against freebayes and mpileup
- name: mpileup
  role: Variant caller configured to report every position for comparison against
    freebayes and GATK Mutect2
- name: Minimap2
  role: Maps cleaned reads to NC_045512 with minimal chaining score of 40 before variant
    calling
- name: R
  role: Language in which lineagespot is implemented and through which the Nt counting
    and tabulation are performed
  repo: github.com/BiodataAnalysisGroup/lineagespot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:a553cd778a03d1ec832eba2951a9e7a9cec0a2f96395cd4718f42dbecc35a494
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GATK Mutect2
  - Minimap2
  - R
  - freebayes
  - lineagespot
  - mpileup
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lineage-nucleotide-mutation-counting

## Summary

Counts unique lineage-characteristic nucleotide mutations (Nt) per SARS-CoV-2 lineage (e.g., B.1.177, B.1.1.7, B.1.351) by running the lineagespot R pipeline on VCFs from competing variant callers, enabling a side-by-side comparison that identified freebayes (low mutation frequency 0.01) as the most productive and informative tool.

## When to use

Use when a FASTQ wastewater (or similar) SARS-CoV-2 sample has been mapped and multiple variant-caller VCFs exist, and you must quantify Nt mutations per lineage to compare caller sensitivity or infer lineage composition.

## When NOT to use

- The input is already a per-lineage mutation count table or collapsed amino-acid substitution table — this skill operates on raw caller VCFs.
- You need amino-acid-level collapsed results (gene + amino acid substitution) for clustering rather than per-caller nucleotide mutation counting.
- The sequencing data are clinical isolates rather than low-frequency mixed wastewater populations, where a single standard caller suffices and caller comparison is unnecessary.

## Inputs

- FASTQ reads of a wastewater sample (ENA study PRJEB44141 / ERP128154)
- SARS-CoV-2 Wuhan reference genome NC_045512
- VCF files from freebayes, GATK Mutect2, and mpileup variant calling
- Lineage definitions from Pangolin rules or outbreak.info

## Outputs

- Per-caller lineage assignment list with unique nucleotide mutation (Nt) counts per lineage
- variant_caller_comparison.csv — Nt counts for B.1.177, B.1.1.7 and B.1.351 side-by-side per caller

## How to apply

Download the FASTQ reads of the target wastewater sample (05–11 February 2021 sample, ENA study PRJEB44141/ERP128154) and map cleaned reads to the SARS-CoV-2 Wuhan reference NC_045512 with Minimap2 using a minimal chaining score (matching bases minus log gap penalty) of 40, following the Illumina ARTIC pipeline variation. Call variants on the same alignments with three callers: freebayes with a low mutation frequency parameter of 0.01 (to capture variants below 1% frequency), GATK Mutect2 in cancer-only mode (default parameters), and mpileup configured to report every position. Run the lineagespot R pipeline on each caller's VCF to assign lineages via Pangolin rules / outbreak.info lineage definitions and count unique nucleotide mutations (Nt) per lineage. Tabulate Nt counts for B.1.177, B.1.1.7 and B.1.351 side-by-side per caller and select the caller with the highest productivity; the paper concluded freebayes is the most productive and informative and all subsequent results rely on freebayes output.

## Related tools

- **lineagespot** (R framework run on each caller's VCF to identify SARS-CoV-2 lineages and count unique nucleotide mutations (Nt) per lineage) — github.com/BiodataAnalysisGroup/lineagespot
- **freebayes** (Variant caller run with a low mutation frequency of 0.01 to capture variants below 1% frequency; judged the most productive and informative)
- **GATK Mutect2** (Variant caller run in cancer-only mode with default parameters for comparison against freebayes and mpileup)
- **mpileup** (Variant caller configured to report every position for comparison against freebayes and GATK Mutect2)
- **Minimap2** (Maps cleaned reads to NC_045512 with minimal chaining score of 40 before variant calling)
- **R** (Language in which lineagespot is implemented and through which the Nt counting and tabulation are performed) — github.com/BiodataAnalysisGroup/lineagespot

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- Reproduced per-caller Nt counts for B.1.177, B.1.1.7 and B.1.351 match the paper's variant-caller comparison table.
- freebayes (mutation frequency 0.01) yields the highest Nt counts across the three lineages, supporting the conclusion that it is the most productive caller.
- Each caller's VCF was produced from the same Minimap2 alignments (minimal chaining score 40) so differences in Nt reflect the caller, not the alignment.
- Lineage assignments derive from Pangolin rules / outbreak.info definitions and the Nt counts represent unique nucleotide mutations per lineage.

## Limitations

- Pangolin-based lineage assignment relies on an assumption that can produce incorrect assignments when different reads satisfy different rules of the same lineage.
- Mutation identification may be hindered by viral adsorption and decay in wastewater.
- Software versions for Minimap2, iVar, freebayes, and SnpEff were not reported, so exact reproduction may require parameter tuning.

## Evidence

- [results] freebayes most productive conclusion: "we consider that the most productive and informative approach is to utilize freebayes as the variant calling tool"
- [results] three callers assessed: "three different variant callers were assessed: (1) freebayes14, (2) mpileup15 and (3) GATK Mutect216 (cancer only mode)"
- [discussion] Minimap2 mapping parameter: "using Minimap2 tool19 with a minimal chaining score (matching bases minus log gap penalty) equal to 40"
- [intro] lineagespot implemented in R: "we propose a novel methodology called lineagespot, implemented as a software tool that can facilitate the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [readme] lineagespot installation from GitHub: [evidence span withheld — anchor status: external_doc]
- [discussion] Pangolin lineage rule assumption: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
