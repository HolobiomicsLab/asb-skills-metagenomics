---
name: mag-cluster-taxonomic-annotation
description: Use when you have VAMB clusters.tsv plus assembled contigs and need taxonomic
  labels and MGS grouping of MAG clusters before abundance estimation, signature gene
  identification, or strain-level analysis with MAGinator.
when_to_use_negative:
- Your MAGs are already taxonomically labelled via a database-derived binner (e.g.
  TaxVamb/Taxometer) rather than de novo VAMB clusters
- You only need to re-cluster species representatives without per-bin ORF calling
  and taxonomic scope assignment
- Input contigs and clusters.tsv still contain '@' characters that have not been replaced,
  which the README's preparatory sed steps exist to fix
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MAGinator
  role: Runs GTDB-tk to taxonomically annotate size-filtered bins and calls ORFs;
    groups VAMB clusters into MGS using taxonomic consensus (--annotation_prevalence)
    and taxonomic scope consensus (--tax_scope_threshold)
  repo: https://github.com/Russel88/MAGinator
- name: GTDB-tk
  role: Reference database (release 220) used by MAGinator to taxonomically annotate
    bins before MGS grouping
- name: VAMB
  role: Produces the clusters.tsv bin assignments that MAGinator annotates and groups
    into MGS
  repo: https://github.com/RasmussenLab/vamb
- name: dRep
  role: Recommended to create species-level clusters with modifiable clustering parameters
    when MAGinator's own MGS grouping is insufficient; MAGinator_setup.R converts
    dRep output into MAGinator inputs
  repo: https://github.com/MrOlm/drep
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:5eb4620019d80670c0e7c8d2c8a0fc8b2925ee834c223bee20d81678e278f98e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - GTDB-tk
  - MAGinator
  - Snakemake
  - VAMB
  - dRep
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# MAG cluster taxonomic annotation

## Summary

Taxonomically annotate de novo MAGs with GTDB-tk (release 220) and group VAMB clusters into metagenomic species (MGS) using MAGinator's taxonomic consensus and scope rules, which underpins downstream abundance estimation and mixed-allele profiling.

## When to use

Use when you have VAMB clusters.tsv plus assembled contigs and need taxonomic labels and MGS grouping of MAG clusters before abundance estimation, signature gene identification, or strain-level analysis with MAGinator.

## When NOT to use

- Your MAGs are already taxonomically labelled via a database-derived binner (e.g. TaxVamb/Taxometer) rather than de novo VAMB clusters
- You only need to re-cluster species representatives without per-bin ORF calling and taxonomic scope assignment
- Input contigs and clusters.tsv still contain '@' characters that have not been replaced, which the README's preparatory sed steps exist to fix

## Inputs

- VAMB clusters.tsv (MAG-to-contig bin assignments)
- Fasta file of all contig sequences with unique names (e.g. all_assemblies.fasta, '@' replaced by '_')
- reads.csv manifest: SampleName,PathToForwardReads,PathToReverseReads
- GTDB-tk database release 220 (gtdbtk_r220_data.tar.gz)
- Optional: dRep species-level clusters and clusters.tsv for MAGinator_setup.R

## Outputs

- Taxonomically annotated MAG bins (GTDB-tk annotations of size-filtered bins)
- MAG clusters grouped into metagenomic species (MGS)
- ORF calls per bin feeding the non-redundant gene catalogue
- clusters.tsv augmented with taxonomic consensus and scope per gene cluster (for functional annotation merging)

## How to apply

Assemble the three MAGinator inputs (VAMB clusters.tsv, a contigs fasta with unique names, and a reads.csv manifest of fastq paths), first replacing '@' with '_' in both the assembly fasta and clusters.tsv as the README prescribes. Filter bins by size with --binsize, then MAGinator runs GTDB-tk against the release 220 database (gtdbtk_r220_data.tar.gz from data.ace.uq.edu.au) to taxonomically annotate bins and call ORFs. VAMB clusters are grouped into metagenomic species (MGS) based on the GTDB-tk annotation, with --annotation_prevalence controlling how prevalent an annotation must be within a VAMB cluster to call taxonomic consensus and --tax_scope_threshold toggling the taxonomic scope consensus used to assign a taxonomic level to gene clusters. For species-level clusters not recovered by MAGinator's MGS grouping, the README recommends dRep with adjustable clustering parameters (e.g. --clustering_min_seq_id, --clustering_coverage) and the MAGinator_setup.R helper script to build the input files. Correct application yields taxonomically labelled, size-filtered MAG clusters that feed the non-redundant gene catalogue and downstream phylo/stats.tab and phylo/pileup outputs.

## Related tools

- **MAGinator** (Runs GTDB-tk to taxonomically annotate size-filtered bins and calls ORFs; groups VAMB clusters into MGS using taxonomic consensus (--annotation_prevalence) and taxonomic scope consensus (--tax_scope_threshold)) — https://github.com/Russel88/MAGinator
- **GTDB-tk** (Reference database (release 220) used by MAGinator to taxonomically annotate bins before MGS grouping)
- **VAMB** (Produces the clusters.tsv bin assignments that MAGinator annotates and groups into MGS) — https://github.com/RasmussenLab/vamb
- **dRep** (Recommended to create species-level clusters with modifiable clustering parameters when MAGinator's own MGS grouping is insufficient; MAGinator_setup.R converts dRep output into MAGinator inputs) — https://github.com/MrOlm/drep

## Examples

```
maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -o my_output -g "/path/to/GTDB-Tk/database/release220/"
```

## Evaluation signals

- clusters.tsv and all_assemblies.fasta contain no '@' characters (post sed 's/@/_/g' preprocessing)
- Every input bin passes the --binsize size filter and receives a GTDB-tk taxonomic annotation (or is retained as unannotated and kept in the analysis)
- Each VAMB cluster is assigned a taxonomic consensus whose prevalence satisfies --annotation_prevalence and a taxonomic scope consistent with --tax_scope_threshold
- Annotated MAG clusters match the dRep species-level clusters generated via MAGinator_setup.R when dRep is used to build inputs
- Downstream phylo/stats.tab output reports signature and marker genes for the annotated clusters, confirming annotation propagated into the pipeline

## Limitations

- Unannotated VAMB clusters are kept in the analysis, so low-prevalence or novel lineages may lack a reliable taxonomic consensus
- High completeness of MAGinator can introduce noise or misclassification that influences downstream estimates (e.g. beta diversity)
- Choice of --annotation_prevalence and --tax_scope_threshold trades off sensitivity vs. specificity of the taxonomic scope consensus

## Evidence

- [readme] Run GTDB-tk to taxonomically annotate bins and call open reading frames (ORFs): "* Run GTDB-tk to taxonomically annotate bins and call open reading frames (ORFs)"
- [readme] Group your VAMB clusters into metagenomic species (MGS) based on the taxonomic annotation: "Group your VAMB clusters into metagenomic species (MGS) based on the taxonomic annotation. (Unannotated VAMB clusters are kept in the"
- [methods] Use --annotation_prevalence to change how prevalent an annotation has to be in a VAMB cluster to call taxonomic consensus: "Use --annotation_prevalence to change how prevalent an annotation has to be in a VAMB cluster to call taxonomic consensus"
- [methods] Use --tax_scope_threshold to toggle the threshold for how to find the taxonomic scope consensus: "Use --tax_scope_threshold to toggle the threshold for how to find the taxonomic scope consensus"
- [readme] Filter bins by size --binsize to control the cutoff: "* Filter bins by size
    * Use --binsize to control the cutoff"
