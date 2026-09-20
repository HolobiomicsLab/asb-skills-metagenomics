---
name: sequence-read-quality-trimming
description: Use when raw FASTQ reads (e.g., Illumina runs from SRA accessions) are
  about to be classified with a k-mer/LCA classifier such as Kraken and low-quality
  bases or adapter contamination could distort exact k-mer matching.
when_to_use_negative:
- Input reads are already trimmed (e.g., SRA-supplied reads with adapters and low-quality
  bases removed)
- The classifier pipeline requires variable-length or paired-end structure that 'NNNNN'
  mate-joining would destroy
- Sequences contain many ambiguous nucleotides you need retained — Kraken ignores
  k-mers spanning ambiguous bases, so excessive Ns reduce classifiable content rather
  than being recoverable by trimming
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: Kraken
  role: Downstream classifier whose exact k-mer/LCA matching motivates the trimming
    step; classifies trimmed reads against a Jellyfish-built database
  repo: github.com/DerrickWood/kraken
- name: Jellyfish
  role: Multithreaded k-mer counter used to build the Kraken database over which the
    trimmed reads are classified
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/gb-2014-15-3-r46
    title: 'Kraken: ultrafast metagenomic sequence classification using exact alignments'
  source_content_hash: sha256:01c46310fff82ab48104c3a5764e41c28c59b5cf6d3b725f181af703fcac8bbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jellyfish
  - Kraken
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sequence-read-quality-trimming

## Summary

Preprocess raw FASTQ sequencing reads by trimming low-quality bases and adapter sequences before taxonomic classification, as done in the Kraken paper's Human Microbiome Project saliva analysis (SRS019120, SRS014468, SRS015055).

## When to use

Use when raw FASTQ reads (e.g., Illumina runs from SRA accessions) are about to be classified with a k-mer/LCA classifier such as Kraken and low-quality bases or adapter contamination could distort exact k-mer matching.

## When NOT to use

- Input reads are already trimmed (e.g., SRA-supplied reads with adapters and low-quality bases removed)
- The classifier pipeline requires variable-length or paired-end structure that 'NNNNN' mate-joining would destroy
- Sequences contain many ambiguous nucleotides you need retained — Kraken ignores k-mers spanning ambiguous bases, so excessive Ns reduce classifiable content rather than being recoverable by trimming

## Inputs

- Raw paired-end FASTQ files from the NCBI Sequence Read Archive (e.g., SRS019120, SRS014468, SRS015055)
- Known adapter sequence references for trimming

## Outputs

- Trimmed FASTQ files with low-quality bases and adapter sequences removed
- Mate-joined FASTA/FASTQ with mates concatenated via 'NNNNN' spacers, ready for Kraken classification

## How to apply

Download the FASTQ files for the accessions of interest from the NCBI Sequence Read Archive, then trim all sequences to remove low-quality bases and adapter sequences prior to classification, exactly as the Kraken authors did for the HMP saliva accessions SRS019120, SRS014468 and SRS015055. The rationale is that Kraken classifies via exact k-mer matches and LCA values: Kraken ignores k-mers with ambiguous nucleotides, and low-quality tails/adapters introduce spurious k-mers that can mislead the classification tree. After trimming, concatenate paired mates with 'NNNNN' between them so each pair is classified as a single unit; the spanning k-mers across the N gap are harmlessly ignored. The trimmed reads then feed into Kraken with a Jellyfish-built database, and results are reported per accession (fraction unclassified, genus-level distribution).

## Related tools

- **Kraken** (Downstream classifier whose exact k-mer/LCA matching motivates the trimming step; classifies trimmed reads against a Jellyfish-built database) — github.com/DerrickWood/kraken
- **Jellyfish** (Multithreaded k-mer counter used to build the Kraken database over which the trimmed reads are classified)

## Examples

```
kraken --db $KRAKEN_DB --fastq-input trimmed_reads.fastq > hmp_saliva_kraken.out
```

## Evaluation signals

- Trimmed read lengths should be non-increasing relative to raw reads, with no residual adapter sequence at read termini
- Fraction of ambiguous 'N' bases should not increase after trimming and mate-joining with 'NNNNN' spacers
- Kraken report per accession should show roughly 68.2% of reads unclassified, and genus-level distribution of classified reads near Streptococcus 30%, Haemophilus 17%, Prevotella 13% (values within a few percentage points are consistent)
- Per-accession reports should be combinable into a single summary table (hmp_saliva_kraken_report.tsv) with matching read totals to the input FASTQ

## Limitations

- Kraken ignores k-mers with ambiguous nucleotides, so 'NNNNN' joins between mates mean spanning k-mers contribute no evidence
- The standard Kraken database was built over complete bacterial, viral, and human genomes; reads outside this reference space remain unclassified (~68.2% here)
- The Kraken paper does not name a specific trimming tool or specify exact trimming thresholds — quality/adapters trimming parameters must be chosen by the practitioner

## Evidence

- [discussion] All sequences were trimmed to remove low quality bases and adapter sequences: "All sequences were trimmed to remove low quality bases and adapter sequences"
- [discussion] Kraken ignores k-mers with ambiguous nucleotides: [evidence span withheld — anchor status: not_found]
- [discussion] Concatenating paired mates with 'NNNNN' so each pair is classified as a single unit: [evidence span withheld — anchor status: not_found]
- [abstract] Kraken is an ultrafast and highly accurate program for assigning taxonomic labels: "Kraken is an ultrafast and highly accurate program for assigning taxonomic labels to metagenomic DNA sequences."
- [results] SRA accessions used in the HMP saliva analysis: "We retrieved the sequences of three accessions (SRS019120, SRS014468 and SRS015055) from the NCBI Sequence Read Archive"
