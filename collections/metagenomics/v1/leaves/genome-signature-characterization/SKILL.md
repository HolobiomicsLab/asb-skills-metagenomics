---
name: genome-signature-characterization
description: Use when characterizing plasmid vs. chromosome sequence sets by %G+C
  content and length (e.g. reproducing PlasFlow's RefSeq training-set statistics)
  or checking whether a nucleotide FASTA dataset matches reported composition properties.
when_to_use_negative:
- Classifying metagenomic contigs as plasmid/chromosome — that is PlasFlow's prediction
  task (neural network on k-mer counts with a 0.7 probability threshold), not signature
  characterization
- Input is already a k-mer count table (e.g. kmern_split_raw_counts_tax_classes_numeric_annotated_filtered_tax.tsv)
  rather than nucleotide FASTA sequences
- Sequences shorter than ~1000 bp — k-mer/genome-signature approaches do not perform
  well on short contigs, so their composition statistics are not informative for this
  purpose
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0091
tools:
- name: PlasFlow
  role: Genome-signature tool whose RefSeq training dataset (chromosomes vs. plasmids)
    these statistics characterize and validate
  repo: https://github.com/smaegol/PlasFlow
- name: Biopython
  role: SeqIO parsing of RefSeq FASTA records and nucleotide counting to compute per-sequence
    %G+C and length
- name: pandas
  role: Joining per-sequence statistics with Supplemental Table S1 labels and groupby
    aggregation of means by molecule type and phylum
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:a2b555be61f5325b226609f3a5b616dd683a500338e219b8c957a71fbe706611
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genome-signature-characterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes nucleotide-composition genome signatures — %G+C content and sequence length — across FASTA sequence collections (e.g. 1961 RefSeq chromosomes and 7604 plasmids) and aggregates them by molecule type or phylum, providing the descriptive statistics that underpin PlasFlow's genome-signature approach and validate its training dataset.

## When to use

Use when characterizing plasmid vs. chromosome sequence sets by %G+C content and length (e.g. reproducing PlasFlow's RefSeq training-set statistics) or checking whether a nucleotide FASTA dataset matches reported composition properties.

## When NOT to use

- Classifying metagenomic contigs as plasmid/chromosome — that is PlasFlow's prediction task (neural network on k-mer counts with a 0.7 probability threshold), not signature characterization
- Input is already a k-mer count table (e.g. kmern_split_raw_counts_tax_classes_numeric_annotated_filtered_tax.tsv) rather than nucleotide FASTA sequences
- Sequences shorter than ~1000 bp — k-mer/genome-signature approaches do not perform well on short contigs, so their composition statistics are not informative for this purpose

## Inputs

- Full-length nucleotide FASTA sequences from NCBI RefSeq (1961 bacterial chromosomes and 7604 plasmids)
- Annotation table mapping sequences to molecule type (chromosome/plasmid) and phylum (Supplemental Table S1)

## Outputs

- Per-sequence statistics table (sequence ID, %G+C content, length in bp)
- Aggregated summary TSV (gc_content_length_summary.tsv) with mean %G+C and mean length grouped by molecule type and phylum

## How to apply

Parse the full-length FASTA sequences (1961 chromosomes and 7604 plasmids downloaded from NCBI RefSeq) with Biopython's SeqIO, then compute %G+C content and length for every record using a Biopython-based nucleotide counter. Join the per-sequence statistics with the molecule-type and phylum labels from Supplemental Table S1 using pandas. Aggregate mean %G+C and mean length grouped by molecule type (and by phylum) with pandas groupby means, and write the aggregated table to gc_content_length_summary.tsv. Validate the result against the reported values — 51.05% G+C / 3587 kb mean for chromosomes and 44.31% G+C / 85 kb mean for plasmids — within a small tolerance; discrepancies indicate parsing or counting errors rather than dataset differences. Remember that PlasFlow trains on 10 kb fragments of these genomes, so full-length statistics characterize the source data, not the actual network input fragments.

## Related tools

- **PlasFlow** (Genome-signature tool whose RefSeq training dataset (chromosomes vs. plasmids) these statistics characterize and validate) — https://github.com/smaegol/PlasFlow
- **Biopython** (SeqIO parsing of RefSeq FASTA records and nucleotide counting to compute per-sequence %G+C and length)
- **pandas** (Joining per-sequence statistics with Supplemental Table S1 labels and groupby aggregation of means by molecule type and phylum)

## Examples

```
from Bio import SeqIO; gc = [(r.id, len(r.seq), 100*(r.seq.upper().count("G")+r.seq.upper().count("C"))/len(r.seq)) for r in SeqIO.parse("refseq_chromosomes_plasmids.fasta", "fasta")]
```

## Evaluation signals

- Computed chromosome means reproduce 51.05% G+C and 3587 kb mean length within a small tolerance
- Computed plasmid means reproduce 44.31% G+C and 85 kb mean length within a small tolerance
- Sequence counts match the source dataset: 1961 chromosomes + 7604 plasmids = 9565 records
- All per-sequence %G+C values fall within 0–100% and lengths are positive integers in bp
- Aggregated TSV contains groups for both molecule types (chromosome, plasmid) and reports means per phylum without unlabelled/NaN groups

## Limitations

- k-mer-based (signature) methods, including PlasFlow, 'does not perform well on short sequences, as it is hard to get proper kmer coverage from them' — composition statistics are most meaningful on full-length or ≥1000 bp sequences
- These statistics describe the full-length RefSeq training genomes, whereas the neural network is actually trained on 10 kb fragments, so full-length means are not the exact network-input statistics
- Statistics are only as complete as the annotation table; Supplemental Table S1 must be available to group by molecule type/phylum, and supplementary tables referenced in the article are not provided inline

## Evidence

- [results] reported chromosome/plasmid composition values: [evidence span withheld — anchor status: not_found]
