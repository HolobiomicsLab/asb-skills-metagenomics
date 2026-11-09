---
name: signature-gene-multiple-sequence-alignment
description: Use when MAGinator has produced signature genes for a MAG cluster and
  per-sample BAMs, and you need SNV-resolution phylogenetic trees to stratify samples
  at subspecies/strain level or associate metadata with strain differences.
when_to_use_negative:
- Input is a per-sample abundance table or StrainPhlAn marker-based profile rather
  than MAGinator signature genes with mapped BAMs — use marker-gene tools like MetaPhlAn/StrainPhlAn
  instead
- Only a single sample (or fewer samples than needed to meet the --min_mapped_signature_genes
  / --min_samples thresholds) is available — a cross-sample SNV tree cannot be constructed
- The goal is species- or genome-level phylogeny of bins (e.g. GTDB-tk/dRep placement),
  not within-MAG-cluster SNV-resolution tree — MAGinator's trees are rooted using
  marker genes within MAG clusters, not whole-genome trees
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: MAGinator
  role: Orchestrates the SNV collection, alignment, concatenation and tree inference
    steps via Snakemake, emitting phylo/trees/, phylo/stats.tab and phylo/stats_genes.tab
  repo: https://github.com/Russel88/MAGinator
- name: Samtools
  role: mpileup on per-sample BAMs to collect SNV information for each signature gene
    and marker gene of each MAG cluster
- name: MAFFT
  role: Aligns each signature gene across samples (including the outgroup and rooting
    marker genes) before concatenation
- name: FastTree
  role: Default fast and approximate maximum-likelihood tree inference per MAG cluster
- name: IQ-TREE
  role: Optional slower but more precise maximum-likelihood tree inference, toggled
    with --phylo iqtree
- name: Snakemake
  role: Work distribution engine running the phylogenetic analysis stage of the MAGinator
    pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:bf5719b3ec2778577779064eff6fa92d174104fb0d05896e2a6ea3d4d0c1d6f1
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
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# signature-gene-multiple-sequence-alignment

## Summary

Aligns per-sample SNV-bearing signature genes of each MAG cluster with MAFFT and concatenates the per-gene alignments so that maximum-likelihood phylogenies (FastTree or IQ-TREE) can be built with leaves corresponding to samples, enabling SNV-resolution subspecies/strain stratification of MAGs.

## When to use

Use when MAGinator has produced signature genes for a MAG cluster and per-sample BAMs, and you need SNV-resolution phylogenetic trees to stratify samples at subspecies/strain level or associate metadata with strain differences.

## When NOT to use

- Input is a per-sample abundance table or StrainPhlAn marker-based profile rather than MAGinator signature genes with mapped BAMs — use marker-gene tools like MetaPhlAn/StrainPhlAn instead
- Only a single sample (or fewer samples than needed to meet the --min_mapped_signature_genes / --min_samples thresholds) is available — a cross-sample SNV tree cannot be constructed
- The goal is species- or genome-level phylogeny of bins (e.g. GTDB-tk/dRep placement), not within-MAG-cluster SNV-resolution tree — MAGinator's trees are rooted using marker genes within MAG clusters, not whole-genome trees

## Inputs

- per-sample BAM files under mapped_reads/bams/ from mapping reads to the MAG cluster contigs
- signature genes identified de novo per MAG cluster (refined by statistical modelling)
- marker genes pre-identified for rooting each phylogeny
- outgroup sequences
- sample detection counts gated by --min_mapped_signature_genes and an allele frequency cutoff

## Outputs

- concatenated MAFFT alignment of signature genes (plus marker genes/outgroup) per MAG cluster
- maximum-likelihood phylogenetic tree per MAG cluster (leaf = sample) written to phylo/trees/
- per-sample mixed-allele statistics in phylo/stats.tab (non-N fraction, read depth, number of bases not reaching allele frequency cutoff)
- per-gene breakdown in phylo/stats_genes.tab

## How to apply

First run samtools mpileup on the per-sample BAMs under mapped_reads/bams/ to collect SNV information for each signature gene and marker gene of each MAG cluster. Apply the inclusion criteria: keep only samples passing --min_mapped_signature_genes and exclude bases not reaching the allele frequency cutoff, recording these statistics in phylo/stats.tab. Align each signature gene across samples with MAFFT, including the pre-identified outgroup and marker genes used for rooting, then concatenate the per-gene MAFFT alignments into a single concatenated alignment per MAG cluster. Infer maximum-likelihood trees per MAG cluster with FastTree (default, fast and approximate) or IQ-TREE (--phylo iqtree, slow and precise), rooting using the marker genes, and write trees to phylo/trees/ with per-sample mixed-allele statistics in phylo/stats.tab and per-gene breakdowns in phylo/stats_genes.tab.

## Related tools

- **MAGinator** (Orchestrates the SNV collection, alignment, concatenation and tree inference steps via Snakemake, emitting phylo/trees/, phylo/stats.tab and phylo/stats_genes.tab) — https://github.com/Russel88/MAGinator
- **Samtools** (mpileup on per-sample BAMs to collect SNV information for each signature gene and marker gene of each MAG cluster)
- **MAFFT** (Aligns each signature gene across samples (including the outgroup and rooting marker genes) before concatenation)
- **FastTree** (Default fast and approximate maximum-likelihood tree inference per MAG cluster)
- **IQ-TREE** (Optional slower but more precise maximum-likelihood tree inference, toggled with --phylo iqtree)
- **Snakemake** (Work distribution engine running the phylogenetic analysis stage of the MAGinator pipeline)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180
```

## Evaluation signals

- Each tree leaf corresponds to a sample and every tree is built from SNVs within the MAG cluster's signature genes (not whole-genome alignments)
- Samples failing the --min_mapped_signature_genes threshold are absent from the tree; bases below the allele frequency cutoff are excluded and counted in phylo/stats.tab
- phylo/stats_genes.tab per-gene breakdown is consistent with the concatenated-alignment composition per MAG cluster
- Trees in phylo/trees/ are rooted via the marker genes included in the MAFFT alignment
- Runs with --phylo iqtree should agree in topology with FastTree runs while being more precise; test-data output on Zenodo (10.5281/zenodo.8279036) can serve as a reference diff

## Limitations

- Mixed alleles within a sample create uncertainty: per-sample non-N fraction and number of bases not reaching the allele frequency cutoff are reported, since a sample can harbour multiple strains (e.g. honey-bee gut shows 86% MAG clusters with mixed alleles vs. 46% in COPSAC2010)
- Only samples meeting the signature-gene detection threshold are included, so low-depth samples are dropped from the phylogeny
- FastTree is approximate; switching to IQ-TREE trades runtime for precision

## Evidence

- [other] MAGinator presents each per-MAG-cluster phylogeny as a maximum-likelihood tree built from sample SNVs within the signature genes, with each leaf corresponding to a sample.: [evidence span withheld — anchor status: not_found]
- [methods] Infer maximum-likelihood phylogenetic trees for each MAG cluster with FastTree (default, fast and approximate) or IQ-TREE (--phylo iqtree, slow and precise), rooting using the marker genes.: [evidence span withheld — anchor status: not_found]
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes.: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [readme] MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences: "SNV-level resolution phylogenetic trees based on signature genes"
