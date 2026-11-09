---
name: upset-set-intersection-analysis
description: Use when several co-occurrence networks have been inferred from the same
  counts table by different methods (e.g., SparCC, propr, SpiecEasi, FlashWeave, COZINE,
  HARMONIES, SPRING, mLDM) and you need to quantify which nodes/edges they share.
when_to_use_negative:
- You have only one inferred network — intersection analysis needs multiple networks
  from the same underlying abundance table
- Networks are built from different datasets or different taxonomy groupings; intersection
  counts would not be comparable
- The goal is abundance/OTU comparison rather than comparing network topology (edge
  and node overlap)
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: MiCoNE
  role: Python package providing the Network objects used to load edge lists and compute
    node/edge intersections across methods
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Executes the 'ni' workflow template (micone init -w ni) that runs the eight
    inference tools with bootstraps and network construction
- name: Python
  role: Language in which interaction matrices/edge lists are loaded into Network
    objects for the UpSet intersection analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:4ebd75f3df882c15e40ac895743d03b97cfa60593d9a3ae0f145a7a33dd69460
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/segrelab/MiCoNE
  tools:
  - COZINE
  - FlashWeave
  - HARMONIES
  - MiCoNE
  - Nextflow
  - Python
  - SPRING
  - SparCC
  - SpiecEasi
  - mLDM
  - propr
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# UpSet set-intersection analysis of microbial co-occurrence networks

## Summary

Quantifies shared and unique nodes and edges across networks inferred by multiple network inference (NI) methods using UpSet-style set intersection analysis, as done in MiCoNE to reveal substantial rewiring despite taxonomic overlap.

## When to use

Use when several co-occurrence networks have been inferred from the same counts table by different methods (e.g., SparCC, propr, SpiecEasi, FlashWeave, COZINE, HARMONIES, SPRING, mLDM) and you need to quantify which nodes/edges they share.

## When NOT to use

- You have only one inferred network — intersection analysis needs multiple networks from the same underlying abundance table
- Networks are built from different datasets or different taxonomy groupings; intersection counts would not be comparable
- The goal is abundance/OTU comparison rather than comparing network topology (edge and node overlap)

## Inputs

- Genus-level OTU/count tables (otu_table.tsv) with Kingdom-to-Species taxonomy columns from the MiCoNE Zenodo deposit (record 7051556 data.zip)
- Sample and observation metadata (obs_metadata.tsv, sample_metadata.tsv)
- Eight filtered network files (JSON, with 'source', 'target', 'weight' edge fields), one per NI method

## Outputs

- Node and edge intersection counts across the eight NI methods (UpSet overlap analysis, reproducing Fig. 4B/C)
- Lists of shared and method-specific nodes and edges (e.g., 8/202 shared edges; 33/68 shared nodes, 0 unique nodes)

## How to apply

Run the eight NI tools from the MiCoNE pipeline ('micone init -w ni'), each preceded by 1000 Resample bootstraps and Pvalue computation, then convert each bootstrapped result into a network with the pvalue-based Make network step using pvalue_threshold 0.05 and interaction_threshold 0.3. Group nodes at the genus taxonomic level (tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']) and write each filtered network with network_group.write('network.', pvalue_filter=True, interaction_filter=False). Load the eight resulting edge lists (with 'source', 'target', 'weight' fields) into MiCoNE Network objects and compute node and edge intersections across methods (UpSet analysis). In the paper's FMT genus-level data this yields 8 shared edges out of 202 unique edges and 33 shared nodes out of 68, with no method producing a unique node — quantifying substantial rewiring between methods that share taxa.

## Related tools

- **MiCoNE** (Python package providing the Network objects used to load edge lists and compute node/edge intersections across methods) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Executes the 'ni' workflow template (micone init -w ni) that runs the eight inference tools with bootstraps and network construction)
- **Python** (Language in which interaction matrices/edge lists are loaded into Network objects for the UpSet intersection analysis)

## Examples

```
micone init -w ni -o <pipeline_dir>
```

## Evaluation signals

- All eight network JSON files exist and each contains 'source', 'target', 'weight' edge fields
- Node and edge intersection counts qualitatively match the overlap pattern reported in Fig. 4B/C (≈8 shared edges of 202; ≈33 shared nodes of 68; no unique node)
- Every method's network was filtered identically: pvalue_filter=True, interaction_filter=False, at pvalue_threshold 0.05 and interaction_threshold 0.3 after 1000 bootstraps
- Nodes were grouped at the same taxonomic level (Genus) in all eight networks before computing intersections

## Limitations

- Only 8 of 202 edges are shared across all methods, so consensus edges are rare and cross-method agreement on edges is low despite full node overlap
- The pipeline requires the mamba package manager and conda environments for each tool; a preinstalled julia may conflict with the micone-flashweave environment
- The data directory (nf_micone/data) must be manually downloaded from Zenodo record 7051556

## Evidence

- [results] only 8 edges (out of 202 total unique edges) are in common between all the methods, while 33 out of 68 unique nodes are shared across all networks: [evidence span withheld — anchor status: not_found]
- [readme] MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow: "MiCoNE`` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow"
- [methods] If you have interaction data stored in the form of an edge list: "If you have interaction data stored in the form of an edge list"
- [methods] Convert each bootstrapped result into a network using the pvalue-based Make network step with pvalue_threshold 0.05 and interaction_threshold 0.3: [evidence span withheld — anchor status: not_found]
