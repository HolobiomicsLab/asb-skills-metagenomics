---
name: taxonomic-group-aggregation
description: Use when a FASTA-derived per-sequence statistics table needs to be summarized
  by taxonomic labels (phylum) or molecule type (chromosome vs plasmid), e.g. to reproduce
  reported dataset statistics before model training.
when_to_use_negative:
- Input is raw sequencing reads or metagenomic contigs without curated molecule-type/phylum
  annotations to group by
- The goal is classification of contigs (use PlasFlow prediction with a probability
  threshold, not summary aggregation)
- Sequences are short fragments (<1000 bp) where k-mer/G+C statistics are unreliable
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Biopython
  role: Parse FASTA with SeqIO and compute per-sequence %G+C content and length
  repo: https://github.com/smaegol/PlasFlow
- name: pandas
  role: Join per-sequence statistics with molecule-type/phylum labels and compute
    groupby means
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow
  role: Provides the RefSeq training dataset (1961 chromosomes, 7604 plasmids) and
    preprocessing scripts whose statistics this skill summarizes
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:e2fc427300455c325a4d06f021c5fca6328065a80f46242f809c4b271ec03f55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/smaegol/PlasFlow
  repo_url: https://github.com/smaegol/PlasFlow
  tools:
  - Biopython
  - PlasFlow
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# taxonomic-group-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-sequence sequence statistics (e.g. %G+C content, sequence length) by taxonomic or molecule-type groups using pandas groupby means, to summarize and validate training datasets such as the PlasFlow RefSeq chromosome/plasmid set.

## When to use

Use when a FASTA-derived per-sequence statistics table needs to be summarized by taxonomic labels (phylum) or molecule type (chromosome vs plasmid), e.g. to reproduce reported dataset statistics before model training.

## When NOT to use

- Input is raw sequencing reads or metagenomic contigs without curated molecule-type/phylum annotations to group by
- The goal is classification of contigs (use PlasFlow prediction with a probability threshold, not summary aggregation)
- Sequences are short fragments (<1000 bp) where k-mer/G+C statistics are unreliable

## Inputs

- RefSeq full-length chromosome and plasmid FASTA files (training dataset, e.g. 1961 chromosomes and 7604 plasmids)
- Annotation table mapping sequence accessions to molecule type and phylum (Supplemental Table S1)

## Outputs

- Aggregated TSV (e.g. gc_content_length_summary.tsv) with mean %G+C and mean length grouped by molecule type and phylum

## How to apply

Parse the FASTA sequences with Biopython's SeqIO and compute %G+C content and length for every sequence with a Biopython-based nucleotide counter. Join these per-sequence statistics with molecule-type and phylum labels from the annotation table (Supplemental Table S1) using pandas. Aggregate with pandas groupby means, grouping by molecule type and by phylum, and write the result to a TSV such as gc_content_length_summary.tsv. Validate by checking that aggregated means reproduce published values — for the PlasFlow training set: 51.05% G+C / 3587 kb mean length for the 1961 chromosome sequences and 44.31% G+C / 85 kb for the 7604 plasmid sequences — within a small tolerance.

## Related tools

- **Biopython** (Parse FASTA with SeqIO and compute per-sequence %G+C content and length) — https://github.com/smaegol/PlasFlow
- **pandas** (Join per-sequence statistics with molecule-type/phylum labels and compute groupby means) — https://github.com/smaegol/PlasFlow
- **PlasFlow** (Provides the RefSeq training dataset (1961 chromosomes, 7604 plasmids) and preprocessing scripts whose statistics this skill summarizes) — https://github.com/smaegol/PlasFlow

## Examples

```
import pandas as pd; stats = pd.read_csv('per_sequence_gc_len.tsv', sep='\t'); summary = stats.groupby(['molecule_type','phylum'])[['gc_content','length_kb']].mean(); summary.to_csv('gc_content_length_summary.tsv', sep='\t')
```

## Evaluation signals

- Computed chromosome means reproduce 51.05% G+C and 3587 kb mean length within a small tolerance
- Computed plasmid means reproduce 44.31% G+C and 85 kb mean length within a small tolerance
- Sequence counts match the expected dataset composition: 1961 chromosomes and 7604 plasmids
- Output TSV contains rows for both groupings (by molecule type and by phylum) with non-null mean values

## Limitations

- Requires a pre-existing, accurate annotation table (molecule type and phylum per accession); the paper notes the annotation step uses rentrez against NCBI and may be slow
- Summary statistics reflect full-length RefSeq sequences; applying the same aggregation to short fragments will not reproduce the reported values

## Evidence

- [results] Chromosome sequences had a mean %G+C of 51.05% and mean length of 3587 kb, whereas plasmids had a mean %G+C of 44.31% and mean length of 85 kb.: [evidence span withheld — anchor status: not_found]
- [readme] Run Rscript create_annotation_of_data.R to create the annotation table based on the RefSeq catalog (filtered to contain only downloaded sequences).: [evidence span withheld — anchor status: external_doc]
- [methods] Parse the RefSeq training FASTA sequences (1961 chromosomes, 7604 plasmids) with Biopython's SeqIO.: [evidence span withheld — anchor status: not_found]
