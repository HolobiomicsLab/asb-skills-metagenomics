---
name: phage-taxonomy-conservatism-rules
description: Use when vConTACT2 has produced genome_by_genome_overview.csv and c1.ntw
  for vOTUs (e.g., from MetaPhage) and a direct taxonomic label per vOTU is needed
  instead of manual network inspection.
when_to_use_negative:
- vConTACT2 outputs were not generated with the INPHARED excluding-refseq database,
  so reference taxonomy metadata in the expected format is unavailable
- A genus-level ICTV classification requiring exhaustive evidence is needed — this
  rule-based inheritance only propagates taxonomy of the closest reference neighbor
- Input is raw reads or unassembled contigs rather than already-clustered vOTUs with
  a vConTACT2 network
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: graphanalyzer
  role: Implements the neighbor-sorting-by-edge-weight taxonomy inheritance and conservatism
    truncation rules on vConTACT2 outputs
  repo: https://github.com/lazzarigioele/graphanalyzer
- name: vConTACT2
  role: Produces the network (c1.ntw) and cluster overview (genome_by_genome_overview.csv)
    that graphanalyzer interprets
- name: INPHARED
  role: Supplies curated phage reference genomes and the protein/gene-to-genome files
    used as vConTACT2 input
  repo: https://github.com/RyanCook94/inphared
- name: MetaPhage
  role: Upstream Nextflow pipeline that generates the vOTUs and runs the vConTACT2
    classification step
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:e6f3cff116de473bad83df71e1f828c43fef28e05c1216092a321e3264894cb0
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phage-taxonomy-conservatism-rules

## Summary

Automates vOTU taxonomic assignment from vConTACT2 network outputs by inheriting taxonomy from same-cluster reference genomes ordered by edge weight, applying conservatism rules that truncate assignments at subfamily/family level depending on the neighbor's status and origin.

## When to use

Use when vConTACT2 has produced genome_by_genome_overview.csv and c1.ntw for vOTUs (e.g., from MetaPhage) and a direct taxonomic label per vOTU is needed instead of manual network inspection.

## When NOT to use

- vConTACT2 outputs were not generated with the INPHARED excluding-refseq database, so reference taxonomy metadata in the expected format is unavailable
- A genus-level ICTV classification requiring exhaustive evidence is needed — this rule-based inheritance only propagates taxonomy of the closest reference neighbor
- Input is raw reads or unassembled contigs rather than already-clustered vOTUs with a vConTACT2 network

## Inputs

- c1.ntw — vConTACT2 graph file with source/target/edge-weight pairs above the significance threshold
- genome_by_genome_overview.csv — vConTACT2 overview with viral cluster (e.g. VC_22) and refined cluster (e.g. VC_22_1) assignments
- Reference taxonomy metadata for INPHARED GenBank genomes (e.g. data_excluding_refseq.tsv or vConTACT2_gene_to_genome.csv)
- vOTU-named viral scaffolds distinguishable by a prefix (e.g. vOTU_1, vOTU_2)

## Outputs

- Taxonomy table CSV with columns for the vOTU, assigned taxonomy, Status, and Weight
- Per-vOTU interactive subnetwork views (2D or 3D plotly graphs) of each vOTU's graph neighborhood

## How to apply

Run graphanalyzer.py on the three vConTACT2/INPHARED outputs: c1.ntw (source/target/edge-weight graph), genome_by_genome_overview.csv (VC/VC subcluster assignments), and the reference taxonomy metadata file. For each vOTU (identified by a --prefix such as 'vOTU'), retrieve its same-cluster neighbors and order them by vConTACT2 edge weight, then inherit taxonomy from the first neighbor that is a reference genome. Apply the conservatism rules: truncate the assignment at subfamily level for Clustered/Singleton or Overlap statuses, truncate at family level when the taxonomy comes from outside the cluster (no reference genome in the same VC), and label the vOTU unclassified otherwise. Rationale: vConTACT2 itself provides no taxonomic association for user sequences — same-VC-subcluster placement implies genus-level relatedness, same-VC-only implies roughly genus-subfamily level, and no shared reference implies at most subfamily/family/order relatedness.

## Related tools

- **graphanalyzer** (Implements the neighbor-sorting-by-edge-weight taxonomy inheritance and conservatism truncation rules on vConTACT2 outputs) — https://github.com/lazzarigioele/graphanalyzer
- **vConTACT2** (Produces the network (c1.ntw) and cluster overview (genome_by_genome_overview.csv) that graphanalyzer interprets)
- **INPHARED** (Supplies curated phage reference genomes and the protein/gene-to-genome files used as vConTACT2 input) — https://github.com/RyanCook94/inphared
- **MetaPhage** (Upstream Nextflow pipeline that generates the vOTUs and runs the vConTACT2 classification step) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 4 --view 2d
```

## Evaluation signals

- Taxonomy table CSV exists and contains both Status and Weight fields
- Every vOTU row carries a status consistent with the conservatism rules (subfamily-level truncation for Clustered/Singleton or Overlap, family-level for out-of-cluster inheritance, 'unclassified' otherwise)
- Each assignment traces to the highest-weight same-cluster reference genome in the sorted neighbor list
- Interactive subnetwork views are generated per vOTU and contain the neighbors supporting the assignment
- Reference genomes in the network resolve via the metadata file; vOTU names match the chosen --prefix

## Limitations

- vConTACT2 does not classify vOTUs directly; without graphanalyzer the classification is left to extensive manual inspection
- The vConTACT2 network is rebuilt every time new data are added, greatly extending runtime
- Taxonomy is inherited, not independently validated — assignments are only as reliable as the closest same-cluster reference and the stated probability heuristics (genus within subcluster, genus-subfamily within VC)
- graphanalyzer was tested only under Linux and depends on networkx, pandas, plotly, openpyxl, and pygraphviz

## Evidence

- [readme] vConTACT2 doesn't provide a direct taxonomic association: [evidence span withheld — anchor status: external_doc]
- [readme] graphanalyzer.py automates manual inspection: [evidence span withheld — anchor status: external_doc]
- [readme] VC subcluster implies genus probability: [evidence span withheld — anchor status: external_doc]
- [discussion] graphanalyzer inherits via vConTACT2 edge weight: [evidence span withheld — anchor status: not_found]
- [discussion] graphanalyzer repo and license: "graphanalyzer is available under the same license at https://github.com/lazzarigioele/graphanalyzer"
