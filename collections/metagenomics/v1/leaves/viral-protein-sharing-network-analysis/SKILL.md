---
name: viral-protein-sharing-network-analysis
description: Use when dereplicated vOTU multifasta sets need genus-to-phylum taxonomic
  labels and network-based classification against curated phage references (e.g.,
  INPHARED) is acceptable, since vConTACT2 alone leaves classification to manual network
  inspection.
when_to_use_negative:
- When input is still raw reads or undereplicated contigs — the viral taxonomy module
  expects Prodigal-annotated proteins from dereplicated vOTUs
- When strict, per-genome taxonomic resolution is required, since network-based cluster
  transfer only yields probabilistic genus/family-level associations within vConTACT2
  VCs
- When runtime must be short and no new data will be added — the vConTACT2 network
  is rebuilt every time new data are added, greatly extending runtime
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: MetaPhage
  role: Pipeline whose viral taxonomy module chains Prodigal --> vConTACT2 --> graphanalyzer
    and aggregates per-vOTU assignments into a taxonomy table
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Prodigal
  role: Predicts protein-coding genes on the dereplicated vOTU multifasta to supply
    proteins for the protein-sharing network
- name: vConTACT2
  role: Clusters vOTUs with curated bacteriophage references in custom-reference modality
    using INPHARED Jan2022 split files by shared protein content
  repo: https://doi.org/10.6084/m9.figshare.20449644
- name: graphanalyzer
  role: Automatically interprets vConTACT2 outputs to transfer taxonomy from INPHARED
    reference genomes to vOTU clusters, replacing manual network inspection
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: INPHARED
  role: Provides curated bacteriophage reference genomes and the vConTACT2 input files
    (proteins.faa, gene_to_genome.csv, data_excluding_refseq.tsv), excluding RefSeq
    to avoid duplicates
  repo: https://github.com/RyanCook94/inphared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:854c0a2bea06cc609305ecb0aa05933ad73221179fd1b74c11a3d0d9ea4e713c
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
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# viral-protein-sharing-network-analysis

## Summary

Assigns taxonomy to vOTUs by building protein-sharing networks with vConTACT2 against curated INPHARED bacteriophage references and automating taxonomy transfer with graphanalyzer, as implemented in the MetaPhage viral taxonomy module.

## When to use

Use when dereplicated vOTU multifasta sets need genus-to-phylum taxonomic labels and network-based classification against curated phage references (e.g., INPHARED) is acceptable, since vConTACT2 alone leaves classification to manual network inspection.

## When NOT to use

- When input is still raw reads or undereplicated contigs — the viral taxonomy module expects Prodigal-annotated proteins from dereplicated vOTUs
- When strict, per-genome taxonomic resolution is required, since network-based cluster transfer only yields probabilistic genus/family-level associations within vConTACT2 VCs
- When runtime must be short and no new data will be added — the vConTACT2 network is rebuilt every time new data are added, greatly extending runtime

## Inputs

- Dereplicated vOTU multifasta (CD-HIT-EST at 0.95 identity, 0.85 alignment coverage)
- Prodigal-predicted protein sequences (FAA) from the vOTUs
- INPHARED custom-reference files: vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv
- vConTACT2 outputs: genome_by_genome_overview.csv and c1.ntw

## Outputs

- vConTACT2 protein-sharing network (c1.ntw) and genome_by_genome_overview.csv with VC/VC subcluster assignments
- Per-vOTU taxonomic assignments (phylum, class, order, family, genus) from graphanalyzer
- taxonomy_table.csv aggregating rank-level labels for each dataset

## How to apply

First predict protein-coding genes on the dereplicated vOTU multifasta with Prodigal and feed the predicted proteins to vConTACT2, mirroring the MetaPhage PRODIGAL --> vConTACT2 flow. Run vConTACT2 in custom-reference modality using the INPHARED Jan2022 split files (vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv) so vOTUs cluster with curated bacteriophage genomes by shared protein content. Because vConTACT2 does not assign taxonomy to user sequences directly, run graphanalyzer on the resulting c1.ntw network and genome_by_genome_overview.csv to automatically transfer taxonomy from INPHARED references (data_excluding_refseq.tsv) to vOTU clusters. Aggregate per-vOTU assignments into a taxonomy table CSV with phylum, class, order, family, and genus columns, then tally distinct taxa per rank per dataset (e.g., DNA virome: 3 phyla/3 classes/4 orders/9 families/32 genera; shotgun: 3/3/4/9 with 19 species-level taxa).

## Related tools

- **MetaPhage** (Pipeline whose viral taxonomy module chains Prodigal --> vConTACT2 --> graphanalyzer and aggregates per-vOTU assignments into a taxonomy table) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Prodigal** (Predicts protein-coding genes on the dereplicated vOTU multifasta to supply proteins for the protein-sharing network)
- **vConTACT2** (Clusters vOTUs with curated bacteriophage references in custom-reference modality using INPHARED Jan2022 split files by shared protein content) — https://doi.org/10.6084/m9.figshare.20449644
- **graphanalyzer** (Automatically interprets vConTACT2 outputs to transfer taxonomy from INPHARED reference genomes to vOTU clusters, replacing manual network inspection) — https://github.com/lazzarigioele/graphanalyzer
- **INPHARED** (Provides curated bacteriophage reference genomes and the vConTACT2 input files (proteins.faa, gene_to_genome.csv, data_excluding_refseq.tsv), excluding RefSeq to avoid duplicates) — https://github.com/RyanCook94/inphared

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 8 --view 2d
```

## Evaluation signals

- taxonomy_table.csv exists and contains rank columns for phylum, class, order, family, and genus
- Per-rank tallies match the reported values within reproducibility tolerance — exactly 9 families and 32 genera for the DNA virome set (3 phyla/3 classes/4 orders)
- Shotgun metagenome set reproduces 3 phyla/3 classes/4 orders/9 families with 19 species-level taxa
- Every vOTU prefixed with the --prefix identifier receives a direct taxonomic association from graphanalyzer rather than requiring manual inspection
- genome_by_genome_overview.csv contains initial and refined viral cluster (VC/VC subcluster) fields populated for all vOTUs

## Limitations

- vConTACT2 does not classify a vOTU directly and without graphanalyzer leaves classification to the user's extensive manual network inspection
- The vConTACT2 network is rebuilt every time new data are added, greatly extending runtime
- All-versus-all protein comparison against large reference databases like INPHARED can require a long time
- vConTACT2 taxonomic transfer is probabilistic: same VC subcluster implies very high probability of same genus, but same VC without subcluster is only genus-subfamily level relatedness

## Evidence

- [discussion] vConTACT2 does not directly classify vOTUs, requiring manual inspection absent graphanalyzer: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [readme] graphanalyzer automates manual inspection of vConTACT2 outputs to give each vOTU a direct taxonomic association: [evidence span withheld — anchor status: not_found]
- [readme] graphanalyzer requires the three vConTACT2/INPHARED output files: [evidence span withheld — anchor status: external_doc]
- [readme] INPHARED excludes RefSeq duplicates and provides data_excluding_refseq.tsv for taxonomic retrieval: [evidence span withheld — anchor status: external_doc]
- [discussion] graphanalyzer is released under GPL-3.0 with the MetaPhage group: "graphanalyzer is available under the same license at https://github.com/lazzarigioele/graphanalyzer"
