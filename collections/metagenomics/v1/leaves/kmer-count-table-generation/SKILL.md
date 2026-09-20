---
name: kmer-count-table-generation
description: 'Use when preparing inputs for decOM in a source-tracking benchmark:
  shotgun metagenome reads (e.g., BioProjects PRJNA1161616 and PRJNA1092107) must
  be converted to k-mer count tables because decOM does not accept Kraken2 taxonomic
  profiles.'
when_to_use_negative:
- When running SourceApp itself — SourceApp maps reads to source-associated genome
  databases (CoverM-based, percent-identity and query-coverage filtered) and does
  not consume k-mer count tables
- When preparing inputs for FEAST or metaSourceTracker — these tools require Kraken2
  taxonomic profiles, not kmtricks k-mer count tables
- When only apportionment accuracy of SourceApp is needed — k-mer tables are solely
  a format-conversion step for the decOM alternative
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: kmtricks
  role: Generates the k-mer count tables from the mesocosm and fecal slurry shotgun
    reads that decOM requires as input
- name: decOM
  role: Alternative source tracking tool that consumes the generated k-mer count tables
    for source attribution on the mesocosm benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.est.5c03603
    title: 'SourceApp: A Novel Metagenomic Source Tracking Tool that can Distinguish
      between Fecal Microbiomes Using Genome-To-Source Associations Benchmarked Against
      Mixed Input Spike-In Mesocosms'
  source_content_hash: sha256:4c8929846edeff12a44b75d1ea4b76e0b6aec9c7dc4f00903486bc5dbf324d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FEAST
  - Kraken2
  - Python
  - SourceApp
  - decOM
  - kmtricks
  - metaSourceTracker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# k-mer count table generation

## Summary

Generate k-mer count tables with kmtricks to serve as the required input format for decOM when benchmarking alternative metagenomic source tracking tools (FEAST, metaSourceTracker, decOM) against SourceApp on the spike-in mesocosm data.

## When to use

Use when preparing inputs for decOM in a source-tracking benchmark: shotgun metagenome reads (e.g., BioProjects PRJNA1161616 and PRJNA1092107) must be converted to k-mer count tables because decOM does not accept Kraken2 taxonomic profiles.

## When NOT to use

- When running SourceApp itself — SourceApp maps reads to source-associated genome databases (CoverM-based, percent-identity and query-coverage filtered) and does not consume k-mer count tables
- When preparing inputs for FEAST or metaSourceTracker — these tools require Kraken2 taxonomic profiles, not kmtricks k-mer count tables
- When only apportionment accuracy of SourceApp is needed — k-mer tables are solely a format-conversion step for the decOM alternative

## Inputs

- Shotgun mesocosm metagenome reads from NCBI BioProject PRJNA1161616
- Fecal spike-in slurry reads from NCBI BioProject PRJNA1092107 (source training data)
- decOM documentation specifying kmtricks parameters and expected table format

## Outputs

- k-mer count tables (kmtricks output) for mesocosm samples, formatted for decOM
- k-mer count tables for fecal slurry/source samples used as decOM training data
- decOM source attribution predictions scored against SI Table S1 ground truth for the tool_comparison_table.csv

## How to apply

Download the shotgun mesocosm metagenome reads from NCBI BioProject PRJNA1161616 and the fecal spike-in slurry reads from PRJNA1092107, since the same reads must feed all compared tools. Run kmtricks on both the mesocosm and the fecal slurry/source metagenomes to produce k-mer count tables, strictly following the decOM documentation for the chosen k-mer size and output format so the tables are directly consumable by decOM. These tables are specific to decOM: metaSourceTracker and FEAST instead require Kraken2 taxonomic profiles generated with default settings. Run decOM on the mesocosm benchmark samples using the fecal slurry/source k-mer profiles as source training data, then score its source attribution predictions against the ground truth cell fractions (Supporting Information Table S1) to compute sensitivity and specificity alongside SourceApp's tuned results (0.898 specificity, 0.902 sensitivity).

## Related tools

- **kmtricks** (Generates the k-mer count tables from the mesocosm and fecal slurry shotgun reads that decOM requires as input)
- **decOM** (Alternative source tracking tool that consumes the generated k-mer count tables for source attribution on the mesocosm benchmark)

## Evaluation signals

- kmtricks produces k-mer count tables for both the mesocosm (PRJNA1161616) and fecal slurry (PRJNA1092107) read sets using a consistent k-mer vocabulary across samples
- Generated tables load in decOM without reformatting errors, confirming conformance to the decOM-documented input specification
- decOM source attribution predictions on the mesocosm samples can be scored against the ground truth cell fractions in Supporting Information Table S1
- Computed decOM specificity falls at or near 0 and sensitivity is high (per the reported benchmark: 0.74 apportionment correlation, specificity 0), matching the reproduced tool-to-tool comparison

## Limitations

- k-mer count tables are only useful for decOM in this benchmark; the same reads must additionally be processed into Kraken2 taxonomic profiles for FEAST and metaSourceTracker
- The parameters for kmtricks are dictated by decOM's documentation, so the tables may not be reusable for other k-mer-based analyses without regeneration
- Benchmark comparison rests on the mesocosm ground truth cell fractions; decOM's poor specificity (0) in this setting may not generalize to other datasets

## Evidence

- [other] kmtricks tables per decOM docs: [evidence span withheld — anchor status: not_found]
- [other] Taxonomic profiles needed for the other alternatives: [evidence span withheld — anchor status: not_found]
- [results] Comparative accuracy motivating the decOM input preparation: [evidence span withheld — anchor status: not_found]
