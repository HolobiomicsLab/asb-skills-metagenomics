---
name: vcontact2-custom-reference-database-configuration
description: Use when classified viral contigs (vOTUs) from a metagenomic/virome assembly
  need phage taxonomy assignments (phylum–genus), and a curated protein-sharing clustering
  against INPHARED reference genomes is required instead of a generic vConTACT2 reference.
when_to_use_negative:
- vConTACT2's all-versus-all protein comparison against large reference databases
  is time-consuming and the network is rebuilt every time new data are added — impractical
  for rapid, incremental updates
- When only read-level taxonomic profiling is needed (e.g. Kraken2/MiniKraken2_V1
  read classification) rather than genome-level vOTU clustering
- When a single viral genome already matches a reference directly and no network-based
  classification is necessary
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: vConTACT2
  role: Clusters vOTU proteins together with INPHARED curated bacteriophage reference
    genomes by shared protein content in custom-reference modality
- name: graphanalyzer
  role: Interprets genome_by_genome_overview.csv and c1.ntw to automatically transfer
    taxonomy from INPHARED reference genomes to each vOTU
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: Prodigal
  role: Predicts protein-coding genes on the dereplicated vOTU multifasta set; predicted
    proteins are fed to the viral taxonomy module
- name: INPHARED
  role: Supplies the monthly-updated curated phage genome database and the vConTACT2
    input files (proteins.faa, gene_to_genome.csv, data_excluding_refseq.tsv)
  repo: https://github.com/RyanCook94/inphared
- name: MetaPhage
  role: Host pipeline whose viral taxonomy module orchestrates the PRODIGAL → vConTACT2
    flow using the INPHARED custom reference
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:dd2ecbe5d1af0358b1a704af56bea0b81575c22257b2004a60b40155279e26fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lazzarigioele/graphanalyzer
  tools:
  - INPHARED
  - MetaPhage
  - Prodigal
  - graphanalyzer
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# vConTACT2 custom-reference database configuration

## Summary

Run vConTACT2 in custom-reference modality using the INPHARED Jan2022 split (vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv) to cluster predicted vOTU proteins together with curated bacteriophage reference genomes by shared protein content, then transfer taxonomy from the clustered references (e.g. via graphanalyzer).

## When to use

Use when classified viral contigs (vOTUs) from a metagenomic/virome assembly need phage taxonomy assignments (phylum–genus), and a curated protein-sharing clustering against INPHARED reference genomes is required instead of a generic vConTACT2 reference.

## When NOT to use

- vConTACT2's all-versus-all protein comparison against large reference databases is time-consuming and the network is rebuilt every time new data are added — impractical for rapid, incremental updates
- When only read-level taxonomic profiling is needed (e.g. Kraken2/MiniKraken2_V1 read classification) rather than genome-level vOTU clustering
- When a single viral genome already matches a reference directly and no network-based classification is necessary

## Inputs

- Dereplicated vOTU multifasta set (contigs after CD-HIT-EST dereplication at 0.95 identity / 0.85 coverage)
- Prodigal-predicted proteins (.faa) from the vOTU set
- INPHARED Jan2022 custom-reference files: vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv

## Outputs

- genome_by_genome_overview.csv (initial and refined viral clusters, e.g. VC_22 / VC_22_1, with confidence metrics)
- c1.ntw (source/target/weight protein-sharing network)
- Per-vOTU taxonomy_table.csv with phylum, class, order, family, and genus columns

## How to apply

First predict protein-coding genes on the dereplicated vOTU multifasta set with Prodigal (as in the MetaPhage flow PRODIGAL → vConTACT2) and feed the predicted proteins to the viral taxonomy module. Configure vConTACT2 in custom-reference modality using the INPHARED Jan2022 split — vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, and data_excluding_refseq.tsv — so vOTUs are clustered alongside curated bacteriophage genomes from GenBank (excluding RefSeq to avoid duplicated accessions). Because vConTACT2 alone does not assign taxonomy to user sequences (it outputs genome_by_genome_overview.csv and c1.ntw only), run graphanalyzer on the resulting protein-sharing network to automatically transfer taxonomy from INPHARED reference genomes to vOTU clusters. Aggregate the per-vOTU assignments into a taxonomy table CSV listing phylum, class, order, family, and genus for each dataset. Note the genome prefixes (e.g. vOTU_1) given to graphanalyzer via --prefix, since it uses that prefix to distinguish user vOTUs from reference genomes.

## Related tools

- **vConTACT2** (Clusters vOTU proteins together with INPHARED curated bacteriophage reference genomes by shared protein content in custom-reference modality)
- **graphanalyzer** (Interprets genome_by_genome_overview.csv and c1.ntw to automatically transfer taxonomy from INPHARED reference genomes to each vOTU) — https://github.com/lazzarigioele/graphanalyzer
- **Prodigal** (Predicts protein-coding genes on the dereplicated vOTU multifasta set; predicted proteins are fed to the viral taxonomy module)
- **INPHARED** (Supplies the monthly-updated curated phage genome database and the vConTACT2 input files (proteins.faa, gene_to_genome.csv, data_excluding_refseq.tsv)) — https://github.com/RyanCook94/inphared
- **MetaPhage** (Host pipeline whose viral taxonomy module orchestrates the PRODIGAL → vConTACT2 flow using the INPHARED custom reference) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 4 --view 2d
```

## Evaluation signals

- taxonomy_table.csv exists and contains the rank columns (phylum, class, order, family, genus) for each dataset
- Per-rank tallies match published values for the DNA virome dataset: 3 phyla, 3 classes, 4 orders, 9 families, 32 genera among classified vOTUs (shotgun set: 3/3/4/9 with 19 species-level taxa), within the reproducibility tolerance of network-based assignment
- genome_by_genome_overview.csv contains both initial (VC_22) and refined (VC_22_1) viral clusters with confidence metrics for all input vOTUs
- c1.ntw is populated with source/target/weight edges above vConTACT2's significance threshold
- Every vOTU with a taxonomic assignment can be traced to a shared refined VC or VC subcluster with an INPHARED reference genome

## Limitations

- vConTACT2 does not classify a vOTU directly; without graphanalyzer the classification would require the user's extensive manual inspection of the VC/subcluster contents
- The network is rebuilt every time new data are added, extending the time required even further
- Depending on the size of the reference database, the all-versus-all protein comparison can require a long time

## Evidence

- [discussion] vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [discussion] The network is rebuilt every time new data are added: "the network is rebuilt every time new data are added, extending the time required even further."
- [readme] INPHARED provides the vConTACT2 input files: [evidence span withheld — anchor status: not_found]
- [readme] Taxonomic information is not included for user sequences: [evidence span withheld — anchor status: external_doc]
- [readme] graphanalyzer automates the manual inspection: [evidence span withheld — anchor status: not_found]
