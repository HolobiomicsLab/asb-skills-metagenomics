---
name: strainphylan-phylogenetic-stratification
description: Use when shotgun metagenomic MAG clusters must be resolved below species
  level — e.g. to separate B. infantis from B. longum — via SNV phylogenies from signature
  genes and abundance-based cluster validation.
when_to_use_negative:
- You only need species-level taxonomic composition — marker-gene profilers like MetaPhlAn,
  MetaPhyler, or mOTUs suffice without SNV phylogenies
- 'The signature gene set defined in one cohort does not transfer: a subset of COPSAC2010
  signature genes was missing in the CHILD cohort, which presumably resulted in an
  underestimation of abundance for a subset of samples'
- MAG clusters cannot be phylogenetically separated above the noise level; altering
  synteny/clustering parameters to remove noise comes at the expense of sensitivity
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Identifies de novo signature genes, estimates subspecies abundances via read
    mapping, and enables SNV-resolution phylogenetic trees from signature genes
  repo: https://github.com/Russel88/MAGinator
- name: StrainPhlAn
  role: Builds SNV-resolution phylogenies from signature genes to identify 2 mutually
    exclusive subspecies clusters
- name: MetaPhlAn
  role: Provides species-level B. longum abundances to which summed MAGinator subspecies
    trajectories are regressed (87% variance explained)
- name: Snakemake
  role: Implements and distributes the MAGinator workflow used to generate the inputs
    for stratification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:477aa577e893ec39f59d907fd3acd01bc61001ebf4f068088e99c012989c7f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - StrainPhlAn
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# StrainPhlAn phylogenetic stratification of MAGs at subspecies resolution

## Summary

Uses StrainPhlAn SNV-resolution phylogenetic trees built from de novo signature genes to split a MAG cluster into mutually exclusive subspecies clusters, then cross-validates the stratification against MAGinator-estimated subspecies abundances (e.g. B. infantis vs. B. longum in the COPSAC2010 and CHILD infant cohorts).

## When to use

Use when shotgun metagenomic MAG clusters must be resolved below species level — e.g. to separate B. infantis from B. longum — via SNV phylogenies from signature genes and abundance-based cluster validation.

## When NOT to use

- You only need species-level taxonomic composition — marker-gene profilers like MetaPhlAn, MetaPhyler, or mOTUs suffice without SNV phylogenies
- The signature gene set defined in one cohort does not transfer: a subset of COPSAC2010 signature genes was missing in the CHILD cohort, which presumably resulted in an underestimation of abundance for a subset of samples
- MAG clusters cannot be phylogenetically separated above the noise level; altering synteny/clustering parameters to remove noise comes at the expense of sensitivity

## Inputs

- MAGinator output for the MAG cluster: de novo signature gene sequences and per-sample signature-gene coverage/abundance estimates
- Shotgun metagenomic reads (FASTQ) from the cohort samples, e.g. COPSAC2010 and CHILD cohorts
- VAMB clusters.tsv, contigs FASTA, and reads.csv (SampleName,PathToForwardReads,PathToReverseReads) for running MAGinator
- MetaPhlAn species-level abundance profiles (e.g. B. longum) for variance-explained comparison

## Outputs

- StrainPhlAn SNV-resolution phylogenetic tree(s) per MAG cluster/split
- Per-sample assignment to mutually exclusive subspecies clusters (e.g. B. infantis vs. B. longum)
- Cross-tabulation of StrainPhlAn cluster membership vs. MAGinator subspecies abundances (Fig. 4-style stratification)
- Validation metrics table, e.g. bifido_subspecies_validation_results.tsv with MSE per cohort (COPSAC2010 103.95, CHILD 878.09) and R-squared (87%)

## How to apply

First run MAGinator to identify a de novo signature gene set for the MAG cluster of interest (refined by statistical modelling, requiring detection with --min_mapped_signature_genes) and to estimate per-sample subspecies abundances via read mapping with signature-reads redistribution of ambiguous reads. Then build StrainPhlAn phylogenetic trees from the SNVs of the signature gene sequences; with StrainPhlAn the samples resolve into 2 mutually exclusive clusters, each representing a subspecies. Assign each sample to one of the two StrainPhlAn clusters and cross-tabulate cluster membership against MAGinator-estimated subspecies abundances (as in Fig. 4), and regress the summed subspecies abundance trajectory against MetaPhlAn species-level abundance to quantify explained variance (87% for B. longum). The workflow is implemented in Snakemake; all metrics (per-sample cluster assignments and abundances) can be written to a results table such as bifido_subspecies_validation_results.tsv.

## Related tools

- **MAGinator** (Identifies de novo signature genes, estimates subspecies abundances via read mapping, and enables SNV-resolution phylogenetic trees from signature genes) — https://github.com/Russel88/MAGinator
- **StrainPhlAn** (Builds SNV-resolution phylogenies from signature genes to identify 2 mutually exclusive subspecies clusters)
- **MetaPhlAn** (Provides species-level B. longum abundances to which summed MAGinator subspecies trajectories are regressed (87% variance explained))
- **Snakemake** (Implements and distributes the MAGinator workflow used to generate the inputs for stratification)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- StrainPhlAn resolves exactly 2 mutually exclusive clusters for the B. infantis/B. longum MAG cluster, each representing a subspecies
- Cross-tabulation shows StrainPhlAn cluster membership separates samples by MAGinator-estimated subspecies abundance as in Fig. 4
- Summed subspecies abundance regressed against MetaPhlAn B. longum species abundance explains 87% of the variance
- Signature-gene fit MSE recomputes to 103.95 for COPSAC2010 and 878.09 for CHILD (better fit in the discovery cohort)
- Per-sample outputs in bifido_subspecies_validation_results.tsv are complete (cluster assignment plus subspecies abundances per sample)

## Limitations

- A subset of signature genes defined in COPSAC2010 was missing in the CHILD cohort, which presumably resulted in an underestimation of the abundance for a subset of the samples
- High completeness of MAGinator may introduce a certain level of noise or misclassification, which influences the estimation of beta diversity
- MAGinator's lack of operon awareness limits synteny cluster accuracy, and users eliminating synteny noise via parameter changes do so at the expense of sensitivity

## Evidence

- [discussion] With StrainPhlAn, we are able to identify 2 mutually exclusive clusters, each representing a subspecies.: "With StrainPhlAn, we are able to identify 2 mutually exclusive clusters, each representing a subspecies."
- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV's) resolution phylogenetic trees from the signature genes.: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes. They are used for additional stratification of the MAGs and can be"
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [discussion] which presumably resulted in an underestimation of the abundance for a subset of the samples: "which presumably resulted in an underestimation of the abundance for a subset of the samples"
