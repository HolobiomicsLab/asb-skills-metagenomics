---
name: vcontact2-network-taxonomy-inference
description: Use when vOTU contigs have been clustered against INPHARED references
  with vConTACT2 (producing genome_by_genome_overview.csv and c1.ntw) and you need
  automated per-vOTU taxonomy plus supporting network views.
when_to_use_negative:
- You have not run vConTACT2 against INPHARED references — graphanalyzer specifically
  interprets vConTACT2 outputs (c1.ntw, genome_by_genome_overview.csv) built with
  INPHARED input files
- Your vOTU names do not match the --prefix convention (e.g. vOTU_1, vOTU_2), since
  the prefix distinguishes the vOTUs to classify from the other genomes in the network
- You need direct taxonomic calls from vConTACT2 itself — the tool provides no taxonomic
  association for user sequences, so this skill is about network-based inheritance,
  not direct vConTACT2 classification
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: graphanalyzer
  role: Automates inspection of vConTACT2 outputs (c1.ntw, genome_by_genome_overview.csv)
    to assign a taxonomy to every vOTU and produce interactive subgraphs
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: vConTACT2
  role: Generates the genome-by-genome network and viral cluster assignments that
    graphanalyzer interprets; invoked via the Docker image packaged with graphanalyzer
    dependencies (v0.11.3)
- name: MetaPhage
  role: Pipeline that runs the vConTACT2 clustering step and calls graphanalyzer for
    vOTU classification within the reads-to-report workflow
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: INPHARED
  role: Supplies the curated phage reference database and the vConTACT2 input files
    (proteins.faa, gene_to_genome.csv) used for taxonomic inheritance
  repo: https://github.com/RyanCook94/inphared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:b7219e0538a3618a6b6280430f76b5c7a5b7e5c46c83f26892b3f82582948d05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lazzarigioele/graphanalyzer
  tools:
  - INPHARED
  - MetaPhage
  - Nextflow
  - graphanalyzer
  - vConTACT2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# vcontact2-network-taxonomy-inference

## Summary

Automates taxonomic classification of vOTUs from the vConTACT2 network using the graphanalyzer tool, which orders same-cluster neighbors by edge weight and inherits taxonomy from the first reference genome, replacing the manual inspection that vConTACT2 otherwise requires.

## When to use

Use when vOTU contigs have been clustered against INPHARED references with vConTACT2 (producing genome_by_genome_overview.csv and c1.ntw) and you need automated per-vOTU taxonomy plus supporting network views.

## When NOT to use

- You have not run vConTACT2 against INPHARED references — graphanalyzer specifically interprets vConTACT2 outputs (c1.ntw, genome_by_genome_overview.csv) built with INPHARED input files
- Your vOTU names do not match the --prefix convention (e.g. vOTU_1, vOTU_2), since the prefix distinguishes the vOTUs to classify from the other genomes in the network
- You need direct taxonomic calls from vConTACT2 itself — the tool provides no taxonomic association for user sequences, so this skill is about network-based inheritance, not direct vConTACT2 classification

## Inputs

- vConTACT2 genome_by_genome_overview.csv (viral cluster, refined VC, confidence metrics)
- vConTACT2 c1.ntw graph file (source/target/edge weight for significant genome pairs)
- INPHARED data_excluding_refseq.tsv or vConTACT2_gene_to_genome.csv (gene-to-genome taxonomy mapping)
- Protein calls (proteins.faa) from vOTU Prodigal predictions against the INPHARED Jan2022 reference

## Outputs

- Per-vOTU taxonomy table CSV with columns for the vOTU, assigned taxonomy, Status (Clustered/Singleton, Overlap, etc.), and Weight
- Interactive per-vOTU subnetwork views (2d or 3d, plotly) of the vConTACT2 graph neighborhood supporting each assignment

## How to apply

Load the vConTACT2 outputs generated from Prodigal-predicted proteins against the INPHARED Jan2022 reference: the genome_by_genome_overview.csv, the c1.ntw graph, and the taxonomy mapping (vConTACT2_gene_to_genome.csv / INPHARED data_excluding_refseq.tsv). For each vOTU node, retrieve its same-cluster neighbors and order them by vConTACT2 edge weight; inherit the taxonomy of the first neighbor that is a reference genome, falling back to non-cluster neighbors when no reference genome is present in the cluster. Apply the conservatism rules: truncate the assignment at subfamily level for Clustered/Singleton or Overlap statuses, truncate at family level when the taxonomy comes from outside the cluster, and label the vOTU unclassified otherwise. Write the taxonomy table as a CSV with columns for the vOTU, assigned taxonomy, Status, and Weight, and generate a per-vOTU interactive (2d or 3d) subnetwork view of the graph neighborhood supporting each assignment. The rationale is that vConTACT2 'does not classify a vOTU directly' and would otherwise leave classification to extensive manual inspection of the overview CSV and network file.

## Related tools

- **graphanalyzer** (Automates inspection of vConTACT2 outputs (c1.ntw, genome_by_genome_overview.csv) to assign a taxonomy to every vOTU and produce interactive subgraphs) — https://github.com/lazzarigioele/graphanalyzer
- **vConTACT2** (Generates the genome-by-genome network and viral cluster assignments that graphanalyzer interprets; invoked via the Docker image packaged with graphanalyzer dependencies (v0.11.3))
- **MetaPhage** (Pipeline that runs the vConTACT2 clustering step and calls graphanalyzer for vOTU classification within the reads-to-report workflow) — https://github.com/MattiaPandolfoVR/MetaPhage
- **INPHARED** (Supplies the curated phage reference database and the vConTACT2 input files (proteins.faa, gene_to_genome.csv) used for taxonomic inheritance) — https://github.com/RyanCook94/inphared

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 4 --view 2d
```

## Evaluation signals

- Taxonomy table CSV exists and contains the Status and Weight fields
- Every vOTU row carries a status consistent with the conservatism rules (subfamily-level truncation for Clustered/Singleton or Overlap; family-level truncation for taxonomy from outside the cluster; unclassified otherwise)
- Each assigned taxonomy traces to the highest-edge-weight reference genome in the vOTU's same-cluster neighbor list (or an out-of-cluster fallback when no reference is in the cluster)
- Interactive subnetwork files are produced for each vOTU assignment and the neighborhood shown contains the reference genome from which the taxonomy was inherited
- vOTU names in the output all start with the chosen --prefix, distinguishing user sequences from reference genomes

## Limitations

- vConTACT2 does not directly classify vOTUs; without graphanalyzer the classification is left to the user's extensive manual inspection of genome_by_genome_overview.csv and c1.ntw
- Assignments are inherited, not de novo: when no reference genome shares the cluster, the taxonomy falls back outside the cluster and is truncated to family level, and may be 'unclassified' outright
- The vConTACT2 network is rebuilt every time new data are added, greatly extending runtime
- graphanalyzer was tested only under Linux and depends on networkx, pandas, plotly, openpyxl, and pygraphviz

## Evidence

- [discussion] vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [readme] graphanalyzer.py was written to automate the manual inspection of the vConTACT2 outputs: [evidence span withheld — anchor status: external_doc]
- [readme] vConTACT2 outputs do not provide a direct taxonomic association for each user viral sequence (vOTU): [evidence span withheld — anchor status: external_doc]
- [readme] graphanalyzer interprets the outputs generated by vConTACT2 and requires three input files: [evidence span withheld — anchor status: external_doc]
- [readme] The prefix distinguishes the vOTUs in input from the other genomes: [evidence span withheld — anchor status: external_doc]
