---
name: coronavirus-genome-taxon-reconciliation
description: Use when a deposited HUBDesign probe set (e.g., probes/Coronavirus FASTA
  plus a GenomeInfo.tab of taxon IDs) must be reconciled against its source genomes
  to check per-taxon probe counts and single- vs multi-virus specificity fractions.
when_to_use_negative:
- You need to design a new probe set rather than audit a deposited one — run the full
  HUBDesign pipeline (Clustering, Assignment, Identification, Filtering, Selection)
  instead
- The target genomes are unknown or unannotatable — HUBDesign relies on annotated
  genomes and cannot design probes for such targets
- You want wet-lab capture performance (fold enrichment from sequenced reads), not
  in silico probe-to-genome specificity
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3293
tools:
- name: BLASTn
  role: Builds a nucleotide database from the 56 annotated coronavirus genomes and
    aligns each probe (version 2.9.1+ defaults) to identify its source genome(s)
- name: HUBDesign
  role: Source of the deposited coronavirus probe set being reconciled; its GenomeInfo.tab
    input format defines the genome-to-taxon mapping
  repo: https://github.com/zacherydickson/HUBDesign
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:644adc0fa6f1b088792281ea5eb7d2bea936ad9f0ba9c3d0fc750b9050de7fe3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  repo_url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - HUBDesign
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# coronavirus-genome-taxon-reconciliation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Audits a deposited HUBDesign coronavirus probe set by mapping every probe back to the 56 RefSeq coronavirus genomes it was designed from, using BLASTn to quantify per-genome probe density and hierarchical taxonomic specificity (single-virus vs multi-virus targeting).

## When to use

Use when a deposited HUBDesign probe set (e.g., probes/Coronavirus FASTA plus a GenomeInfo.tab of taxon IDs) must be reconciled against its source genomes to check per-taxon probe counts and single- vs multi-virus specificity fractions.

## When NOT to use

- You need to design a new probe set rather than audit a deposited one — run the full HUBDesign pipeline (Clustering, Assignment, Identification, Filtering, Selection) instead
- The target genomes are unknown or unannotatable — HUBDesign relies on annotated genomes and cannot design probes for such targets
- You want wet-lab capture performance (fold enrichment from sequenced reads), not in silico probe-to-genome specificity

## Inputs

- FASTA file of deposited HUBDesign probe sequences (probes/Coronavirus directory)
- GenomeInfo.tab listing the 56 RefSeq coronavirus genome paths and taxon IDs
- BLASTn database built from the 56 annotated coronavirus RefSeq genomes

## Outputs

- Tab-delimited per-genome probe-count table (probes assigned per taxon)
- Specificity distribution summary (fraction of single-virus vs multi-virus probes, and genome-count histogram for multi-virus probes)
- Per-probe hit list mapping each probe to its source genome(s) from BLASTn

## How to apply

Load the deposited probe sequences from the HUBDesign repository's probes/Coronavirus directory together with the GenomeInfo.tab listing the 56 RefSeq coronavirus genomes and their taxon IDs. Build a BLASTn nucleotide database from the 56 annotated genomes and align every probe against it with BLASTn (version 2.9.1+ defaults) so each probe is assigned to the genome(s) it matches. Tabulate probes per taxon and verify the expected pattern: 8 probes for each of the 23 gammacoronaviruses and zero probes for txid1503293 and txid693998. Classify each probe by the number of distinct genomes it hits — single-virus vs multi-virus — and compute the specificity distribution, checking the single-virus fraction against the reported 62.5% and that 78.1% of multi-virus probes target only two or three genomes. Write a tab-delimited per-genome probe-count table plus a companion specificity-distribution summary; note that zero-probe taxa reflect the published version's cluster-assignment behavior, which newer HUBDesign versions changed.

## Related tools

- **BLASTn** (Builds a nucleotide database from the 56 annotated coronavirus genomes and aligns each probe (version 2.9.1+ defaults) to identify its source genome(s))
- **HUBDesign** (Source of the deposited coronavirus probe set being reconciled; its GenomeInfo.tab input format defines the genome-to-taxon mapping) — https://github.com/zacherydickson/HUBDesign

## Examples

```
makeblastdb -in genomes.fna -dbtype nucl -out corona_db; blastn -query probes.fna -db corona_db -outfmt 6 -out probe_hits.tsv
```

## Evaluation signals

- Per-taxon counts show exactly 8 probes for each of the 23 gammacoronaviruses
- Zero probes are assigned to txid1503293 and txid693998
- Single-virus fraction of probes equals the reported 62.5%
- Among multi-virus probes, 78.1% target only two or three distinct genomes
- Re-running BLASTn (2.9.1+ defaults) against the 56-genome database reproduces the same per-genome probe counts

## Limitations

- No probes exist for txid1503293 and txid693998 under the published version; the current HUBDesign constructs a new cluster-assignment tree, so results may differ by pipeline version
- HUBDesign requires annotated genomes; probes cannot be designed for targets with unknown or unannotatable genomes
- Probe sets made with earlier HUBDesign versions can contain non-specific probes due to a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity)

## Evidence

- [readme] The Guide.tree file was generated from the lineages provided in NCBI's taxonomy for 56 refseq coronavirus genomes: "The Guide.tree file was generated from the lineages provided in NCBI's taxonomy for 56 refseq coronavirus genomes"
- [discussion] the current version of HUBDesign constructs a new tree to be used during cluster assignment: "the current version of HUBDesign constructs a new tree to be used during cluster assignment, which is based on the observed clusters and optionally guided by a user-provided tree"
- [discussion] processing 56 viral genomes is a minor task compared with the capabilities of the pipeline: "processing 56 viral genomes is a minor task compared with the capabilities of the pipeline"
- [methods] Filtering against NCBI's non-redundant nucleotide database for non-viral hits: "Filtering against NCBI's non-redundant nucleotide database for non-viral hits"
