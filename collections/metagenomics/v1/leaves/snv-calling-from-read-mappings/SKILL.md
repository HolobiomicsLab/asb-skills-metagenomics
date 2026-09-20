---
name: snv-calling-from-read-mappings
description: Use when per-sample BAMs of shotgun reads mapped to de novo signature
  genes are available and you need SNV-resolution phylogenies or mixed-allele statistics
  to stratify MAG clusters below species resolution.
when_to_use_negative:
- You only have raw FASTQ reads without prior assembly, VAMB binning (clusters.tsv),
  and read mapping — this skill operates downstream on per-sample BAMs against precomputed
  signature genes.
- The goal is reference-based composition profiling from marker-gene databases (MetaPhlAn,
  MetaPhyler, mOTUs) rather than de novo SNV phylogenies of your own MAG clusters.
- The MAG cluster is present in fewer samples than the --min_samples threshold used
  for signature-gene refinement, so no de novo signature-gene set exists to call SNVs
  against.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Snakemake pipeline that runs the mpileup SNV collection, applies --min_mapped_signature_genes
    and allele-frequency filters, and emits trees to phylo/trees/ plus stats to phylo/stats.tab
    and phylo/stats_genes.tab
  repo: https://github.com/Russel88/MAGinator.git
- name: Samtools
  role: mpileup over per-sample BAMs collects SNV information for each signature gene
    and marker gene of each MAG cluster
- name: MAFFT
  role: aligns each signature gene across samples (with outgroup and rooting marker
    genes) before concatenation
- name: FastTree
  role: default fast, approximate maximum-likelihood tree builder for the concatenated
    per-cluster alignments
- name: IQ-TREE
  role: precise (slow) maximum-likelihood tree inference selected with --phylo iqtree
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:2de15c8745d48b3c199e983157f75c7967a5d3d8d051c4a6eac49ec4bfb7ebce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator.git
  tools:
  - FastTree
  - IQ-TREE
  - MAFFT
  - MAGinator
  - Samtools
  - Snakemake
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# snv-calling-from-read-mappings

## Summary

Calls per-sample SNVs from read mappings against de novo MAGinator signature and marker genes (Samtools mpileup over per-sample BAMs), applies allele-frequency and --min_mapped_signature_genes thresholds, and concatenates MAFFT alignments for FastTree/IQ-TREE maximum-likelihood trees that stratify MAG clusters at subspecies/strain level.

## When to use

Use when per-sample BAMs of shotgun reads mapped to de novo signature genes are available and you need SNV-resolution phylogenies or mixed-allele statistics to stratify MAG clusters below species resolution.

## When NOT to use

- You only have raw FASTQ reads without prior assembly, VAMB binning (clusters.tsv), and read mapping — this skill operates downstream on per-sample BAMs against precomputed signature genes.
- The goal is reference-based composition profiling from marker-gene databases (MetaPhlAn, MetaPhyler, mOTUs) rather than de novo SNV phylogenies of your own MAG clusters.
- The MAG cluster is present in fewer samples than the --min_samples threshold used for signature-gene refinement, so no de novo signature-gene set exists to call SNVs against.

## Inputs

- Per-sample BAM files of reads mapped to MAGinator contigs/genes (mapped_reads/bams/)
- Per-MAG-cluster signature gene sequences plus the pre-identified outgroup and marker genes used for rooting
- Inclusion settings: --min_mapped_signature_genes (minimum detected signature genes per sample) and the allele-frequency cutoff

## Outputs

- Rooted maximum-likelihood phylogenetic tree per MAG cluster in phylo/trees/ (each leaf = a sample)
- phylo/stats.tab with per-sample non-N fraction, read depth, and number of bases not reaching the allele-frequency cutoff
- phylo/stats_genes.tab with the per-gene breakdown of mixed-allele statistics

## How to apply

Run Samtools mpileup over the per-sample BAM files under mapped_reads/bams/ to collect SNV information for each signature gene and marker gene of each MAG cluster. Apply the inclusion criteria: keep only samples passing the --min_mapped_signature_genes detection threshold and exclude bases not reaching the allele-frequency cutoff, writing per-sample mixed-allele statistics (non-N fraction, read depth, number of bases below the cutoff) to phylo/stats.tab. Align each signature gene across samples with MAFFT, adding the pre-identified outgroup and the marker genes reserved for rooting, then concatenate the per-gene alignments into a single alignment per MAG cluster. Infer a maximum-likelihood tree per MAG cluster with FastTree (default, fast and approximate) or IQ-TREE (--phylo iqtree, slow and precise), rooted via the marker genes, and write trees to phylo/trees/ with the per-gene breakdown in phylo/stats_genes.tab. The rationale is that SNVs within signature genes capture within-cluster strain-level variation, so each tree leaf is a sample and the tree supports subspecies-level stratification of MAGs.

## Related tools

- **MAGinator** (Snakemake pipeline that runs the mpileup SNV collection, applies --min_mapped_signature_genes and allele-frequency filters, and emits trees to phylo/trees/ plus stats to phylo/stats.tab and phylo/stats_genes.tab) — https://github.com/Russel88/MAGinator.git
- **Samtools** (mpileup over per-sample BAMs collects SNV information for each signature gene and marker gene of each MAG cluster)
- **MAFFT** (aligns each signature gene across samples (with outgroup and rooting marker genes) before concatenation)
- **FastTree** (default fast, approximate maximum-likelihood tree builder for the concatenated per-cluster alignments)
- **IQ-TREE** (precise (slow) maximum-likelihood tree inference selected with --phylo iqtree)

## Examples

```
maginator --vamb_clusters clusters.tsv --reads reads.csv --contigs contigs.fasta --gtdb_db data/release220/ --output test_out --max_mem 180 --phylo iqtree
```

## Evaluation signals

- phylo/stats.tab contains, per included sample, the non-N fraction, read depth, and number of bases not reaching the allele-frequency cutoff, and phylo/stats_genes.tab gives a consistent per-gene breakdown.
- Samples failing the --min_mapped_signature_genes detection threshold are excluded from the trees, and bases below the allele-frequency cutoff are excluded from the SNV calls.
- Exactly one rooted maximum-likelihood tree per MAG cluster is written to phylo/trees/, with leaves corresponding to samples and rooting determined by the marker genes.
- Default runs use FastTree; rerunning with --phylo iqtree on the same concatenated alignment should yield consistent, more precise topologies.
- For the three SRA test samples (dfc99c_A, f9d84e_A, 221641_A), the phylo/ outputs can be compared against the expected output archive on Zenodo (10.5281/zenodo.8279036).

## Limitations

- Mixed alleles within samples complicate strain stratification: the proportion of MAG clusters with mixed alleles varies widely across datasets (honey-bee gut 86%, Tara Oceans 68%, COPSAC2010 46%).
- FastTree is fast but approximate, while IQ-TREE is slow but precise — tree accuracy trades off against runtime.
- Signature genes are de novo and dataset-specific: a subset defined in COPSAC2010 was missing in the CHILD cohort, which presumably caused abundance underestimation for a subset of samples, so cross-cohort transfer of the signature-gene SNV trees is imperfect.

## Evidence

- [readme] SNV-level resolution phylogenetic trees based on signature genes: "SNV-level resolution phylogenetic trees based on signature genes"
- [readme] MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences: "MAGinator creates a phylogenetic tree for each species so you can associate your metadata with subspecies/strain level differences"
- [intro] MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes.: "MAGinator also enables the creation of Single Nucleotide Variant (SNV’s) resolution phylogenetic trees from the signature genes."
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [discussion] From the COPSAC2010 cohort we demonstrated MAGinator’s ability to create SNV-level trees based on the sequences from the signature genes of a MAG cluster: "From the COPSAC2010 cohort we demonstrated MAGinator’s ability to create SNV-level trees based on the sequences from the signature genes of a MAG cluster"
- [discussion] While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%): "While the honey-bee gut exhibits the highest proportion of MAG clusters with mixed alleles (86%)"
