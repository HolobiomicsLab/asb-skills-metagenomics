---
name: signature-gene-abundance-estimation
description: Use when shotgun metagenomics samples have been assembled and binned
  into VAMB clusters/MAGs and you need accurate abundance profiles (and subspecies-level
  stratification) of those de novo MAGs rather than database-based marker gene profiling.
when_to_use_negative:
- Input is already a taxonomic abundance table (e.g. MetaPhlAn2 output) — re-profiling
  with MAGinator is unnecessary unless de novo MAG-level resolution is required
- Only a few samples are available — signature gene refinement depends on the MAG
  cluster being present in enough samples (--min_samples), so the statistical model
  may not be fittable
- The goal is a fast taxonomic profile from marker gene databases (MetaPhlAn, MetaPhyler,
  mOTUs) without the assembly/binning prerequisite
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Identifies de novo signature genes, fits the statistical model, and estimates
    absolute/relative abundances of MAG clusters from mapped reads
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Orchestrates the MAGinator pipeline and the recommended reads_to_bins.Snakefile
    preprocessing/assembly/binning workflow
- name: GTDB-tk
  role: Taxonomically annotates bins and calls open reading frames (ORFs) that feed
    the gene catalogue and signature gene identification
- name: VAMB
  role: Produces the clusters.tsv binning input whose clusters are grouped into metagenomic
    species and quantified
  repo: https://github.com/RasmussenLab/vamb
- name: OPAL
  role: Evaluates the resulting abundance profile (purity/completeness) against CAMI
    II ground truth
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:56c145e2649171bd0a327f70102da693142ffad013b05675ecd67ebaaee0d803
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - GTDB-tk
  - MAGinator
  - MetaPhlAn
  - OPAL
  - Snakemake
  - StrainPhlAn
  - VAMB
  - conda
  - mamba
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# signature-gene-abundance-estimation

## Summary

MAGinator's abundance-estimation step: identify a set of signature genes de novo from the MAG clusters in the dataset, refine them by statistical modelling, then map reads to those genes (with length/identity/min-map filters) to estimate per-sample abundances of MAG clusters at higher-than-species resolution.

## When to use

Use when shotgun metagenomics samples have been assembled and binned into VAMB clusters/MAGs and you need accurate abundance profiles (and subspecies-level stratification) of those de novo MAGs rather than database-based marker gene profiling.

## When NOT to use

- Input is already a taxonomic abundance table (e.g. MetaPhlAn2 output) — re-profiling with MAGinator is unnecessary unless de novo MAG-level resolution is required
- Only a few samples are available — signature gene refinement depends on the MAG cluster being present in enough samples (--min_samples), so the statistical model may not be fittable
- The goal is a fast taxonomic profile from marker gene databases (MetaPhlAn, MetaPhyler, mOTUs) without the assembly/binning prerequisite

## Inputs

- VAMB clusters.tsv file grouping contigs into bins
- Contigs FASTA with unique sequence names (e.g. all_assemblies.fasta from the reads_to_bins.Snakefile workflow)
- Comma-separated reads.csv formatted as SampleName,PathToForwardReads,PathToReverseReads
- GTDB-tk release 220 database (gtdbtk_r220_data.tar.gz)

## Outputs

- abundance_phyloseq.RData with absolute MAG-cluster abundance profiles
- relative_abundance_phyloseq.RData with relative abundance profiles
- gene_count_matrix.tsv gene count matrix
- Signature gene set per MAG cluster used for abundance estimation
- OPAL-compatible profiler output (when benchmarking against CAMI II ground truth)

## How to apply

Run MAGinator on the three required inputs (VAMB clusters.tsv, a contigs FASTA with unique names, and a comma-separated reads.csv with SampleName,PathToForwardReads,PathToReverseReads) after installing via conda/mamba and providing the GTDB-tk release 220 database with --gtdb_db. Bins are first size-filtered (--binsize), GTDB-tk annotates bins and calls ORFs, and VAMB clusters are grouped into metagenomic species (MGS). ORFs are clustered into a non-redundant gene catalogue; signature genes are identified directly from the data and refined according to statistical modelling (parameters --min_samples and --min_mapped_signature_genes control refinement and per-sample detection). Reads are mapped to the catalogue with filters --min_length (aligned basepairs), --min_identity, and --min_map, ambiguous reads are redistributed based on uniquely mapping reads, and the signature gene model is fitted to yield absolute and relative abundances exported as abundance_phyloseq.RData and relative_abundance_phyloseq.RData plus a gene_count_matrix.tsv.

## Related tools

- **MAGinator** (Identifies de novo signature genes, fits the statistical model, and estimates absolute/relative abundances of MAG clusters from mapped reads) — https://github.com/Russel88/MAGinator
- **Snakemake** (Orchestrates the MAGinator pipeline and the recommended reads_to_bins.Snakefile preprocessing/assembly/binning workflow)
- **GTDB-tk** (Taxonomically annotates bins and calls open reading frames (ORFs) that feed the gene catalogue and signature gene identification)
- **VAMB** (Produces the clusters.tsv binning input whose clusters are grouped into metagenomic species and quantified) — https://github.com/RasmussenLab/vamb
- **OPAL** (Evaluates the resulting abundance profile (purity/completeness) against CAMI II ground truth)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- On the CAMI strain-madness benchmark, expect genus abundance-estimation completeness of 99.8% and species completeness of 89.6% for MAGinator's profile
- Exported abundance_phyloseq.RData and relative_abundance_phyloseq.RData must contain per-sample abundance vectors for all MAG clusters passing --binsize filtering
- Cross-cohort transfer check: fitting COPSAC2010-derived B. infantis/B. longum signature genes should give MSE 103.95 (COPSAC2010) vs 878.09 (CHILD), and the two subspecies trajectories summed should explain 87% of MetaPhlAn B. longum variance
- Read mapping filter sanity: reads are kept only if they pass --min_length aligned basepairs, --min_identity percent identity, and --min_map percent mapped thresholds before counting
- Expected output can be compared to the reference output on the test set (samples dfc99c_A, f9d84e_A, 221641_A) published on Zenodo (doi:10.5281/zenodo.8279036)

## Limitations

- A subset of signature genes defined in one cohort (COPSAC2010) was missing in the CHILD cohort, presumably causing underestimation of abundance for a subset of samples
- High completeness of MAGinator may introduce noise or misclassification that influences the estimation of beta diversity
- Performance depends on the number of samples in which a MAG cluster is present for signature gene refinement (--min_samples)

## Evidence

- [intro] Signature gene identification and abundance estimation: "C readmapping and counting, D Signature gene identification and abundance estimations, E Phylogenetic analysis"
- [intro] Statistically refined de novo signature genes: "This is done by identifying a set of signature genes directly from the given data and refining them according to statistical modelling to pick the ideal set suitable for abundance estimation"
- [methods] Read-mapping filter parameters: "Use --min_length to filter for the minimum number of basepairs that must be aligned to keep a read"
- [methods] Signature gene detection threshold: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [readme] Statistical model for best genes: "MAGinator uses a statistical model to find the best genes for calculating accurate abundances"
- [discussion] Cross-cohort signature gene transfer limitation: "which presumably resulted in an underestimation of the abundance for a subset of the samples"
