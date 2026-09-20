---
name: lineagespot-output-interpretation
description: Use when NGS-derived variant calls (VCF) from SARS-CoV-2 wastewater samples
  must be converted into lineage-level inference, or when wastewater-inferred lineages
  need to be checked against patient-derived Pangolin lineages for the same area and
  time periods.
when_to_use_negative:
- Input is raw FASTQ reads rather than variant call files — lineagespot expects VCF
  input, and the raw-data preprocessing (iVar trimming, Minimap2 mapping to NC_045512)
  must be run first
- The goal is precise within-sample variant frequency quantification rather than community-level
  lineage inference — the Pangolin-rule-based assignment can mislabel lineages when
  reads satisfy different rules of the same lineage
- Clinical comparison lineages are unavailable or unmatched in time/space — the concordance
  assessment is a qualitative alignment with predominant lineages and cannot be judged
  without matched Pangolin-assigned clinical samples
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: lineagespot
  role: R framework that identifies SARS-CoV-2 mutations from VCF file(s) and infers
    lineage distributions from wastewater samples
  repo: https://github.com/BiodataAnalysisGroup/lineagespot
- name: Pangolin
  role: Source of the lineage definitions/rules used to map detected mutation profiles
    to SARS-CoV-2 lineages and to assign clinical sample lineages for comparison
- name: freebayes
  role: Variant calling tool judged most productive and informative for low-frequency
    (< 0.01) mutations, producing the VCF inputs consumed by lineagespot
- name: outbreak.info
  role: Pre-calculated lineage-characteristic mutation profiles used as an alternative
    lineage definition source
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-06625-6
    title: Detecting SARS-CoV-2 lineages and mutational load in municipal wastewater
      and a use-case in the metropolitan area of Thessaloniki, Greece
  source_content_hash: sha256:04d6f42bfa94e17c0471bc0ca89d5ab78d9319e2f6691c433b07ceb5311e3597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BiodataAnalysisGroup/lineagespot
  tools:
  - Pangolin
  - R
  - freebayes
  - lineagespot
  - outbreak.info
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lineagespot-output-interpretation

## Summary

Interpret lineagespot outputs — merged VCF-derived mutation tables collapsed at gene and amino acid substitution level — to infer SARS-CoV-2 lineage distributions in wastewater and qualitatively compare them with predominant clinical (Pangolin-assigned) lineages across matched time periods, as demonstrated on 14 Thessaloniki wastewater samples.

## When to use

Use when NGS-derived variant calls (VCF) from SARS-CoV-2 wastewater samples must be converted into lineage-level inference, or when wastewater-inferred lineages need to be checked against patient-derived Pangolin lineages for the same area and time periods.

## When NOT to use

- Input is raw FASTQ reads rather than variant call files — lineagespot expects VCF input, and the raw-data preprocessing (iVar trimming, Minimap2 mapping to NC_045512) must be run first
- The goal is precise within-sample variant frequency quantification rather than community-level lineage inference — the Pangolin-rule-based assignment can mislabel lineages when reads satisfy different rules of the same lineage
- Clinical comparison lineages are unavailable or unmatched in time/space — the concordance assessment is a qualitative alignment with predominant lineages and cannot be judged without matched Pangolin-assigned clinical samples

## Inputs

- VCF file(s) from low-frequency variant calling (freebayes) on ARTIC-based wastewater NGS reads mapped to NC_045512
- lineagespot R package (installed via devtools from BiodataAnalysisGroup/lineagespot)
- Pangolin lineage assignment rules / outbreak.info lineage-characteristic mutation profiles
- Patient-derived WGS lineage calls from the same area and time periods (ENA study PRJEB44141 / ERP128154)

## Outputs

- Combined table of detected nucleotide mutations with amino acid substitutions, allele frequency and overlapping gene
- Inferred SARS-CoV-2 lineage distribution per wastewater sample
- Per-period comparison table of predominant wastewater vs. clinical lineages (e.g. lineage_agreement.tsv) with an overall agreement rate

## How to apply

Run lineagespot on the variant files from the NGS pipeline (freebayes was the variant caller judged most productive for low-frequency mutations, detecting mutations at frequencies < 0.01), then merge the per-sample VCFs into a combined table of nucleotide mutations with amino acid substitutions, allele frequencies and overlapping genes. Collapse this table at the gene and amino acid substitution level to reduce noise introduced by nucleotide mutations mapping to the same amino acid change, and optionally cluster the collapsed substitutions (Euclidean distance, ward.D method) normalized by gene length. Lineage assignment uses lineage definitions retrieved as Pangolin rules or through outbreak.info pre-calculated profiles, so a sample's reads matching characteristic mutation rules yield the inferred lineage distribution. For concordance assessment, tabulate the predominant lineagespot-inferred wastewater lineage versus the predominant Pangolin-assigned clinical lineage for each of the 14 time periods and compute an overall agreement rate; the paper's benchmark is the recorded evolution and rapid domination of the Alpha variant (B.1.1.7) in the Thessaloniki community.

## Related tools

- **lineagespot** (R framework that identifies SARS-CoV-2 mutations from VCF file(s) and infers lineage distributions from wastewater samples) — https://github.com/BiodataAnalysisGroup/lineagespot
- **Pangolin** (Source of the lineage definitions/rules used to map detected mutation profiles to SARS-CoV-2 lineages and to assign clinical sample lineages for comparison)
- **freebayes** (Variant calling tool judged most productive and informative for low-frequency (< 0.01) mutations, producing the VCF inputs consumed by lineagespot)
- **outbreak.info** (Pre-calculated lineage-characteristic mutation profiles used as an alternative lineage definition source)

## Examples

```
devtools::install_github("BiodataAnalysisGroup/lineagespot")
```

## Evaluation signals

- The merged output table contains, per mutation: nucleotide change, amino acid substitution, allele frequency and overlapping gene
- Collapsing at gene and amino acid substitution level removes duplicate nucleotide entries mapping to the same amino acid change
- The inferred lineage trajectory reproduces the paper's benchmark: recording the evolution and rapid domination of the Alpha variant (B.1.1.7) across the 14 Thessaloniki time periods
- Predominant wastewater lineages roughly align with predominant Pangolin-assigned clinical lineages across the 14 periods, with a computed overall agreement rate
- Permutations (e.g. Student's t-test on mean allele frequency and F test on variances between replicates) show no significant replicate differences

## Limitations

- Pangolin-rule-based assignment relies on an assumption that can produce incorrect assignments when different groups of reads satisfy different rules of the same lineage
- Preprocessing was inconsistent across samples: for the four samples between 19 March 2021 and 15 April 2021, primer trimming and remapping to the reference genome were not applied
- Mutation identification may be hindered by viral adsorption and decay in wastewater
- Software versions for Minimap2, iVar, freebayes and SnpEff are not reported in the paper

## Evidence

- [abstract] we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS): "we propose a novel methodology, called lineagespot, for the monitoring of mutations and the detection of SARS-CoV-2 lineages in wastewater samples using next-generation sequencing (NGS)"
- [intro] the lineage assignments defined by Pangolin13, this method identifies all SARS-CoV-2 variants present in the wastewater: "the lineage assignments defined by Pangolin13, this method identifies all SARS-CoV-2 variants present in the wastewater"
- [results] Qualitative assessment between the major lineages found in the targeted clinical samples, retrieved from the ENA study ID: PRJEB44141 (ERP128154), and across the 14 time periods: "Qualitative assessment between the major lineages found in the targeted clinical samples, retrieved from the ENA study ID: PRJEB44141 (ERP128154), and across the 14 time periods"
- [results] the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition): "the lineage definition (retrieved either as a Pangolin rule or through the outbreak.info definition)"
- [discussion] Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned.: "Given a group of reads that satisfy a rule A of lineage L, and another group of reads that satisfy rule B from the same lineage L, then the lineage L is incorrectly assigned."
- [readme] aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s): "aims to identify SARS-CoV-2 related mutations based on a single (or a list) of variant(s) file(s)"
