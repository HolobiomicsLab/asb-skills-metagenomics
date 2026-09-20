---
name: metagenomic-strain-phylogeny-reconstruction
description: Use when you have per-MAG-cluster signature genes and per-sample read
  mappings and need phylogenetic trees whose leaves are samples, to associate metadata
  with subspecies/strain-level differences.
when_to_use_negative:
- The goal is species-level taxonomic composition only — marker-gene mappers such
  as MetaPhlAn, MetaPhyler, or mOTUs suffice and no MAG clusters exist.
- Samples are not binned into MAG clusters (e.g. no VAMB clusters.tsv) or too few
  samples carry the MAG cluster to meet the --min_samples signature-gene refinement
  requirement.
- Samples fall below the --min_mapped_signature_genes detection threshold, leaving
  too few informative SNV positions for a reliable tree.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Orchestrates the SNV phylogeny module within its Snakemake pipeline, applying
    sample-inclusion and allele-frequency filters and emitting trees to phylo/trees/
  repo: https://github.com/Russel88/MAGinator
- name: Samtools
  role: mpileup-based collection of SNV information per signature gene and marker
    gene from per-sample BAM files
- name: MAFFT
  role: Aligns each signature gene across samples, including outgroup and marker genes,
    before concatenation
- name: FastTree
  role: Default fast, approximate maximum-likelihood tree inference per MAG cluster
- name: IQ-TREE
  role: Slower, more precise maximum-likelihood tree inference, enabled with --phylo
    iqtree
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:68cefc4dde4057a75607d92a153eabbb1ff13ae8b849fee8b781aa83b967817a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - FastTree
  - IQ-TREE
  - MAFFT
  - MAGinator
  - Samtools
  - Snakemake
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# metagenomic-strain-phylogeny-reconstruction

## Summary

Constructs SNV-resolution maximum-likelihood phylogenies for each MAG cluster from sample-level single nucleotide variants in its de novo signature genes, enabling subspecies/strain-level stratification of metagenome-assembled genomes.

## When to use

Use when you have per-MAG-cluster signature genes and per-sample read mappings and need phylogenetic trees whose leaves are samples, to associate metadata with subspecies/strain-level differences.

## When NOT to use

- The goal is species-level taxonomic composition only — marker-gene mappers such as MetaPhlAn, MetaPhyler, or mOTUs suffice and no MAG clusters exist.
- Samples are not binned into MAG clusters (e.g. no VAMB clusters.tsv) or too few samples carry the MAG cluster to meet the --min_samples signature-gene refinement requirement.
- Samples fall below the --min_mapped_signature_genes detection threshold, leaving too few informative SNV positions for a reliable tree.

## Inputs

- Per-sample BAM files of reads mapped to MAG contigs (mapped_reads/bams/)
- Per-MAG-cluster signature gene sequences and marker genes (pre-identified de novo via statistical refinement)
- Pre-identified outgroup sequences for alignment
- Sample inclusion threshold setting --min_mapped_signature_genes and allele frequency cutoff parameters

## Outputs

- Per-MAG-cluster maximum-likelihood phylogenetic trees (sample leaves) in phylo/trees/
- Per-sample mixed-allele statistics table phylo/stats.tab (non-N fraction, read depth, bases not reaching allele frequency cutoff)
- Per-gene SNV breakdown table phylo/stats_genes.tab

## How to apply

Run Samtools mpileup on the per-sample BAM files under mapped_reads/bams/ to collect SNV information for each signature gene and marker gene of each MAG cluster. Include only samples passing the --min_mapped_signature_genes detection threshold and exclude bases not reaching the allele frequency cutoff, recording these mixed-allele statistics (non-N fraction, read depth, bases below cutoff) in phylo/stats.tab and phylo/stats_genes.tab. Align each signature gene across samples with MAFFT, including the pre-identified outgroup and the marker genes used for rooting, then concatenate the per-gene alignments into a single alignment per MAG cluster. Infer maximum-likelihood trees rooted via the marker genes using FastTree (fast, approximate default) or IQ-TREE (--phylo iqtree, slower and more precise), writing results to phylo/trees/. Each tree leaf corresponds to a sample, so metadata can be projected directly onto tips.

## Related tools

- **MAGinator** (Orchestrates the SNV phylogeny module within its Snakemake pipeline, applying sample-inclusion and allele-frequency filters and emitting trees to phylo/trees/) — https://github.com/Russel88/MAGinator
- **Samtools** (mpileup-based collection of SNV information per signature gene and marker gene from per-sample BAM files)
- **MAFFT** (Aligns each signature gene across samples, including outgroup and marker genes, before concatenation)
- **FastTree** (Default fast, approximate maximum-likelihood tree inference per MAG cluster)
- **IQ-TREE** (Slower, more precise maximum-likelihood tree inference, enabled with --phylo iqtree)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- phylo/trees/ contains one maximum-likelihood tree per MAG cluster, each rooted via marker genes and with leaves corresponding to samples.
- phylo/stats.tab reports non-N fraction, read depth, and number of bases not reaching the allele frequency cutoff for every included sample.
- phylo/stats_genes.tab provides the per-gene SNV breakdown consistent with the concatenated MAFFT alignment.
- All samples present in a tree meet the --min_mapped_signature_genes detection threshold; excluded samples are absent from the tree.
- Trees inferred with IQ-TREE (--phylo iqtree) should be topologically consistent with FastTree defaults on the same concatenated alignment.

## Limitations

- Mixed alleles within samples are handled by excluding bases not reaching the allele frequency cutoff, but mixed strain populations reduce per-sample phylogenetic signal (e.g. 86% of MAG clusters show mixed alleles in the honey-bee gut dataset vs 46% in COPSAC2010).
- SNV trees are used for additional stratification of MAGs and depend on de novo signature genes; a subset of signature genes defined in one cohort may be missing in another, causing underestimation in cross-cohort transfer.
- IQ-TREE (--phylo iqtree) is slow relative to the default FastTree inference, a trade-off between precision and runtime.

## Evidence

- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes. They are used for additional stratification of the MAGs: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [results] From the COPSAC2010 cohort we demonstrated MAGinator’s ability to create SNV-level trees based on the sequences from the signature genes of a MAG cluster: "From the COPSAC2010 cohort we demonstrated MAGinator’s ability to create SNV-level trees based on the sequences from the signature genes of a MAG cluster"
- [readme] SNV-level resolution phylogenetic trees based on signature genes — MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences: "MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences"
