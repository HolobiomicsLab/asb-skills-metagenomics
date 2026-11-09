---
name: taxonomic-rank-tally-validation
description: Use when reproducing or validating taxonomic classification counts of
  vOTUs from virome or shotgun metagenome assemblies, where expected per-rank tallies
  (phylum through genus/species) must be matched against published values.
when_to_use_negative:
- The goal is de novo viral taxonomy discovery rather than reproducing the MetaPhage
  benchmark tallies against INPHARED-based reference assignment
- Input contigs have not been dereplicated into vOTUs (95% ANI / 85% aligned fraction
  clustering), so counts would not be comparable to the published per-rank tallies
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaPhage
  role: Pipeline whose taxonomy module (Prodigal -> vConTACT2 -> graphanalyzer) produces
    the vOTU taxonomic classifications being tallied
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Prodigal
  role: Predicts protein-coding genes on the dereplicated vOTU multifasta; predicted
    proteins are fed to the viral taxonomy module
- name: vConTACT2
  role: Clusters vOTUs with curated bacteriophage reference genomes by shared protein
    content in custom-reference modality using the INPHARED Jan2022 split
- name: graphanalyzer
  role: Automatically transfers taxonomy from INPHARED reference genomes to vOTU clusters
    by interpreting genome_by_genome_overview.csv and c1.ntw, replacing manual network
    inspection
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: INPHARED
  role: Curated bacteriophage genome database providing the custom vConTACT2 reference
    files and data_excluding_refseq.tsv taxonomy metadata
  repo: https://github.com/RyanCook94/inphared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:dafe35c02bc9f75dbdebf91bae6423a098afa89acdf20b121825d3e0ee6a44bc
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# taxonomic-rank-tally-validation

## Summary

Reproduce the MetaPhage taxonomy-assignment benchmark: run Prodigal -> vConTACT2 (custom INPHARED Jan2022 reference) -> graphanalyzer on dereplicated vOTUs, aggregate per-rank assignments into a taxonomy_table.csv, and validate per-rank tallies against published counts (DNA virome: 3 phyla/3 classes/4 orders/9 families/32 genera).

## When to use

Use when reproducing or validating taxonomic classification counts of vOTUs from virome or shotgun metagenome assemblies, where expected per-rank tallies (phylum through genus/species) must be matched against published values.

## When NOT to use

- The goal is de novo viral taxonomy discovery rather than reproducing the MetaPhage benchmark tallies against INPHARED-based reference assignment
- Input contigs have not been dereplicated into vOTUs (95% ANI / 85% aligned fraction clustering), so counts would not be comparable to the published per-rank tallies

## Inputs

- Dereplicated vOTU multifasta sets (CD-HIT-EST dereplication at 0.95 identity, 0.85 coverage) for the DNA virome and shotgun metagenome datasets
- INPHARED Jan2022 reference files: vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv
- vConTACT2 outputs: genome_by_genome_overview.csv and c1.ntw

## Outputs

- taxonomy_table.csv with per-vOTU phylum, class, order, family, and genus labels for each dataset
- Per-rank tallies of distinct taxa (e.g., 3/3/4/9 phyla-classes-orders-families with 32 genera for the DNA virome set) for comparison against published counts

## How to apply

Predict protein-coding genes on the dereplicated vOTU multifasta with Prodigal and feed the predicted proteins to vConTACT2 in custom-reference modality using the INPHARED Jan2022 split files (vConTACT2_proteins.faa, vConTACT2_gene_to_genome.csv, data_excluding_refseq.tsv) so vOTUs cluster with curated bacteriophage genomes by shared protein content. Because vConTACT2 alone does not classify vOTUs directly, run graphanalyzer on the resulting genome_by_genome_overview.csv and c1.ntw network (with --metas data_excluding_refseq.tsv) to automatically transfer taxonomy from INPHARED reference genomes to vOTU clusters. Aggregate per-vOTU assignments into a taxonomy table CSV listing phylum, class, order, family, and genus labels, then tally distinct taxa per rank per dataset. Validate by checking the CSV exists, contains the rank columns, and per-rank tallies match reported values (DNA virome: exactly 9 families and 32 genera; shotgun: 3/3/4/9 with 19 species-level taxa).

## Related tools

- **MetaPhage** (Pipeline whose taxonomy module (Prodigal -> vConTACT2 -> graphanalyzer) produces the vOTU taxonomic classifications being tallied) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Prodigal** (Predicts protein-coding genes on the dereplicated vOTU multifasta; predicted proteins are fed to the viral taxonomy module)
- **vConTACT2** (Clusters vOTUs with curated bacteriophage reference genomes by shared protein content in custom-reference modality using the INPHARED Jan2022 split)
- **graphanalyzer** (Automatically transfers taxonomy from INPHARED reference genomes to vOTU clusters by interpreting genome_by_genome_overview.csv and c1.ntw, replacing manual network inspection) — https://github.com/lazzarigioele/graphanalyzer
- **INPHARED** (Curated bacteriophage genome database providing the custom vConTACT2 reference files and data_excluding_refseq.tsv taxonomy metadata) — https://github.com/RyanCook94/inphared

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 8 --view 2d
```

## Evaluation signals

- taxonomy_table.csv exists and contains rank columns for phylum, class, order, family, and genus
- DNA virome per-rank tallies match published counts: exactly 3 phyla, 3 classes, 4 orders, 9 families, and 32 genera among classified vOTUs
- Shotgun dataset tallies match reported 3/3/4/9 phyla-classes-orders-families with 19 species-level taxa
- graphanalyzer produced a direct taxonomic association for every vOTU (prefix, e.g., vOTU_*, present in genome_by_genome_overview.csv and c1.ntw)
- Tallies fall within the reproducibility tolerance of the network-based assignment when rerun

## Limitations

- vConTACT2 does not classify vOTUs directly and, absent graphanalyzer, leaves classification to the user's extensive manual network inspection
- The vConTACT2 network is rebuilt every time new data are added, greatly extending runtime; the all-versus-all protein comparison against large reference databases is time-consuming
- Classification depends on reference genomes sharing the same VC/VC subcluster; vOTUs without nearby references may only be assignable at higher taxonomic ranks

## Evidence

- [other] Taxonomic classification of the DNA virome data set with vConTACT2 and graphanalyzer identified 3 phyla, 3 classes, 4 orders, 9 families, and 32 genera among the classified vOTUs.: [evidence span withheld — anchor status: not_found]
- [discussion] vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection.: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [readme] graphanalyzer.py was written to try to automate the manual inspection of the vConTACT2 outputs, providing the user a direct taxonomic association for every vOTU.: [evidence span withheld — anchor status: external_doc]
- [readme] graphanalyzer requires genome_by_genome_overview.csv, c1.ntw, and data_excluding_refseq.tsv as inputs with a --prefix distinguishing vOTUs from reference genomes.: [evidence span withheld — anchor status: external_doc]
- [readme] The INPHARED reference files are based on the excluding_refseq genome database, so taxonomy is retrieved via data_excluding_refseq.tsv.: [evidence span withheld — anchor status: external_doc]
