---
name: edge-weight-ranked-neighbor-traversal
description: Use when vConTACT2 outputs (genome_by_genome_overview.csv, c1.ntw) must
  be converted into a direct per-vOTU taxonomy table instead of manual network inspection.
when_to_use_negative:
- Taxonomy is already directly provided by the classifier output (vConTACT2 does not
  classify vOTUs, but tools like geNomad or sanity-checked reference mapping may make
  this traversal unnecessary).
- The network was not built against a reference database with gene-to-genome taxonomy
  files (e.g. no INPHARED proteins.faa / gene_to_genome.csv), since there is no reference
  neighbor from which to inherit taxonomy.
- The goal is de novo taxonomic assignment of viruses absent from the network reference
  space entirely (Singleton vOTUs with no cluster support).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0780
tools:
- name: graphanalyzer
  role: Implements the edge-weight-ranked neighbor traversal, conservatism rules,
    CSV output, and interactive subgraphs
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: vConTACT2
  role: Produces the network (c1.ntw) and cluster (genome_by_genome_overview.csv)
    inputs that the traversal interprets
  repo: https://bitbucket.org/MAVERICLab/vcontact2/
- name: MetaPhage
  role: Pipeline that runs vConTACT2 and graphanalyzer to classify vOTUs built from
    Prodigal-predicted proteins against the INPHARED Jan2022 reference
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: INPHARED
  role: Supplies curated phage reference genomes plus vConTACT2 input files (proteins.faa,
    gene_to_genome.csv, data_excluding_refseq.tsv)
  repo: https://github.com/RyanCook94/inphared
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:0c87ce3b99d6132b41137964e4f2ca3aa0b01f266a26615f7bfc47e63a8b3269
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# edge-weight-ranked-neighbor-traversal

## Summary

Automates taxonomic classification of vOTUs from a vConTACT2 network by traversing each vOTU's same-cluster neighbors in descending edge-weight order and inheriting taxonomy from the first reference genome encountered, with conservatism rules encoded in graphanalyzer.

## When to use

Use when vConTACT2 outputs (genome_by_genome_overview.csv, c1.ntw) must be converted into a direct per-vOTU taxonomy table instead of manual network inspection.

## When NOT to use

- Taxonomy is already directly provided by the classifier output (vConTACT2 does not classify vOTUs, but tools like geNomad or sanity-checked reference mapping may make this traversal unnecessary).
- The network was not built against a reference database with gene-to-genome taxonomy files (e.g. no INPHARED proteins.faa / gene_to_genome.csv), since there is no reference neighbor from which to inherit taxonomy.
- The goal is de novo taxonomic assignment of viruses absent from the network reference space entirely (Singleton vOTUs with no cluster support).

## Inputs

- vConTACT2 genome_by_genome_overview.csv (viral cluster, refined VC, confidence metrics)
- vConTACT2 c1.ntw graph file (source/target/edge weight per genome pair)
- INPHARED data_excluding_refseq.tsv or vConTACT2_gene_to_genome.csv (GenBank gene-to-genome taxonomy)

## Outputs

- CSV taxonomy table with per-vOTU assigned taxonomy, Status, and Weight columns
- Interactive 2D or 3D per-vOTU subnetwork views (Plotly) of the vConTACT2 graph neighborhood

## How to apply

Load the vConTACT2 genome_by_genome_overview.csv and c1.ntw (source/target/edge-weight pairs) plus the INPHARED reference metadata (vConTACT2_gene_to_genome.csv / data_excluding_refseq.tsv). For each vOTU node (identified by a --prefix such as 'vOTU'), retrieve its same-cluster neighbors and sort them by vConTACT2 edge weight, then inherit the taxonomy of the first neighbor that is a reference genome; if no reference is in the cluster, fall back to non-cluster neighbors. Apply the conservatism rules: truncate at subfamily level for Clustered/Singleton or Overlap statuses, truncate at family level when taxonomy comes from outside the cluster, and label the vOTU unclassified otherwise. Write a CSV with columns for vOTU, assigned taxonomy, Status, and Weight, and emit an interactive 2D or 3D subnetwork view per vOTU supporting each assignment.

## Related tools

- **graphanalyzer** (Implements the edge-weight-ranked neighbor traversal, conservatism rules, CSV output, and interactive subgraphs) — https://github.com/lazzarigioele/graphanalyzer
- **vConTACT2** (Produces the network (c1.ntw) and cluster (genome_by_genome_overview.csv) inputs that the traversal interprets) — https://bitbucket.org/MAVERICLab/vcontact2/
- **MetaPhage** (Pipeline that runs vConTACT2 and graphanalyzer to classify vOTUs built from Prodigal-predicted proteins against the INPHARED Jan2022 reference) — https://github.com/MattiaPandolfoVR/MetaPhage
- **INPHARED** (Supplies curated phage reference genomes plus vConTACT2 input files (proteins.faa, gene_to_genome.csv, data_excluding_refseq.tsv)) — https://github.com/RyanCook94/inphared

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --view 2d --threads 4
```

## Evaluation signals

- Taxonomy table CSV exists and contains both the Status and Weight fields.
- Every vOTU row carries a status consistent with the conservatism rules (subfamily-level truncation for Clustered/Singleton or Overlap; family-level truncation for out-of-cluster inheritance; unclassified otherwise).
- Each assigned vOTU's inherited taxonomy matches the taxonomy of the highest-weight same-cluster reference neighbor in c1.ntw / data_excluding_refseq.tsv.
- Interactive subnetwork views are generated per vOTU (2D or 3D per --view) and show the reference neighbor supporting the assignment.
- Concordance check: vOTUs sharing a refined VC subcluster with the same reference genome receive the same genus-level taxonomy.

## Limitations

- vConTACT2 does not classify a vOTU directly, so without graphanalyzer the classification is left to extensive manual inspection of genome_by_genome_overview.csv and c1.ntw.
- If a vOTU's cluster contains no reference genome, taxonomy can only be inferred from non-cluster neighbors and is truncated at family level; truly isolated vOTUs remain unclassified.
- Taxonomic inheritance quality depends on the INPHARED reference database version (e.g. Jan2022) and the 'excluding_refseq' duplication-avoidance scheme; the network is rebuilt every time new data are added, extending runtime.

## Evidence

- [readme] The two main outputs generated by vConTACT2 are the `genome_by_genome_overview.csv` and the `c1.ntw`.: [evidence span withheld — anchor status: not_found]
- [readme] graphanalyzer was written to automate manual inspection of vConTACT2 outputs providing a direct taxonomic association for every vOTU.: [evidence span withheld — anchor status: external_doc]
- [discussion] vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection: "vConTACT2 does not classify a vOTU directly but instead would leave the classification up to the user's extensive manual inspection"
- [readme] the source / target / edge weight information for all genome pairs higher than the significance threshold: [evidence span withheld — anchor status: external_doc]
- [other] The task card describes the algorithm: neighbors sorted by edge weight, taxonomy inherited from first reference genome with conservatism rules.: [evidence span withheld — anchor status: not_found]
