---
name: taxonomic-specificity-classification
description: Use when you have a deposited probe set (e.g. from the HUBDesign probes/Coronavirus
  directory) plus the annotated source genomes, and need to determine what fraction
  of probes are genome-specific versus cross-reactive across multiple viruses.
when_to_use_negative:
- Probes cannot be meaningfully classified for genomes that are unknown or unannotatable,
  since HUBDesign relies on annotated genomes to cluster sequences and probes only
  exist for targets with known genomes
- If the goal is capture enrichment validation (wet-lab hybridization performance),
  this BLASTn-based in silico specificity mapping does not measure enrichment efficiency,
  which varies per probe set depending on target sequence properties
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: BLASTn
  role: Aligns each probe against the BLASTn database built from the annotated source
    genomes (version 2.9.1+ defaults) to identify which genome(s) each probe hits,
    enabling the single-virus vs multi-virus specificity classification
  repo: https://github.com/zacherydickson/HUBDesign
- name: HUBDesign
  role: Source of the deposited probe set and GenomeInfo.tab; the probes being classified
    were designed by bin/HUBDesign using the guide tree from NCBI taxonomy lineages
  repo: https://github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Suggested annotation tool used to produce the annotated genome files that
    constitute the BLASTn database for mapping probes back
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:1fdfb75ba9598f65c685ab6fb73fbd6a389148a7a948ebd3c3439add4df3f13a
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
  - PROKKA
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# taxonomic-specificity-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify each probe in a HUBDesign-designed probe set by the number of distinct genomes it hits after BLASTn mapping back to the source genomes, yielding a per-genome probe-count table and a single-virus vs multi-virus specificity distribution. This quantifies how specific the deposited probe set is at the taxonomic level of individual genomes.

## When to use

Use when you have a deposited probe set (e.g. from the HUBDesign probes/Coronavirus directory) plus the annotated source genomes, and need to determine what fraction of probes are genome-specific versus cross-reactive across multiple viruses.

## When NOT to use

- Probes cannot be meaningfully classified for genomes that are unknown or unannotatable, since HUBDesign relies on annotated genomes to cluster sequences and probes only exist for targets with known genomes
- If the goal is capture enrichment validation (wet-lab hybridization performance), this BLASTn-based in silico specificity mapping does not measure enrichment efficiency, which varies per probe set depending on target sequence properties

## Inputs

- Probe sequences (FASTA) from the deposited HUBDesign probe set, e.g. probes/Coronavirus directory
- Annotated source genomes (PROKKA-annotated RefSeq files, e.g. 56 RefSeq coronavirus genomes) with a tab-delimited GenomeInfo.tab mapping genome paths to taxon-ids
- NCBI taxonomy lineages used to generate the Guide.tree (optional reference for taxon naming)

## Outputs

- Tab-delimited per-genome probe-count table (probes assigned per taxon from BLASTn hits)
- Specificity distribution summary (single-virus vs multi-virus probe fractions, e.g. 62.5% single-virus)
- Per-probe assignment of source genome(s) from BLASTn hits

## How to apply

Load the probe FASTA sequences and the annotated RefSeq genomes listed in the GenomeInfo.tab file (56 RefSeq coronavirus genomes in the test set, each with its taxon-id). Build a BLASTn database from the annotated genomes and align every probe against it with BLASTn version 2.9.1+ defaults to assign each probe to its source genome(s). Tabulate probes per taxon from the BLASTn hits, verifying the expected pattern (e.g. 8 probes for each of the 23 gammacoronaviruses, zero for txid1503293 and txid693998). Then classify each probe by the number of distinct genomes it hits (single-virus vs multi-virus) and compute the specificity distribution, checking the single-virus fraction against the reported 62.5%. Finally write a tab-delimited per-genome probe-count table and a companion specificity distribution summary.

## Related tools

- **BLASTn** (Aligns each probe against the BLASTn database built from the annotated source genomes (version 2.9.1+ defaults) to identify which genome(s) each probe hits, enabling the single-virus vs multi-virus specificity classification) — https://github.com/zacherydickson/HUBDesign
- **HUBDesign** (Source of the deposited probe set and GenomeInfo.tab; the probes being classified were designed by bin/HUBDesign using the guide tree from NCBI taxonomy lineages) — https://github.com/zacherydickson/HUBDesign
- **PROKKA** (Suggested annotation tool used to produce the annotated genome files that constitute the BLASTn database for mapping probes back)

## Examples

```
makeblastdb -in genomes/*.fna -dbtype nucl; blastn -query probes/Coronavirus/probes.fna -db genomes -out hits.tsv -outfmt 6
```

## Evaluation signals

- Per-genome probe counts match the expected pattern: 8 probes for each of the 23 gammacoronaviruses and zero probes for txid1503293 and txid693998
- The computed single-virus probe fraction matches the reported 62.5%
- Among multi-virus probes, the majority (78.1%) target only two or three genomes
- Every probe in the deposited FASTA receives at least one BLASTn hit against the 56-genome database (no unassigned probes)
- Output table is tab-delimited with one row per taxon whose counts sum to the total number of deposited probes (counting multi-genome probes per hit)

## Limitations

- No probes exist for txid1503293 and txid693998 under the published version; the current version of HUBDesign constructs a new cluster-assignment tree, so re-running the design may not reproduce the deposited probe set exactly
- The sepsis probe set produced with an earlier HUBDesign version has a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), so specificity classification of probe sets from older versions may reveal non-truly-specific probes, e.g. highly divergent Streptococcus spp. probes
- In silico BLASTn specificity does not guarantee wet-lab enrichment; enrichment is dependent on target sequence properties and will vary for each probe set produced

## Evidence

- [methods] Build a BLASTn database and align every probe against the source genomes: [evidence span withheld — anchor status: not_found]
