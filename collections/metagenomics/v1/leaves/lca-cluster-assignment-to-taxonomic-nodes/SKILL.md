---
name: lca-cluster-assignment-to-taxonomic-nodes
description: 'Reference for designing probes at multiple taxonomic levels: when annotated
  genomes and a guide tree are available and candidate probes must target node-level
  (shared) sequences rather than individual genomes.'
when_to_use_negative:
- Input genomes are unannotated or unannotatable — HUBDesign relies on annotated genomes
  to cluster sequences, and probes cannot be designed for targets with unknown genomes
- Only a single organism/genome is targeted and no taxonomic hierarchy or cross-strait
  breadth of coverage is needed — node-level LCA assignment adds no value
- Sequences within clusters are so divergent that a 15% r2t-divergence cutoff cannot
  form meaningful representative sequences
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: HUBDesign
  role: Pipeline wrapping the assignment phase; bin/Assignment.pl assigns clusters
    to LCA nodes guided by the provided tree
  repo: github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Annotates the input genomes (kingdom set to viruses) so genes can be clustered
    and assigned
- name: MAFFT
  role: Multiple sequence alignment used to build representative sequences that the
    assignment step maps to nodes
- name: BioPerl
  role: Perl library dependency (1.7.10+) for parsing tree and sequence formats in
    the assignment scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:0c79bc6181efd852c365c9b7b8114189846f18511c01e49ec9b9e70c80b4bc84
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BioPerl
  - HUBDesign
  - MAFFT
  - PROKKA
  - Perl
  - SA_BOND
  - sdust
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lca-cluster-assignment-to-taxonomic-nodes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

HUBDesign's assignment phase assigns representative sequences (produced by clustering genes ≤15% divergent) to the lowest common ancestor (LCA) of the organisms possessing them, then concatenates representatives per node into pseudo-genomes for taxonomically hierarchical probe design.

## When to use

Reference for designing probes at multiple taxonomic levels: when annotated genomes and a guide tree are available and candidate probes must target node-level (shared) sequences rather than individual genomes.

## When NOT to use

- Input genomes are unannotated or unannotatable — HUBDesign relies on annotated genomes to cluster sequences, and probes cannot be designed for targets with unknown genomes
- Only a single organism/genome is targeted and no taxonomic hierarchy or cross-strait breadth of coverage is needed — node-level LCA assignment adds no value
- Sequences within clusters are so divergent that a 15% r2t-divergence cutoff cannot form meaningful representative sequences

## Inputs

- Representative sequences FASTA (RepSeq.fna) from the Clustering phase (genes ≤15% divergence collapsed via MAFFT)
- ClusterInfo / cluster composition files output by Clustering.pl
- Newick-formatted guide tree of input taxa (e.g., test/Guide.tree built from NCBI taxonomy lineages of 56 RefSeq coronavirus genomes)
- GenomeInfo.tab mapping annotated genome paths (PROKKA, kingdom=viruses) to NCBI taxon-ids

## Outputs

- Assignment.tsv tab-delimited file mapping representative sequences to LCA taxonomic nodes
- Node pseudo-genomes: concatenated representative sequences assigned to each tree node

## How to apply

After the Clustering phase collapses genes with sequences at most 15% divergent into representative sequences (RepSeq.fna via MAFFT alignment), run the Assignment phase (`bin/Assignment.pl`) with the ClusterInfo and the guide tree so each representative sequence is assigned to the lowest common ancestor of the organisms that possess it. Apply the penetrance threshold (0.5 in the coronavirus design) so a gene is included in a node's pseudo-genome only if at least that proportion of the node's descendants possess it — low penetrance eliminates many regions and reduces cross-reactivity control, high penetrance can admit non-identifying horizontally transferred elements. Concatenate all representatives assigned to each node into a pseudo-genome for that organism; these pseudo-genomes then feed SA_BOND-based candidate identification. Node-level assignment is what allows probes to simultaneously and specifically capture both known and related (novel) sequences.

## Related tools

- **HUBDesign** (Pipeline wrapping the assignment phase; bin/Assignment.pl assigns clusters to LCA nodes guided by the provided tree) — github.com/zacherydickson/HUBDesign
- **PROKKA** (Annotates the input genomes (kingdom set to viruses) so genes can be clustered and assigned)
- **MAFFT** (Multiple sequence alignment used to build representative sequences that the assignment step maps to nodes)
- **BioPerl** (Perl library dependency (1.7.10+) for parsing tree and sequence formats in the assignment scripts)

## Examples

```
bin/Assignment.pl -tree test/Guide.tree -penetrance 0.5 ClusterInfo.tab RepSeq.fna > Assignment.tsv
```

## Evaluation signals

- Assignment.tsv is produced and each representative sequence maps to a node in the guide tree consistent with the NCBI taxonomy lineages of the input genomes
- With penetrance 0.5, node pseudo-genomes include only genes present in ≥50% of a node's descendant genomes; lowering penetrance should visibly expand pseudo-genome content
- In the reference 56-genome coronavirus run, downstream selection of 13,500 probes yields 25.0% nucleotide coverage, 4.72x depth, and efficiency 1.87 matching Table 1
- Intermediate files are retained (--keep) and RepSeq.fna / Assignment.tsv formats are preserved for independent re-running of later phases
- The pipeline completes without the legacy flaw of partitioning clusters by nominal taxonomy rather than observed sequence similarity (fixed in newer versions)

## Limitations

- Low penetrance values eliminate many regions from consideration for cross-reactivity, reducing specificity of the final probe set; high values may admit non-identifying horizontally transferred elements
- The published sepsis design partitioned clusters by nominal taxonomy rather than observed sequence similarity, causing truly non-specific probes (e.g., Streptococcus spp.); the current version instead constructs a cluster-assignment tree from observed clusters, optionally guided by a user tree — a version discrepancy with what was validated
- No probes exist for some taxa (txid1503293, txid693998) under the published version
- Assignment quality depends entirely on upstream clustering correctness and on the guide tree (default hierarchy is co-occurrence of clusters between taxa if no tree is given)

## Evidence
