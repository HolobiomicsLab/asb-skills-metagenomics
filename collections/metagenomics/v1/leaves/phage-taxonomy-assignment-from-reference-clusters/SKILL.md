---
name: phage-taxonomy-assignment-from-reference-clusters
description: Use when a dereplicated vOTU multifasta needs phylum-to-genus (and species-level)
  taxonomic labels by shared-protein-content similarity to curated INPHARED/GenBank
  phage reference genomes.
when_to_use_negative:
- When sequences have no vOTU-style naming prefix and cannot be distinguished from
  reference genomes in the network files, since graphanalyzer relies on --prefix matching
- When reference-based cluster sharing is expected to fail, e.g. deeply novel phages
  with no INPHARED relatives — assignments will remain unclassified at genus level
- For direct read-level taxonomy (use Kraken2/MiniKraken2_V1 read classification instead)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaPhage
  role: pipeline whose viral taxonomy module orchestrates the PRODIGAL -> vConTACT2
    flow
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: vConTACT2
  role: clusters vOTUs with INPHARED bacteriophage references by shared protein content
    in custom-reference modality
- name: graphanalyzer
  role: automatically transfers taxonomy from INPHARED references to vOTU clusters
    by interpreting vConTACT2 outputs
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: Prodigal
  role: predicts protein-coding genes on the dereplicated vOTU multifasta feeding
    the taxonomy module
- name: INPHARED
  role: curated GenBank phage reference database providing vConTACT2 input files and
    taxonomy metadata
  repo: https://github.com/RyanCook94/inphared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:91d6fab7ec1c28795c519933179dcf6c90bbed162767548075445878fc261f46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - INPHARED
  - MetaPhage
  - Prodigal
  - graphanalyzer
  - vConTACT2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# phage-taxonomy-assignment-from-reference-clusters

## Summary

Assigns taxonomy to dereplicated vOTUs by clustering them with curated bacteriophage references in a protein-sharing network (vConTACT2 on INPHARED) and automatically transferring labels from the network with graphanalyzer, because vConTACT2 alone leaves classification to manual inspection.

## When to use

Use when a dereplicated vOTU multifasta needs phylum-to-genus (and species-level) taxonomic labels by shared-protein-content similarity to curated INPHARED/GenBank phage reference genomes.

## When NOT to use

- When sequences have no vOTU-style naming prefix and cannot be distinguished from reference genomes in the network files, since graphanalyzer relies on --prefix matching
- When reference-based cluster sharing is expected to fail, e.g. deeply novel phages with no INPHARED relatives — assignments will remain unclassified at genus level
- For direct read-level taxonomy (use Kraken2/MiniKraken2_V1 read classification instead)

## Inputs

- dereplicated vOTU multifasta (CD-HIT-EST at 0.95 identity / 0.85 coverage)
- predicted protein sequences from Prodigal
- INPHARED vConTACT2_proteins.faa
- INPHARED vConTACT2_gene_to_genome.csv
- INPHARED data_excluding_refseq.tsv
- vConTACT2 outputs: genome_by_genome_overview.csv and c1.ntw

## Outputs

- vConTACT2 protein-sharing network (c1.ntw) and genome_by_genome_overview.csv with VC/VC subcluster assignments
- graphanalyzer per-vOTU taxonomic associations plus interactive 2D/3D subgraphs
- taxonomy_table.csv with phylum, class, order, family, genus labels per vOTU per dataset
- per-rank taxon tallies for DNA virome and shotgun datasets

## How to apply

Predict protein-coding genes on the dereplicated vOTU multifasta with Prodigal and feed predicted proteins to the viral taxonomy module (MetaPhage flow: PRODIGAL --> vConTACT2). Run vConTACT2 in custom-reference modality with the INPHARED Jan2022 'excluding_refseq' split — vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, and data_excluding_refseq.tsv — so vOTUs cluster with curated bacteriophage references by shared protein content. Because vConTACT2 does not classify a vOTU directly, run graphanalyzer on the genome_by_genome_overview.csv and c1.ntw outputs, supplying data_excluding_refseq.tsv as the metadata table, to automatically transfer taxonomy from reference genomes sharing VC subclusters to the vOTUs. Aggregate per-vOTU assignments into a taxonomy table CSV with phylum, class, order, family, and genus columns per dataset, then tally distinct taxa per rank and compare with published counts (DNA virome: 3 phyla, 3 classes, 4 orders, 9 families, 32 genera; shotgun: 3/3/4/9 with 19 species-level taxa). vOTUs must be named with the prefix expected by --prefix (e.g. vOTU_1) so graphanalyzer can distinguish them from reference genomes.

## Related tools

- **MetaPhage** (pipeline whose viral taxonomy module orchestrates the PRODIGAL -> vConTACT2 flow) — https://github.com/MattiaPandolfoVR/MetaPhage
- **vConTACT2** (clusters vOTUs with INPHARED bacteriophage references by shared protein content in custom-reference modality)
- **graphanalyzer** (automatically transfers taxonomy from INPHARED references to vOTU clusters by interpreting vConTACT2 outputs) — https://github.com/lazzarigioele/graphanalyzer
- **Prodigal** (predicts protein-coding genes on the dereplicated vOTU multifasta feeding the taxonomy module)
- **INPHARED** (curated GenBank phage reference database providing vConTACT2 input files and taxonomy metadata) — https://github.com/RyanCook94/inphared

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 4 --view 2d
```

## Evaluation signals

- taxonomy_table.csv exists and contains rank columns for phylum, class, order, family, and genus
- Per-rank tallies match published counts: DNA virome exactly 3 phyla, 3 classes, 4 orders, 9 families, 32 genera
- Shotgun metagenome tallies match 3/3/4/9 with 19 species-level taxa
- vConTACT2 produced both genome_by_genome_overview.csv (with VC/VC_subcluster columns) and c1.ntw before graphanalyzer runs
- Every vOTU in the input multifasta receives an entry (taxonomy or unclassified) in the graphanalyzer output

## Limitations

- vConTACT2 does not directly classify vOTUs; without graphanalyzer the classification is left to the user's extensive manual inspection of the network
- The all-versus-all protein comparison against large reference databases is time-consuming, and the network is rebuilt every time new data are added, greatly extending runtime
- INPHARED inputs are the excluding_refseq version to avoid duplicating RefSeq genomes, so taxonomy must be retrieved via data_excluding_refseq.tsv rather than RefSeq metadata

## Evidence

- [other] Taxonomic classification of the DNA virome data set with vConTACT2 and graphanalyzer identified 3 phyla, 3 classes, 4 orders, 9 families, and 32 genera among the classified vOTUs.: [evidence span withheld — anchor status: not_found]
- [discussion] vConTACT2 does not directly classify vOTUs, requiring manual inspection absent graphanalyzer: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [readme] graphanalyzer automates the manual inspection of vConTACT2 outputs: [evidence span withheld — anchor status: not_found]
- [readme] graphanalyzer input requirements: [evidence span withheld — anchor status: not_found]
- [readme] INPHARED excluding_refseq rationale: [evidence span withheld — anchor status: external_doc]
