---
name: wilcoxon-discovery-replication-differential-abundance
description: Use when comparing taxon abundances between two cohorts (e.g. IBD patients
  vs controls) across a discovery and an independent replication cohort to identify
  validated differentially abundant taxa from MAGinator-derived abundance tables.
when_to_use_negative:
- The analysis question is about community-wide beta diversity separation (e.g. Jensen-Shannon
  PCoA and PERMANOVA testing) rather than per-taxon differential abundance
- The goal is functional annotation (KEGG orthologs, GO terms) or synteny clustering
  of genes rather than taxon-level abundance testing
- Abundances come directly from a marker-gene database profiler (e.g. MetaPhlAn2)
  when the intent is to exploit MAGinator's subspecies-level resolution instead of
  replicating the original baseline
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Produces the signature-gene-based abundance matrices (abundance_phyloseq.RData,
    relative_abundance_phyloseq.RData) on which the Wilcoxon discovery/replication
    workflow is run
  repo: https://github.com/Russel88/MAGinator
- name: MetaPhlAn
  role: Original marker-gene-based profiling tool whose IBD taxa counts (18 validated)
    serve as the baseline against which the MAGinator-based Wilcoxon results are compared
- name: Snakemake
  role: Runs the recommended reads_to_bins.Snakefile preprocessing/assembly/binning
    workflow that generates the VAMB clusters and contigs feeding MAGinator
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:5ee62099ab57df26ea0e616cb8ca86e4afb757ec572e4c61ae6efef79e00050f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - conda
  - mamba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# wilcoxon-discovery-replication-differential-abundance

## Summary

A two-cohort Wilcoxon rank-sum discovery/replication workflow for identifying and validating differentially abundant taxa between IBD patients and controls, applied to MAGinator's high-phylogenetic-resolution abundance matrices instead of the original MetaPhlAn2 profiles.

## When to use

Use when comparing taxon abundances between two cohorts (e.g. IBD patients vs controls) across a discovery and an independent replication cohort to identify validated differentially abundant taxa from MAGinator-derived abundance tables.

## When NOT to use

- The analysis question is about community-wide beta diversity separation (e.g. Jensen-Shannon PCoA and PERMANOVA testing) rather than per-taxon differential abundance
- The goal is functional annotation (KEGG orthologs, GO terms) or synteny clustering of genes rather than taxon-level abundance testing
- Abundances come directly from a marker-gene database profiler (e.g. MetaPhlAn2) when the intent is to exploit MAGinator's subspecies-level resolution instead of replicating the original baseline

## Inputs

- abundance_phyloseq.RData (MAGinator signature-gene-based abundance matrix)
- relative_abundance_phyloseq.RData (relative-abundance taxa table)
- Sample metadata assigning samples to IBD patient and healthy-control cohorts for discovery and replication
- reads.csv sample index (SampleName,PathToForwardReads,PathToReverseReads) for the Franzosa et al. samples surviving assembly

## Outputs

- List of differentially abundant taxa between IBD patients and controls identified in the discovery cohort
- Count of taxa successfully validated in the independent replication cohort (target: 213 vs 18 in the original MetaPhlAn2 analysis)
- ibd_permanova_da_results.tsv tabulating validated taxa counts and PERMANOVA F/R² per matrix

## How to apply

Run the Wilcoxon two-cohort discovery/replication differential abundance workflow on the MAGinator taxa tables (abundance_phyloseq.RData and relative_abundance_phyloseq.RData produced from VAMB clusters.tsv, all_assemblies.fasta and the reads.csv sample index) to identify taxa differentially abundant between IBD patients and controls. Validate each discovery-cohort hit in an independent cohort to obtain the count of validated taxa. Rationale: MAGinator's de novo signature-gene abundances group MAGs into phylogenetically separated clusters at higher resolution than species, so the same Wilcoxon design recovers far more validated taxa (213) than the original MetaPhlAn2-based Franzosa et al. analysis (18). Tabulate the 18 discovery vs 213 validated taxa counts alongside the companion PERMANOVA F/R² values (MetaPhlAn2 F=10.82/R²=4.81%, MAGinator F=10.21/R²=4.55%, filtered F=11.23/R²=4.99%) and save as ibd_permanova_da_results.tsv.

## Related tools

- **MAGinator** (Produces the signature-gene-based abundance matrices (abundance_phyloseq.RData, relative_abundance_phyloseq.RData) on which the Wilcoxon discovery/replication workflow is run) — https://github.com/Russel88/MAGinator
- **MetaPhlAn** (Original marker-gene-based profiling tool whose IBD taxa counts (18 validated) serve as the baseline against which the MAGinator-based Wilcoxon results are compared)
- **Snakemake** (Runs the recommended reads_to_bins.Snakefile preprocessing/assembly/binning workflow that generates the VAMB clusters and contigs feeding MAGinator)

## Evaluation signals

- The number of taxa validated in the independent replication cohort should far exceed the original MetaPhlAn2 count of 18, reaching ~213 when run on MAGinator abundances
- PERMANOVA companion values should be reproducible: F=10.82/R²=4.81% (MetaPhlAn2), F=10.21/R²=4.55% (MAGinator), F=11.23/R²=4.99% (filtered MAGinator)
- Discovery and replication steps must use disjoint cohorts: only taxa significant in discovery that also pass Wilcoxon testing in the independent cohort count as validated
- Input tables must be the MAGinator signature-gene phyloseq objects (abundance_phyloseq.RData / relative_abundance_phyloseq.RData), not raw VAMB cluster counts, to ensure taxa are phylogenetically resolved above species level
- Final results should be written as ibd_permanova_da_results.tsv containing both taxa counts and the per-matrix F/R² values

## Limitations

- MAGinator's high completeness may introduce noise or misclassification that influences beta diversity estimation and could propagate into differential abundance calls
- A subset of signature genes defined in one cohort can be missing in an independent cohort, presumably causing abundance underestimation for a subset of samples and affecting cross-cohort validation of taxa
- Validation counts depend on how many samples survive assembly and preprocessing (216 Franzosa et al. samples), so cohort size and filtering thresholds directly affect the number of validatable taxa

## Evidence

- [intro] we show the functionality of MAGinator on a public dataset of inflammatory bowel disease (IBD) patients, where we identify differentially abundant taxa between patients and controls at high: "we show the functionality of MAGinator on a public dataset of inflammatory bowel disease (IBD) patients, where we identify differentially abundant taxa between patients and controls at high"
- [discussion] the beta diversity was comparable to the analysis carried out by Franzosa et al. Reanalysing their data demonstrates how MAGinator can be used for a metagenomic association study.: "Reanalysing their data demonstrates how MAGinator can be used for a metagenomic association study."
- [readme] MAGinator uses a statistical model to find the best genes for calculating accurate abundances: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
