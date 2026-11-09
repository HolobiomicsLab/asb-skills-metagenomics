---
name: interactive-subnetwork-visualization
description: Use when you need to visually justify or audit taxonomic assignments
  of vOTUs within a vConTACT2 network, e.g. inspecting which INPHARED reference genomes
  neighbor each vOTU in its cluster.
when_to_use_negative:
- When no vConTACT2 network has been built (e.g. classification was done by marker-based
  tools like Demovir or VirHostMatcher) — there is no graph neighborhood to visualize.
- When you only need batch taxonomic assignments without visual audit — the taxonomy
  table alone suffices and subgraph rendering adds runtime.
- When input genomes were not named with the --prefix distinguishing vOTUs from reference
  genomes, the script cannot separate user scaffolds from references.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0797
tools:
- name: graphanalyzer
  role: Generates the per-vOTU interactive subnetwork views (2D/3D, plotly-based)
    of the vConTACT2 graph supporting each taxonomic assignment
  repo: github.com/lazzarigioele/graphanalyzer
- name: vConTACT2
  role: Produces the network outputs (c1.ntw, genome_by_genome_overview.csv) whose
    neighborhoods are visualized
- name: MetaPhage
  role: Upstream Nextflow pipeline that runs vConTACT2 and graphanalyzer on vOTUs
    and integrates the interactive subgraphs into the report
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:912fa0a3435a9b8809a240c6c80a273a24f86811634c9502cfe1fd3f919028f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaPhage
  - Nextflow
  - graphanalyzer
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Interactive subnetwork visualization

## Summary

Generate per-vOTU interactive (2D or 3D) subgraph views of the vConTACT2 network neighborhood so each taxonomic assignment made by graphanalyzer can be visually inspected against neighboring reference genomes.

## When to use

Use when you need to visually justify or audit taxonomic assignments of vOTUs within a vConTACT2 network, e.g. inspecting which INPHARED reference genomes neighbor each vOTU in its cluster.

## When NOT to use

- When no vConTACT2 network has been built (e.g. classification was done by marker-based tools like Demovir or VirHostMatcher) — there is no graph neighborhood to visualize.
- When you only need batch taxonomic assignments without visual audit — the taxonomy table alone suffices and subgraph rendering adds runtime.
- When input genomes were not named with the --prefix distinguishing vOTUs from reference genomes, the script cannot separate user scaffolds from references.

## Inputs

- c1.ntw graph description file from vConTACT2 (source/target/edge weight for genome pairs above the significance threshold)
- genome_by_genome_overview.csv from vConTACT2 (viral clusters, refined subclusters, confidence metrics)
- Taxonomy metadata file (e.g. INPHARED data_excluding_refseq.tsv mapping GenBank accessions to taxonomy)

## Outputs

- Per-vOTU interactive 2D or 3D subnetwork view (plotly) of the vConTACT2 graph neighborhood
- Supporting taxonomy table CSV with vOTU, assigned taxonomy, Status, and Weight columns

## How to apply

Run graphanalyzer.py on the three vConTACT2/INPHARED inputs — the graph file c1.ntw (source/target/edge-weight pairs), the genome_by_genome_overview.csv (VC, VC_subcluster, and confidence metrics), and the taxonomy metadata file — pointing --output at an existing directory. Pass --prefix (e.g. vOTU) so the script distinguishes user viral scaffolds from reference genomes in the network, and set --view to either 2d (default) or 3d to select the interactive rendering type. The tool uses networkx, pandas, plotly, openpyxl and pygraphviz to extract each vOTU's same-cluster neighborhood and render it as an interactive plotly graph alongside the taxonomy table. Increase --threads to speed up generation across many vOTUs. Use the resulting subgraphs to confirm that the taxonomy inherited by edge-weight ordering matches the visually apparent placement of the vOTU among reference genomes.

## Related tools

- **graphanalyzer** (Generates the per-vOTU interactive subnetwork views (2D/3D, plotly-based) of the vConTACT2 graph supporting each taxonomic assignment) — github.com/lazzarigioele/graphanalyzer
- **vConTACT2** (Produces the network outputs (c1.ntw, genome_by_genome_overview.csv) whose neighborhoods are visualized)
- **MetaPhage** (Upstream Nextflow pipeline that runs vConTACT2 and graphanalyzer on vOTUs and integrates the interactive subgraphs into the report) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
graphanalyzer.py --graph c1.ntw --csv genome_by_genome_overview.csv --metas data_excluding_refseq.tsv --output ./ --prefix vOTU --threads 4 --view 2d
```

## Evaluation signals

- Interactive subgraph files exist for every vOTU in the taxonomy table CSV, matching the --prefix naming scheme (e.g. vOTU_1, vOTU_2).
- Each subgraph's node set corresponds to the vOTU's same-cluster neighbors listed in c1.ntw with edge weights above the significance threshold.
- Reference genome nodes in the subgraph carry taxonomy resolved via the metadata file, and vOTU nodes are visually adjacent to the reference from which taxonomy was inherited.
- Requested view type (--view 2d or 3d) is reflected in the rendered output format.
- In the MetaPhage report, the per-vOTU subnetwork is retrievable and consistent with the Status/Weight fields in the taxonomy table.

## Limitations

- The vConTACT2 network is rebuilt every time new data are added, extending the time required even further, and the all-versus-all protein comparison against large reference databases is time-consuming.
- vConTACT2 does not classify a vOTU directly; the subnetwork is a support view, and the taxonomy still depends on the edge-weight inheritance and conservatism rules.
- Script was tested only under Linux and relies on external libraries (networkx, pandas, plotly, openpyxl, pygraphviz).

## Evidence

- [readme] graphanalyzer.py was written to try to automate the manual inspection of the vConTACT2 outputs, providing the user a direct taxonomic association for every vOTU, plus a bunch of interactive graphs.: [evidence span withheld — anchor status: not_found]
- [readme] --view (or -w) defines which type of interactive graphs to produce. It can be 2d or 3d (default is 2d).: [evidence span withheld — anchor status: not_found]
- [readme] --prefix (or -p) defines the prefix that distinguishes the vOTUs in input from the other genomes.: [evidence span withheld — anchor status: not_found]
- [other] 6. Generate a per-vOTU interactive subnetwork view of the vConTACT2 graph neighborhood supporting each assignment.: [evidence span withheld — anchor status: not_found]
- [discussion] graphanalyzer is available under the same license at https://github.com/lazzarigioele/graphanalyzer: "graphanalyzer is available under the same license at https://github.com/lazzarigioele/graphanalyzer"
