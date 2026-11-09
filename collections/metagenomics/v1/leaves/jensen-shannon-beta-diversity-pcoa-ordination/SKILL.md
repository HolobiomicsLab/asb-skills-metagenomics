---
name: jensen-shannon-beta-diversity-pcoa-ordination
description: Use when you have two or more compositional taxa abundance matrices (e.g.
  a reference MetaPhlAn2 profile and a de novo MAG-based profile) for the same samples
  with binary metadata (IBD vs control) and need to quantify whether one resolution
  better separates the groups via beta diversity.
when_to_use_negative:
- Input is a raw gene count matrix (gene_count_matrix.tsv) for functional profiling
  rather than a compositional taxa table for beta diversity
- Samples lack binary group metadata (e.g. IBD vs control), so PERMANOVA separation
  testing is not meaningful
- The abundance tables come from different sample sets and cannot be directly compared
  with the original reference profile
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Produces the de novo MAG-based abundance matrices (abundance_phyloseq.RData,
    relative_abundance_phyloseq.RData) whose beta diversity is compared against the
    MetaPhlAn2 baseline
  repo: https://github.com/Russel88/MAGinator.git
- name: MetaPhlAn
  role: Provides the original marker-gene-based compositional profile used as the
    reference matrix in the beta diversity comparison
- name: phyloseq
  role: Format of the MAGinator abundance outputs (phyloseq RData objects) used for
    ordination and PERMANOVA in R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:6e8ba28d7b90a0aaa5055c96ff458c38d34112fa552eb18621d5ad1c8005ddcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator.git
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - conda
  - mamba
  - phyloseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Jensen-Shannon beta diversity PCoA ordination

## Summary

Computes Jensen-Shannon beta diversity on compositional abundance matrices, ordinate them with PCoA, and tests group separation with PERMANOVA. Used in the MAGinator paper to re-analyse the Franzosa et al. IBD study and compare the original MetaPhlAn2 profile against MAGinator-derived and filtered MAGinator profiles.

## When to use

Use when you have two or more compositional taxa abundance matrices (e.g. a reference MetaPhlAn2 profile and a de novo MAG-based profile) for the same samples with binary metadata (IBD vs control) and need to quantify whether one resolution better separates the groups via beta diversity.

## When NOT to use

- Input is a raw gene count matrix (gene_count_matrix.tsv) for functional profiling rather than a compositional taxa table for beta diversity
- Samples lack binary group metadata (e.g. IBD vs control), so PERMANOVA separation testing is not meaningful
- The abundance tables come from different sample sets and cannot be directly compared with the original reference profile

## Inputs

- Original MetaPhlAn2 species-level abundance matrix for the 216 Franzosa et al. IBD samples
- MAGinator relative abundance matrix (relative_abundance_phyloseq.RData from the MAGinator run)
- Filtered MAGinator taxa abundance matrix (low-prevalence/noisy taxa removed)
- Sample metadata with IBD patient vs healthy control labels

## Outputs

- Jensen-Shannon dissimilarity matrix per abundance matrix
- PCoA ordination plots/coordinates per matrix
- PERMANOVA F and R² values per matrix (target: F=10.82/R²=4.81% MetaPhlAn2, F=10.21/R²=4.55% MAGinator, F=11.23/R²=4.99% filtered)
- Summary table, e.g. ibd_permanova_da_results.tsv

## How to apply

For each abundance matrix (original MetaPhlAn2, unfiltered MAGinator, and filtered MAGinator taxa table), compute a Jensen-Shannon dissimilarity matrix across samples and ordinate with PCoA. Then run PERMANOVA on each dissimilarity matrix using the IBD-vs-control grouping to obtain F and R² values. In the Franzosa et al. reanalysis the targets were F=10.82/R²=4.81% for MetaPhlAn2, F=10.21/R²=4.55% for MAGinator, and F=11.23/R²=4.99% for the filtered MAGinator matrix, with beta diversity comparable to the original analysis. Apply the filtered MAGinator matrix (taxa filtered, e.g. via --min_mapped_signature_genes-style filtering) to reduce noise introduced by MAGinator's high completeness. Tabulate the three F/R² pairs and the validated taxa counts and save them, e.g. as ibd_permanova_da_results.tsv.

## Related tools

- **MAGinator** (Produces the de novo MAG-based abundance matrices (abundance_phyloseq.RData, relative_abundance_phyloseq.RData) whose beta diversity is compared against the MetaPhlAn2 baseline) — https://github.com/Russel88/MAGinator.git
- **MetaPhlAn** (Provides the original marker-gene-based compositional profile used as the reference matrix in the beta diversity comparison)
- **phyloseq** (Format of the MAGinator abundance outputs (phyloseq RData objects) used for ordination and PERMANOVA in R)

## Evaluation signals

- PERMANOVA F and R² should fall near the reported targets: F=10.82/R²=4.81% (MetaPhlAn2), F=10.21/R²=4.55% (MAGinator), F=11.23/R²=4.99% (filtered)
- The filtered MAGinator matrix should yield the highest PERMANOVA F of the three matrices
- MAGinator-based beta diversity should be comparable to the original Franzosa et al. analysis
- All three matrices must be computed on the same 216 samples to make F/R² values directly comparable
- PCoA ordination of each matrix should visually reproduce the IBD-vs-control separation reported in the reanalysis

## Limitations

- MAGinator's high completeness may introduce a certain level of noise or misclassification, which influences the estimation of beta diversity — hence the need for the filtered MAGinator matrix
- Abundance estimates from de novo signature genes can underestimate taxa when signature genes are missing in a cohort (as seen in the COPSAC2010-to-CHILD cross-cohort transfer)
- PERMANOVA F/R² comparisons are only valid against the same sample set and grouping definition used in the original study

## Evidence

- [other] Compute Jensen-Shannon beta diversity on the original MetaPhlAn2 matrix, the MAGinator matrix, and the filtered MAGinator matrix, and ordinate with PCoA: [evidence span withheld — anchor status: not_found]
- [other] PERMANOVA target values per matrix: [evidence span withheld — anchor status: not_found]
- [discussion] Beta diversity comparability to Franzosa et al.: "the beta diversity was comparable to the analysis carried out by Franzosa et al. Reanalysing their data demonstrates how MAGinator can be used for a metagenomic association study."
- [discussion] Noise from high completeness affects beta diversity: "However, it may also introduce a certain level of noise or misclassification, which influences the estimation of beta diversity."
- [intro] IBD reanalysis demonstrates the method: "we show the functionality of MAGinator on a public dataset of inflammatory bowel disease (IBD) patients"
