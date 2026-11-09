---
name: genus-level-taxonomic-aggregation
description: Use when OTU-level networks inferred from a genus-annotatable counts
  table show unstable node identities across methods, and the analysis goal is to
  quantify shared nodes/edges (e.g., UpSet intersections) at a coarser taxonomic rank.
when_to_use_negative:
- Input abundance table lacks taxonomy annotations (no Kingdom-to-Species columns),
  so Group cannot collapse nodes at any level
- The analysis goal is strain- or OTU-level resolution of individual edges rather
  than cross-method comparison of coarse-grained network structure
- Raw 16S reads still need sequence processing (demultiplexing, trimming, OTU assignment)
  before networks exist to be grouped
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MiCoNE
  role: Provides the 'Group' transform (tax_levels parameter) and Network object write()
    method used to aggregate nodes at genus level and export filtered networks
  repo: https://github.com/segrelab/MiCoNE
- name: Nextflow
  role: Executes the ni pipeline template ('micone init -w ni') in which the Group
    step runs after network inference and Make network
  repo: https://github.com/segrelab/MiCoNE
- name: Python
  role: Used to load the grouped edge lists into MiCoNE Network objects and compute
    node/edge intersections across the eight methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1128/msystems.00961-22
    title: 'Inferring microbial co-occurrence networks from amplicon data: a systematic
      evaluation'
  source_content_hash: sha256:aa03fe089ac87982d890c51d3af9fe8c17692475f1ac83ce426eadea8b4d829a
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9565217391304348
n_claims: 23
---

# genus-level-taxonomic-aggregation

## Summary

Aggregate OTU-level co-occurrence networks by collapsing nodes at defined taxonomic levels (Phylum through Species) using the MiCoNE 'Group' transform, so that cross-method network comparisons operate on genus-level nodes rather than noisy OTU-level identifiers.

## When to use

Use when OTU-level networks inferred from a genus-annotatable counts table show unstable node identities across methods, and the analysis goal is to quantify shared nodes/edges (e.g., UpSet intersections) at a coarser taxonomic rank.

## When NOT to use

- Input abundance table lacks taxonomy annotations (no Kingdom-to-Species columns), so Group cannot collapse nodes at any level
- The analysis goal is strain- or OTU-level resolution of individual edges rather than cross-method comparison of coarse-grained network structure
- Raw 16S reads still need sequence processing (demultiplexing, trimming, OTU assignment) before networks exist to be grouped

## Inputs

- otu_table.tsv (genus-level counts table)
- obs_metadata.tsv with Kingdom-to-Species taxonomy columns
- sample_metadata.tsv
- Bootstrapped interaction/pvalue matrices from each NI method

## Outputs

- Grouped (genus-level) network files written as 'network.*' JSON-compatible edge lists with 'source', 'target', 'weight' fields
- Node and edge intersection counts across methods (UpSet analysis input, cf. Fig. 4B/C)

## How to apply

After inferring bootstrapped networks (e.g., 1000 Resample bootstraps followed by Pvalue computation and Make network with pvalue_threshold 0.05 and interaction_threshold 0.3), apply the MiCoNE 'Group' transform with tax_levels ['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species'], which requires the Kingdom-to-Species taxonomy columns of obs_metadata.tsv. Grouping merges OTU nodes assigned to the same taxon at the chosen level, stabilizing node identity so that intersections across the eight NI methods (sparcc, propr, spieceasi, flashweave, cozine, harmonies, spring, mldm) are comparable. Write each filtered grouped network via network_group.write('network.', pvalue_filter=True, interaction_filter=False), preserving pvalue-based edge filtering from the Make network step. Rationale: OTU-level nodes are method- and run-specific, whereas genus-level grouping reveals that 33 of 68 unique nodes are shared across all methods even though only 8 of 202 edges are conserved.

## Related tools

- **MiCoNE** (Provides the 'Group' transform (tax_levels parameter) and Network object write() method used to aggregate nodes at genus level and export filtered networks) — https://github.com/segrelab/MiCoNE
- **Nextflow** (Executes the ni pipeline template ('micone init -w ni') in which the Group step runs after network inference and Make network) — https://github.com/segrelab/MiCoNE
- **Python** (Used to load the grouped edge lists into MiCoNE Network objects and compute node/edge intersections across the eight methods)

## Examples

```
micone init -w ni -o pipeline_dir && bash run.sh  # then in Python: net = Network.load('network.sparcc.tsv'); net.group(tax_levels=['Phylum','Class','Order','Family','Genus','Species']); net.write('network.', pvalue_filter=True, interaction_filter=False)
```

## Evaluation signals

- All eight grouped network JSON files exist and each contains 'source', 'target', 'weight' edge fields
- pvalue_filter=True was applied on write (edges retained pass pvalue_threshold 0.05 from the Make network step), while interaction_filter=False leaves interaction_threshold 0.3 unenforced at export
- Node intersection across the eight methods shows no method producing a unique node, with 33 of 68 unique nodes shared by all methods
- Edge intersection counts qualitatively match Fig. 4B/C: only 8 of 202 total unique edges common to all methods (substantial rewiring)
- Grouped node count is consistent with genus-level collapsing: fewer nodes than the OTU-level networks and tied to distinct genera in obs_metadata.tsv

## Limitations

- Grouping discards OTU-level resolution; method-specific edges lost by aggregation reflect rewiring (only 8/202 edges conserved) even when nodes are fully shared
- The MiCoNE Python library installed via pip does not provide pipeline execution functionality, so Group must run inside the full conda/Nextflow pipeline
- MiCoNE requires the mamba package manager, otherwise 'micone init' will not work; the FlashWeave julia environment can conflict with a preinstalled julia

## Evidence

- [methods] Group nodes at the genus taxonomic level and write each filtered network with network_group.write('network.', pvalue_filter=True, interaction_filter=False): [evidence span withheld — anchor status: not_found]
- [methods] Group is a Transform step with a tax_levels config option: "| Transform | Group              | tax_levels        | "['Phylum', 'Class', 'Order', 'Family', 'Genus', 'Species']" |"
- [methods] Python library can load interaction data into Network objects: "If you have an interaction matrix stored in a ``tsv`` or ``csv`` format, you can load it into a ``Network`` object"
- [results] 33 of 68 unique nodes shared across all methods while only 8 of 202 edges are conserved: [evidence span withheld — anchor status: not_found]
- [readme] MiCoNE is a Python package for exploring tool effects in the 16S workflow: "`MiCoNE` is a Python package for the exploration of the effects of various possible tools used during the 16S data processing workflow on the inferred co-occurrence networks."
