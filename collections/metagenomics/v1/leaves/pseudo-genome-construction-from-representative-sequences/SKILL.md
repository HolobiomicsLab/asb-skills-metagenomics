---
name: pseudo-genome-construction-from-representative-sequences
description: Use when you need to organize homologous genes from many annotated genomes
  into a taxonomy-aware set of pseudo-genomes before designing hybridization-capture
  probes at multiple taxonomic levels.
when_to_use_negative:
- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes
  to efficiently cluster sequences, so probes cannot be designed for such targets
  directly
- You only need probes for a single known sequence rather than hierarchical taxonomic-level
  capture across related organisms
- Taxa are so diverse that clusters cannot be formed at the <=15% divergence threshold,
  leaving nodes without valid representative sequences
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: HUBDesign
  role: Pipeline implementing clustering, LCA assignment, and pseudo-genome construction
    (bin/Clustering.pl, bin/Assignment.pl)
  repo: github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Suggested tool for annotating input genomes before clustering
- name: MAFFT
  role: Supported multiple sequence aligner used via BioPerl to align gene families
    before building neighbour-joining trees
- name: BioPerl
  role: Perl library used to run alignments and build neighbour-joining gene trees
    during clustering
- name: Perl
  role: Runtime for the HUBDesign pipeline scripts (Perl 5.10.0+ required)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:4ec6572c17a3d2d9ddc9236e4906312158a266a5923044d3f4dc4b4b8cc51175
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
  - CLUSTAL OMEGA
  - HUBDesign
  - MAFFT
  - MUSCLE
  - PROKKA
  - Perl
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# pseudo-genome-construction-from-representative-sequences

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

HUBDesign stage that collapses homologous, <=15%-divergent gene sequences into representative consensus sequences, assigns each to the lowest common ancestor (LCA) of the organisms possessing it, and concatenates representatives (with 50% penetrance) into node-level pseudo-genomes for downstream probe identification.

## When to use

Use when you need to organize homologous genes from many annotated genomes into a taxonomy-aware set of pseudo-genomes before designing hybridization-capture probes at multiple taxonomic levels.

## When NOT to use

- Input genomes are unknown or unannotatable — HUBDesign relies on annotated genomes to efficiently cluster sequences, so probes cannot be designed for such targets directly
- You only need probes for a single known sequence rather than hierarchical taxonomic-level capture across related organisms
- Taxa are so diverse that clusters cannot be formed at the <=15% divergence threshold, leaving nodes without valid representative sequences

## Inputs

- PROKKA-annotated GFF files for input genomes (e.g., 56 RefSeq coronavirus genomes)
- GenomeInfo.tab (tab-delimited paths-to-annotated-genome and taxon-id)
- Guide.tree (Newick tree derived from NCBI taxonomy lineages)
- Supported multiple sequence aligner output (MAFFT 7.0+, MUSCLE 3.8.425+, CLUSTAL OMEGA 1.2.1+)

## Outputs

- RepSeq.fna (FASTA of representative consensus sequences for each collapsed cluster)
- Assignment.tsv (tab-delimited LCA assignment of representatives to taxonomic nodes)
- FASTA of node pseudo-genomes (concatenated representatives with flanking buffers)
- A tab delimited file detailing the composition of gene clusters

## How to apply

Run bin/Clustering.pl on the PROKKA-annotated GFF files (e.g., the 56 RefSeq coronavirus genomes listed in GenomeInfo.tab) to group homologous gene sequences into clusters; align each gene family with MAFFT 7.0+, MUSCLE 3.8.425+, or CLUSTAL OMEGA 1.2.1+ via BioPerl, build neighbour-joining gene trees, and collapse sequences at nodes whose root-to-tip divergence is at most 15% into representative consensus sequences written to RepSeq.fna. Then run bin/Assignment.pl to assign each representative to the LCA of the organisms that possess it, guided by the Guide.tree derived from NCBI taxonomy, producing Assignment.tsv. Apply the penetrance threshold (--penetrance 0.5) so a gene is included in a node's pseudo-genome only when present in at least 50% of that node's descendants, and concatenate all representatives assigned to each taxonomic node, with flanking buffers, into node pseudo-genomes exported as FASTA. The 15% divergence cutoff reflects that hybridization to sequences up to 20% divergent is possible, balancing breadth of coverage against probe specificity.

## Related tools

- **HUBDesign** (Pipeline implementing clustering, LCA assignment, and pseudo-genome construction (bin/Clustering.pl, bin/Assignment.pl)) — github.com/zacherydickson/HUBDesign
- **PROKKA** (Suggested tool for annotating input genomes before clustering)
- **MAFFT** (Supported multiple sequence aligner used via BioPerl to align gene families before building neighbour-joining trees)
- **BioPerl** (Perl library used to run alignments and build neighbour-joining gene trees during clustering)
- **Perl** (Runtime for the HUBDesign pipeline scripts (Perl 5.10.0+ required))

## Examples

```
bin/Clustering.pl --r2t-divergence 0.15 test/*.gff > RepSeq.fna && bin/Assignment.pl --penetrance 0.5 --guide-tree test/Guide.tree ClusterInfo.tsv > Assignment.tsv
```

## Evaluation signals

- All 56 input taxa are represented by at least one pseudo-genome in the output FASTA
- RepSeq.fna contains representative consensus sequences only for clusters whose root-to-tip divergence is at most 15%
- Assignment.tsv assigns each representative to the LCA of the organisms possessing it, consistent with the Guide.tree derived from NCBI taxonomy
- A gene appears in a node's pseudo-genome only if present in at least 50% of that node's descendants (--penetrance 0.5)
- Cluster composition file matches the gene cluster composition reported in the pipeline's tab-delimited cluster output

## Limitations

- HUBDesign requires annotated genomes; probes cannot be designed for unknown or unannotatable genomes, only for close relatives
- Published probe sets (e.g., sepsis) were produced with earlier HUBDesign versions whose cluster partitioning used nominal taxonomy rather than observed sequence similarity, causing non-specific probes (e.g., for Streptococcus spp.); current versions construct a new cluster-assignment tree — a version discrepancy with what was validated
- No probes exist for txid1503293 and txid693998 under the published version; --penetrance settings trade off specificity (low values reduce specificity via cross-reactive regions) and inclusion of horizontally transferred elements (high values)

## Evidence
