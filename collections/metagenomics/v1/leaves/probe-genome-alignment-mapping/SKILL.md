---
name: probe-genome-alignment-mapping
description: 'Reference for reanalyzing a published HUBDesign probe set: when you
  have probe sequences plus the annotated source genomes (e.g. GenomeInfo.tab listing
  56 RefSeq coronavirus genomes) and need per-genome probe counts, specificity fractions,
  or comparison to reported extremes.'
when_to_use_negative:
- You need to design a new probe set from genomes rather than audit an existing one
  — use the full HUBDesign pipeline (Clustering through Selection) instead
- The probe set lacks accompanying genome-to-taxon metadata, so BLASTn hits cannot
  be assigned to species/genus
- Goal is experimental capture enrichment analysis (read deduplication, qPCR-amplicon
  removal) rather than in silico probe-to-genome mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BLASTn
  role: Builds a nucleotide database from the annotated source genomes and aligns
    every probe sequence against it (version 2.9.1+ defaults) to recover each probe's
    source genome(s) and hit multiplicity
- name: HUBDesign
  role: Source of the deposited probe FASTA sets and the GenomeInfo.tab / Guide.tree
    inputs being audited by this mapping skill
  repo: github.com/zacherydickson/HUBDesign
- name: PROKKA
  role: Suggested annotation tool used to produce the annotated genomes against which
    probes are aligned (test genomes were created by running PROKKA with kingdom set
    to viruses)
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100069
    title: Probe design for simultaneous, targeted capture of diverse metagenomic
      targets
  source_content_hash: sha256:009cb4701d1c4a2e4020b6da496760c900871ca597901fc159c7f04f928f8a30
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zacherydickson/HUBDesign
  tools:
  - BLASTn
  - HUBDesign
  - PROKKA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# probe-genome-alignment-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns a deposited HUBDesign probe FASTA back to its source annotated genomes with BLASTn (2.9.1+ defaults) to quantify per-genome (or per-species/per-genus) probe density and each probe's taxonomic specificity (single-virus vs multi-virus hits). It validates that the published probe set behaves as described before it is reused for new capture experiments.

## When to use

Reference for reanalyzing a published HUBDesign probe set: when you have probe sequences plus the annotated source genomes (e.g. GenomeInfo.tab listing 56 RefSeq coronavirus genomes) and need per-genome probe counts, specificity fractions, or comparison to reported extremes.

## When NOT to use

- You need to design a new probe set from genomes rather than audit an existing one — use the full HUBDesign pipeline (Clustering through Selection) instead
- The probe set lacks accompanying genome-to-taxon metadata, so BLASTn hits cannot be assigned to species/genus
- Goal is experimental capture enrichment analysis (read deduplication, qPCR-amplicon removal) rather than in silico probe-to-genome mapping

## Inputs

- Probe sequences in FASTA format from the HUBDesign GitHub repository (e.g. probes/Coronavirus, probes/Sepsis directories)
- Genome-to-taxon metadata table (GenomeInfo.tab: paths to annotated genomes and taxon IDs)
- Annotated genome FASTA files (e.g. 56 RefSeq coronavirus genomes, or 1,926 sepsis bacterial genomes from 81 species / 35 genera; PROKKA-annotated)

## Outputs

- Tab-delimited per-genome probe-count table (e.g. coronavirus per-taxon counts; sepsis_species_probe_counts.tsv)
- Probe specificity distribution summary (fraction of probes hitting a single genome vs multiple genomes; number of distinct genomes hit per probe)
- Per-species and per-genus probe count tables for multi-genome probe sets

## How to apply

Load the deposited probe FASTA (e.g. probes/Coronavirus or probes/Sepsis from the HUBDesign GitHub) together with the genome-to-taxon metadata such as GenomeInfo.tab. Build a BLASTn database from the annotated genomes (e.g. the 56 RefSeq coronaviruses or the 1,926 sepsis bacterial genomes) and align every probe with BLASTn version 2.9.1+ defaults to identify each probe's source genome(s). Assign each probe's taxon of origin from its best hit and tabulate probes per taxon, verifying expected patterns (8 probes for each of the 23 gammacoronaviruses; zero probes for txid1503293 and txid693998). Classify each probe's specificity by the number of distinct genomes it hits, and for multi-taxon sets aggregate hits to species and genus using a permissive identity threshold because probes may hybridize to sequences up to 20% divergent. Finally write a tab-delimited per-genome probe-count table and a specificity distribution summary, checking the reported invariants (62.5% single-virus coronavirus probes; sepsis median 478 probes/species, all 1,926 strains genus-covered, 53.3% species-covered).

## Related tools

- **BLASTn** (Builds a nucleotide database from the annotated source genomes and aligns every probe sequence against it (version 2.9.1+ defaults) to recover each probe's source genome(s) and hit multiplicity)
- **HUBDesign** (Source of the deposited probe FASTA sets and the GenomeInfo.tab / Guide.tree inputs being audited by this mapping skill) — github.com/zacherydickson/HUBDesign
- **PROKKA** (Suggested annotation tool used to produce the annotated genomes against which probes are aligned (test genomes were created by running PROKKA with kingdom set to viruses))

## Examples

```
makeblastdb -in coronavirus_genomes.fna -dbtype nucl -out corona_db; blastn -query probes.fna -db corona_db -outfmt 6 -out probe_hits.tsv
```

## Evaluation signals

- Coronavirus per-taxon counts match the expected pattern: exactly 8 probes for each of the 23 gammacoronaviruses, and zero probes for txid1503293 and txid693998
- Coronavirus specificity distribution reproduces the reported 62.5% of probes specific to a single virus, with 78.1% of multi-virus probes targeting only two or three
- Sepsis species-level probe counts reproduce reported Table S6 extremes: C. botulinum 12, C. perfringens 44, C. tetani 71, R. prowazekii 53, B. burgdorferi 90, median 478; spiked strains from 110 (S. sanguinis) to 564 (B. multivorans)
- All 1,926 sepsis strains have at least genus-level probe coverage and 53.3% have species-level coverage
- Output tables are valid TSV with one row per genome/species and counts summing to the total deposited probe count

## Limitations

- Probes may hybridize to sequences up to ~20% divergent, so a permissive identity threshold is needed and hit counts reflect cross-reactivity, not just exact origins
- The deposited sepsis probe set was produced with an earlier HUBDesign version having a cluster-partitioning flaw (nominal taxonomy rather than observed sequence similarity), which affects its specificity profile
- Under the published version no probes exist for txid1503293 and txid693998; the current HUBDesign instead constructs a cluster-assignment tree, so reproducing probe sets may differ by version

## Evidence

- [other] Build a BLASTn database from the 56 annotated coronavirus genomes and align every probe against it using BLASTn (version 2.9.1+ defaults) to identify each probe's source genome(s).: [evidence span withheld — anchor status: not_found]
