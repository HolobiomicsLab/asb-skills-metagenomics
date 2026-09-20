---
name: metagenomic-read-mapping-and-counting
description: Use when shotgun metagenomic reads (FASTQ) must be quantified against
  de novo MAG clusters or signature gene sets to estimate organism/subspecies abundances
  per sample, e.g. tracking B. infantis and B. longum across the COPSAC2010 and CHILD
  infant cohorts.
when_to_use_negative:
- You only need standard species-level taxonomic profiles from marker genes — a marker-gene
  profiler alone (MetaPhlAn, MetaPhyler, mOTUs) suffices and MAGinator's MAG-based
  mapping is unnecessary overhead
- You have no MAG bins / VAMB clusters.tsv — MAGinator requires de novo MAG clusters,
  contigs, and reads as input
- The task is functional annotation rather than abundance profiling — use EggNOG mapper
  output instead of read mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: 'Performs the read-mapping and counting step: maps shotgun reads to the gene
    catalogue, redistributes ambiguous reads using a signature-reads profile, and
    estimates MAG/subspecies abundances'
  repo: https://github.com/Russel88/MAGinator
- name: Vamb
  role: Produces the clusters.tsv MAG clustering files that define the contig/gene
    targets reads are mapped against
  repo: https://github.com/RasmussenLab/vamb
- name: MetaPhlAn
  role: Species-level marker-gene profiles used as the reference (B. longum species
    abundance) for validating MAGinator subspecies abundance estimates
- name: StrainPhlAn
  role: Identifies the two mutually exclusive subspecies clusters used to cross-validate
    MAGinator-estimated subspecies abundances
- name: Snakemake
  role: Workflow engine underlying MAGinator's read-mapping, counting, and signature-gene
    abundance estimation steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:668645432263f581c33e5ad8acb680b75301898b3ac9da80bf55f35da02eb3da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - MAGinator
  - MetaPhlAn
  - Snakemake
  - StrainPhlAn
  - Vamb
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metagenomic-read-mapping-and-counting

## Summary

Map shotgun metagenomic reads against a non-redundant gene catalogue / MAG-derived signature genes and count mapped reads to estimate per-sample abundances, as implemented in MAGinator's read-mapping and counting step. Ambiguous reads are redistributed using a signature-reads profile derived from uniquely mapping reads, enabling de novo subspecies-level abundance profiling.

## When to use

Use when shotgun metagenomic reads (FASTQ) must be quantified against de novo MAG clusters or signature gene sets to estimate organism/subspecies abundances per sample, e.g. tracking B. infantis and B. longum across the COPSAC2010 and CHILD infant cohorts.

## When NOT to use

- You only need standard species-level taxonomic profiles from marker genes — a marker-gene profiler alone (MetaPhlAn, MetaPhyler, mOTUs) suffices and MAGinator's MAG-based mapping is unnecessary overhead
- You have no MAG bins / VAMB clusters.tsv — MAGinator requires de novo MAG clusters, contigs, and reads as input
- The task is functional annotation rather than abundance profiling — use EggNOG mapper output instead of read mapping

## Inputs

- clusters.tsv from VAMB (RasmussenLab/vamb)
- FASTA file of all contig sequences with unique names (e.g. assembly/all_assemblies.fasta after sed 's/@/_/g')
- CSV file mapping SampleName,PathToForwardReads,PathToReverseReads for FASTQ reads
- GTDB-tk database release 220 (gtdbtk_r220_data.tar.gz)
- Non-redundant gene catalogue containing de novo signature gene sets (e.g. COPSAC2010-derived B. infantis and B. longum signature genes)

## Outputs

- Per-sample MAG/MGS abundance estimates from read counts
- Per-sample B. infantis / B. longum subspecies abundance trajectories from signature gene model fit
- Fit quality metrics (MSE per cohort: 103.95 COPSAC2010, 878.09 CHILD)
- bifido_subspecies_validation_results.tsv with MSE per cohort, R-squared, per-sample StrainPhlAn cluster and abundance assignments

## How to apply

Input MAGinator with VAMB clusters.tsv, a contigs FASTA with unique names, and a CSV mapping sample names to forward/reverse FASTQ paths; MAGinator filters bins by size (--binsize), runs GTDB-tk (release 220) for taxonomic annotation and ORF calling, groups VAMB clusters into metagenomic species (MGS), then performs readmapping and counting against the gene catalogue. Apply read filters via --min_length (minimum aligned basepairs to keep a read), --min_identity (minimum percent identity of a mapped read), and --min_map (minimum percent of a read that must be mapped). The signature-reads profile redistributes ambiguous reads across taxa based on uniquely mapping reads; downstream signature-gene abundance estimation requires a sample to detect at least --min_mapped_signature_genes signature genes to be included in the analysis. Fit the signature gene model per cohort, compute fit MSE (COPSAC2010 103.95 vs CHILD 878.09), and validate summed subspecies trajectories by regressing against MetaPhlAn B. longum species abundances (87% variance explained) and cross-tabulating with StrainPhlAn subspecies clusters.

## Related tools

- **MAGinator** (Performs the read-mapping and counting step: maps shotgun reads to the gene catalogue, redistributes ambiguous reads using a signature-reads profile, and estimates MAG/subspecies abundances) — https://github.com/Russel88/MAGinator
- **Vamb** (Produces the clusters.tsv MAG clustering files that define the contig/gene targets reads are mapped against) — https://github.com/RasmussenLab/vamb
- **MetaPhlAn** (Species-level marker-gene profiles used as the reference (B. longum species abundance) for validating MAGinator subspecies abundance estimates)
- **StrainPhlAn** (Identifies the two mutually exclusive subspecies clusters used to cross-validate MAGinator-estimated subspecies abundances)
- **Snakemake** (Workflow engine underlying MAGinator's read-mapping, counting, and signature-gene abundance estimation steps)

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g "/path/to/GTDB-Tk/database/release220/"
```

## Evaluation signals

- Recomputed signature-gene fit MSE matches 103.95 for COPSAC2010 and 878.09 for CHILD
- Regression of summed B. infantis + B. longum subspecies trajectories against MetaPhlAn B. longum species abundance explains 87% of variance
- StrainPhlAn's two mutually exclusive subspecies clusters separate samples by MAGinator-estimated subspecies abundance as in Fig. 4
- Per-sample abundance rows are only produced for samples passing --min_mapped_signature_genes (minimum number of detected signature genes)
- All metrics and per-sample assignments are written to bifido_subspecies_validation_results.tsv with expected columns (MSE, R-squared, cluster membership)

## Limitations

- A subset of signature genes defined in COPSAC2010 was missing in the CHILD cohort, presumably causing abundance underestimation for a subset of samples (CHILD fit MSE 878.09 vs COPSAC2010 103.95)
- High completeness of MAGinator may introduce noise or misclassification affecting beta diversity estimation
- Expected MAGinator output for the test set is version-dependent: the Zenodo reference output was generated with GTDB-tk db release207_v2, not release 220

## Evidence

- [intro] C readmapping and counting, D Signature gene identification and abundance estimations: "C readmapping and counting, D Signature gene identification and abundance estimations"
- [methods] Use --min_length to filter for the minimum number of basepairs that must be aligned to keep a read: "Use --min_length to filter for the minimum number of basepairs that must be aligned to keep a read"
- [methods] Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis: "Use --min_mapped_signature_genes to change minimum number of signature genes to be detected in the sample to be included in the analysis"
- [abstract] MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs).: "MAGinator provides de novo identification of subspecies-level microbes and accurate abundance estimates of metagenome-assembled genomes (MAGs)."
- [discussion] These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies.: "These results were reproduced in the CHILD cohort using the signature genes identified in COPSAC2010 for the two subspecies."
- [readme] MAGinator needs 3 input files: The clusters.tsv files from VAMB: "MAGinator needs 3 input files: * The clusters.tsv files from [VAMB](https://github.com/RasmussenLab/vamb)"
