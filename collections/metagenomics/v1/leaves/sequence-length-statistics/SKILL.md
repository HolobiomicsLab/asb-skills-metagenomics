---
name: sequence-length-statistics
description: Use when you need to characterize a FASTA reference set (chromosomes,
  plasmids, contigs) by per-sequence length and %G+C content, then aggregate means
  by group labels and reproduce published dataset statistics.
when_to_use_negative:
- Input sequences are already short k-mer fragments rather than full-length sequences,
  since fragment statistics will not match full-genome reference values
- You need sequence classification itself (plasmid vs chromosome) — use PlasFlow's
  neural-network prediction, not descriptive length/%G+C statistics
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0080
tools:
- name: Biopython
  role: Parses FASTA records via SeqIO and computes %G+C content and sequence length
    for every sequence
- name: pandas
  role: Joins per-sequence statistics with molecule-type/phylum labels and aggregates
    group means via groupby
- name: PlasFlow
  role: 'Context tool: the statistics skill characterizes the RefSeq training dataset
    (1961 chromosomes, 7604 plasmids) used to build PlasFlow''s neural network models'
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:316a9cde9cbe8ea44221300307382ef141a8328351595dddc3f93a660ea640f1
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# sequence-length-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for computing per-sequence nucleotide statistics (sequence length and %G+C content) on FASTA datasets and aggregating them by category (e.g. molecule type, phylum) to characterize reference collections, as done for the 1961 RefSeq chromosomes and 7604 plasmids used to train PlasFlow.

## When to use

Use when you need to characterize a FASTA reference set (chromosomes, plasmids, contigs) by per-sequence length and %G+C content, then aggregate means by group labels and reproduce published dataset statistics.

## When NOT to use

- Input sequences are already short k-mer fragments rather than full-length sequences, since fragment statistics will not match full-genome reference values
- You need sequence classification itself (plasmid vs chromosome) — use PlasFlow's neural-network prediction, not descriptive length/%G+C statistics

## Inputs

- FASTA file of full-length sequences (e.g. RefSeq chromosomes and plasmids)
- Annotation/metadata table mapping sequence IDs to labels (e.g. Supplemental Table S1 with molecule type and phylum)

## Outputs

- Per-sequence statistics table (sequence ID, length, %G+C, labels)
- Aggregated summary table of mean %G+C and mean length grouped by molecule type and phylum (e.g. gc_content_length_summary.tsv)

## How to apply

Parse each FASTA record with Biopython's SeqIO and compute sequence length and %G+C with a Biopython-based nucleotide counter. Join the per-sequence statistics with category labels (e.g. molecule type and phylum from Supplemental Table S1 for the PlasFlow training data) using pandas. Aggregate with pandas groupby means, grouped separately by molecule type and by phylum, and write the result to a TSV such as gc_content_length_summary.tsv. Validate by checking the aggregated means against published reference values — for the PlasFlow RefSeq training set: 51.05% G+C and 3587 kb mean length for 1961 chromosomes, and 44.31% G+C and 85 kb mean length for 7604 plasmids — within a small tolerance.

## Related tools

- **Biopython** (Parses FASTA records via SeqIO and computes %G+C content and sequence length for every sequence)
- **pandas** (Joins per-sequence statistics with molecule-type/phylum labels and aggregates group means via groupby)
- **PlasFlow** (Context tool: the statistics skill characterizes the RefSeq training dataset (1961 chromosomes, 7604 plasmids) used to build PlasFlow's neural network models) — https://github.com/smaegol/PlasFlow

## Examples

```
from Bio import SeqIO; import pandas as pd; recs = list(SeqIO.parse('refseq_train.fasta','fasta')); stats = pd.DataFrame([{'id': r.id, 'len': len(r.seq), 'gc': 100*(r.seq.count('G')+r.seq.count('C'))/len(r.seq)} for r in recs]); stats.join(labels, on='id').groupby('molecule_type')[['gc','len']].mean().to_csv('gc_content_length_summary.tsv', sep='\t')
```

## Evaluation signals

- Computed chromosome mean %G+C and mean length reproduce 51.05% and 3587 kb (from 1961 sequences) within a small tolerance
- Computed plasmid mean %G+C and mean length reproduce 44.31% and 85 kb (from 7604 sequences) within a small tolerance
- Per-sequence table contains length and %G+C for every FASTA record (no missing or zero-length entries)
- Grouped summary contains one row per molecule type (and per phylum) with correct record counts: 1961 chromosomes and 7604 plasmids
- Output TSV (e.g. gc_content_length_summary.tsv) is well-formed with expected column headers and numeric means

## Limitations

- Reported means only reproduce if the same full-length RefSeq records are used; PlasFlow's own downstream workflow filters sequences by length (leaving only those longer than 1000 bp), which would shift length statistics away from full-genome reference values
- The reference values (51.05%/3587 kb chromosomes; 44.31%/85 kb plasmids) apply to the PlasFlow RefSeq training set, not to arbitrary sequence collections
- Training-set statistics depend on correct label joins with Supplemental Table S1; misassigned phylum/molecule-type labels will invalidate grouped means

## Evidence

- [supplementary] Chromosome sequences had a mean %G+C of 51.05% and mean length of 3587 kb, whereas plasmids had a mean %G+C of 44.31% and mean length of 85 kb.: [evidence span withheld — anchor status: not_found]
