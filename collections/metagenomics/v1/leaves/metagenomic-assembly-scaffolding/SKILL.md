---
name: metagenomic-assembly-scaffolding
description: Use when raw sequencing reads (FASTQ) must be turned into a filtered
  scaffold/contig FASTA before kmer-signature-based classification (e.g. PlasFlow
  plasmid prediction) on metagenomic or small-genome assemblies.
when_to_use_negative:
- Input sequences are already filtered contigs longer than 1000 bp — re-assembling
  would be redundant
- The goal requires plasmid recovery from the de Bruijn assembly graph with paired
  reads (PlasmidSpades/Recycler approach), not kmer-signature classification of assembled
  scaffolds
- 'Short contigs (<1 kb) must be classified: kmer-signature methods are explicitly
  unreliable for short sequences'
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3697
tools:
- name: SPAdes
  role: Assembler producing the scaffolds from FASTQ reads prior to length filtering
    and plasmid classification
- name: PlasFlow
  role: Downstream consumer of the filtered assembly; classifies contigs as plasmid/chromosome
    via kmer genome signatures and a neural network
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script (bundled with PlasFlow, requires Bioperl and Getopt) that filters
    the assembly FASTA by a sequence length threshold
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:af0e579ae8b7363b5570bcac18e6e499e12cfcf727d149730f028c0a1be3eacc
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
  - PlasFlow
  - SPAdes
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metagenomic-assembly-scaffolding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles metagenomic or isolate sequencing reads into contigs/scaffolds with SPAdes and filters the assembly by sequence length (retaining only sequences longer than 1000 bp) to prepare a kmer-appropriate input for downstream plasmid prediction with PlasFlow.

## When to use

Use when raw sequencing reads (FASTQ) must be turned into a filtered scaffold/contig FASTA before kmer-signature-based classification (e.g. PlasFlow plasmid prediction) on metagenomic or small-genome assemblies.

## When NOT to use

- Input sequences are already filtered contigs longer than 1000 bp — re-assembling would be redundant
- The goal requires plasmid recovery from the de Bruijn assembly graph with paired reads (PlasmidSpades/Recycler approach), not kmer-signature classification of assembled scaffolds
- Short contigs (<1 kb) must be classified: kmer-signature methods are explicitly unreliable for short sequences

## Inputs

- Sequencing reads in FASTQ format (paired-end)
- Unfiltered SPAdes scaffold/contig FASTA (pre-filtering, e.g. 240 scaffolds in the Aeromonas sp. O23A workflow)

## Outputs

- Length-filtered FASTA of scaffolds/contigs longer than 1000 bp (36 sequences in the study)
- Filtered FASTA suitable as --input for PlasFlow.py classification

## How to apply

Assemble the FASTQ reads into scaffolds using the SPAdes assembler (the article's workflow used a SPAdes 3.9.1 assembly; the Aeromonas sp. O23A replication task yielded 240 scaffolds before filtering). Then run the PlasFlow-provided Perl script filter_sequences_by_length.pl with a length threshold of 1000 bp, leaving only sequences longer than 1000 bp — in the study this reduced 240 scaffolds to 36 sequences longer than 1 kb. The length filter is required because PlasFlow and other kmer-based methods do not perform well on short sequences where proper kmer coverage cannot be obtained; filtering also speeds up prediction and prevents excessive RAM usage. The filtered FASTA is the input to PlasFlow.py classification with --threshold 0.7, producing plasmid/chromosome/unclassified bins and a prediction TSV.

## Related tools

- **SPAdes** (Assembler producing the scaffolds from FASTQ reads prior to length filtering and plasmid classification)
- **PlasFlow** (Downstream consumer of the filtered assembly; classifies contigs as plasmid/chromosome via kmer genome signatures and a neural network) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl script (bundled with PlasFlow, requires Bioperl and Getopt) that filters the assembly FASTA by a sequence length threshold) — https://github.com/smaegol/PlasFlow

## Examples

```
filter_sequences_by_length.pl -input Aeromonas_O23A_scaffolds.fasta -output filtered_scaffolds.fasta -thresh 1000
```

## Evaluation signals

- Filtering output contains zero sequences shorter than or equal to the threshold (1000 bp in the study)
- Filtered assembly reproduces the expected sequence count — 36 sequences retained from 240 SPAdes scaffolds in the Aeromonas sp. O23A replication
- Filtered FASTA is parseable by PlasFlow.py and yields a prediction TSV with all contigs assigned plasmid/chromosome/unclassified at threshold 0.7
- Downstream PlasFlow bins reproduce the reported outcome: eight predicted plasmids, five matching the four known curated plasmids as in Supplemental Table S7

## Limitations

- PlasFlow (and kmer-based methods generally) does not perform well on short sequences because proper kmer coverage cannot be obtained, making short-contig results unreliable
- Additional length filtering is needed to prevent too high RAM usage when metagenomic assemblies contain large numbers of short contigs
- Uncertain predictions below the probability threshold are left unclassified rather than resolved

## Evidence
