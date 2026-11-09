---
name: mag-abundance-profiling-from-shotgun-reads
description: Use when you have VAMB bins (clusters.tsv), their contigs FASTA, and
  the original paired-end FASTQ reads, and need accurate per-MAG-cluster abundance
  estimates plus subspecies-level phylogenetic stratification rather than a marker-gene-only
  profile.
when_to_use_negative:
- You only need a fast compositional profile from marker gene databases — mapping-based
  profilers like MetaPhlAn, MetaPhyler or mOTUs suffice and MAGinator's de novo MAG
  workflow is unnecessary.
- You have no MAG bins yet and no plan to assemble/bin — MAGinator requires VAMB clusters.tsv
  and contigs as input; run the reads_to_bins.Snakefile workflow (or your own assembly/binning)
  first.
- Your downstream question is single-sample taxonomic profiling with few samples —
  signature gene refinement requires multiple samples (--min_samples) and mixed-allele/noise
  behaviour at strain level may degrade beta diversity estimates (highest mixed-allele
  proportion observed was 86% in the honey-bee gut).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: 'Core pipeline: bin size filtering, GTDB-tk annotation/ORF calling, MGS grouping,
    gene clustering, read mapping/counting, signature gene model fitting, abundance
    export'
  repo: https://github.com/Russel88/MAGinator
- name: Vamb
  role: Produces the clusters.tsv bins and contig input via the reads_to_bins.Snakefile
    preprocessing/assembly/binning workflow
  repo: https://github.com/RasmussenLab/vamb
- name: GTDB-tk
  role: Taxonomically annotates bins and calls open reading frames (ORFs); release
    220 database
- name: OPAL
  role: Benchmarking the exported abundance profile against CAM II ground truth to
    obtain purity and completeness at genus and species ranks
- name: dRep
  role: Recommended post-binning step to create species-level clusters with modifiable
    clustering parameters before MAGinator setup
  repo: https://github.com/MrOlm/drep
- name: Snakemake
  role: Orchestrates the reads_to_bins preprocessing workflow and MAGinator's integrated
    work distribution on large-scale data or slurm/qsub clusters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:dcd0065b27f7ae948050171cdd654b00b52826a410685454d9bbd828b6072662
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - GTDB-tk
  - MAGinator
  - OPAL
  - Snakemake
  - Vamb
  - conda
  - dRep
  - mamba
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mag-abundance-profiling-from-shotgun-reads

## Summary

Profile the absolute and relative abundances of metagenome-assembled genomes (MAGs) from shotgun metagenomics reads using MAGinator's de novo signature-gene approach, combining gene- and contig-based methods to reach subspecies-level resolution. Validated on the CAMI II strain-madness dataset with OPAL purity/completeness metrics.

## When to use

Use when you have VAMB bins (clusters.tsv), their contigs FASTA, and the original paired-end FASTQ reads, and need accurate per-MAG-cluster abundance estimates plus subspecies-level phylogenetic stratification rather than a marker-gene-only profile.

## When NOT to use

- You only need a fast compositional profile from marker gene databases — mapping-based profilers like MetaPhlAn, MetaPhyler or mOTUs suffice and MAGinator's de novo MAG workflow is unnecessary.
- You have no MAG bins yet and no plan to assemble/bin — MAGinator requires VAMB clusters.tsv and contigs as input; run the reads_to_bins.Snakefile workflow (or your own assembly/binning) first.
- Your downstream question is single-sample taxonomic profiling with few samples — signature gene refinement requires multiple samples (--min_samples) and mixed-allele/noise behaviour at strain level may degrade beta diversity estimates (highest mixed-allele proportion observed was 86% in the honey-bee gut).

## Inputs

- VAMB clusters.tsv file (bin-to-contig assignments from the variational autoencoder binner)
- Contigs FASTA with unique sequence names (headers sanitized by replacing '@' with '_')
- Comma-separated reads CSV formatted SampleName,PathToForwardReads,PathToReverseReads
- GTDB-tk database (e.g. release 220, gtdbtk_r220_data.tar.gz)
- Optional: eggNOG-mapper annotations.tsv for functional annotation of gene clusters

## Outputs

- Absolute abundance profile (abundance/abundance_phyloseq.RData, phyloseq object)
- Relative abundance profile (relative_abundance_phyloseq.RData, phyloseq object)
- Metagenomic species (MGS) groupings of VAMB clusters with taxonomic consensus annotations
- Non-redundant gene catalogue and taxonomic scope for each gene cluster
- SNV-resolution phylogenetic trees built from signature genes
- Synteny clusters of genes within subspecies

## How to apply

First generate the three MAGinator inputs via the recommended reads_to_bins.Snakefile workflow: VAMB clusters.tsv, a contigs FASTA with unique names (strip '@' from headers with sed), and a comma-separated reads file (SampleName,PathToForwardReads,PathToReverseReads). Then run MAGinator with the GTDB-tk release 220 database: it filters bins by size (--binsize), taxonomically annotates bins and calls ORFs with GTDB-tk, groups VAMB clusters into metagenomic species (MGS) based on taxonomic annotation, clusters ORFs into gene clusters, maps reads back with length/identity/min-map filters (--min_length, --min_identity, --min_map), and fits a statistical model on de novo signature genes (--min_mapped_signature_genes, --min_samples) to estimate abundances. Export absolute and relative abundance profiles from abundance/abundance_phyloseq.RData and relative_abundance_phyloseq.RData (phyloseq objects), convert them to OPAL-compatible profiler format if benchmarking against CAMI ground truth. Validate by running OPAL at genus and species ranks: on the CAMI strain-madness benchmark MAGinator achieved genus completeness 99.8%, species completeness 89.6%, genus purity 80.1%, and species purity 90.1%.

## Related tools

- **MAGinator** (Core pipeline: bin size filtering, GTDB-tk annotation/ORF calling, MGS grouping, gene clustering, read mapping/counting, signature gene model fitting, abundance export) — https://github.com/Russel88/MAGinator
- **Vamb** (Produces the clusters.tsv bins and contig input via the reads_to_bins.Snakefile preprocessing/assembly/binning workflow) — https://github.com/RasmussenLab/vamb
- **GTDB-tk** (Taxonomically annotates bins and calls open reading frames (ORFs); release 220 database)
- **OPAL** (Benchmarking the exported abundance profile against CAM II ground truth to obtain purity and completeness at genus and species ranks)
- **dRep** (Recommended post-binning step to create species-level clusters with modifiable clustering parameters before MAGinator setup) — https://github.com/MrOlm/drep
- **Snakemake** (Orchestrates the reads_to_bins preprocessing workflow and MAGinator's integrated work distribution on large-scale data or slurm/qsub clusters)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db release220/ --output out --max_mem 180
```

## Evaluation signals

- OPAL purity/completeness vs CAMI II strain-madness ground truth: expect genus completeness ~99.8%, species completeness ~89.6%, genus purity ~80.1% (rank 5), species purity ~90.1% (best among compared profilers vs LSHVec's 100% genus / 37.5% species purity).
- Abundance phyloseq exports (abundance_phyloseq.RData and relative_abundance_phyloseq.RData) parse as valid phyloseq objects containing absolute and relative abundances.
- All bins below the --binsize cutoff are excluded before GTDB-tk annotation; VAMB clusters grouped into MGS have a consistent taxonomic consensus controlled by --annotation_prevalence.
- Mapped reads survive the triple filter: minimum aligned basepairs (--min_length), minimum percent identity (--min_identity), and minimum percent of read mapped (--min_map).
- Each MAG cluster's abundance is estimated from ≥ --min_mapped_signature_genes signature genes detected in ≥ --min_samples samples; clusters failing these thresholds should be absent from the profile.

## Limitations

- High completeness of abundance estimates may introduce noise or misclassification, which influences the estimation of beta diversity.
- Signature genes found de novo in one cohort (e.g. COPSAC2010) may be missing in another cohort (CHILD), presumably causing abundance underestimation for a subset of samples.
- Accuracy of synteny clusters is limited by MAGinator's lack of operon awareness.
- Users can eliminate synteny noise at the expense of sensitivity by altering user-modifiable parameters.
- Test-data runs required ~180 GB RAM per job (real 72m27s on a slurm server); scale --max_cores and --max_mem accordingly on smaller servers.

## Evidence

- [abstract] Here, we present MAGinator, a freely available tool, tailored for profiling of shotgun metagenomics datasets.: "Here, we present MAGinator, a freely available tool, tailored for profiling of shotgun metagenomics datasets."
- [intro] The fidelity of our estimated abundances is demonstrated on the CAMI strain-madness dataset, where we benchmark MAGinator against similar tools: "The fidelity of our estimated abundances is demonstrated on the Critical Assessment of Metagenome Interpretation (CAMI) strain-madness dataset, where we benchmark MAGinator against similar tools"
- [methods] All you need for running MAGinator is snakemake and mamba.: "All you need for running MAGinator is snakemake and mamba."
- [readme] MAGinator needs 3 input files: the clusters.tsv files from VAMB, a fasta file with sequences of all contigs, a comma-separated file giving the position of the fastq files: "A comma-separated file giving the position of the fastq files with your sequencing reads formatted as: SampleName,PathToForwardReads,PathToReverseReads"
- [discussion] However, it may also introduce a certain level of noise or misclassification, which influences the estimation of beta diversity.: "However, it may also introduce a certain level of noise or misclassification, which influences the estimation of beta diversity."
