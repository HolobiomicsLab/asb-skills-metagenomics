---
name: blast-based-sequence-similarity-verification
description: Use when a kmer-signature classifier (e.g. PlasFlow) has binned assembly
  contigs as plasmids and you need sequence-level confirmation against known, manually
  curated plasmid references.
when_to_use_negative:
- Input contigs have not been classified yet — run PlasFlow.py (--threshold 0.7) on
  length-filtered (>1000 bp) assembly first; BLAST verification is a downstream confirmation
  step, not a classifier.
- No curated plasmid reference sequences exist for the organism/sample, since database-similarity
  verification depends on matching known references (a limitation of BLAST-based approaches
  such as PLACNET).
- Contigs are shorter than 1000 bp — PlasFlow explicitly recommends filtering these
  out because kmer-based results for short sequences are unreliable.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Sequence similarity search used to confirm that predicted plasmid sequences
    match the known manually curated plasmids
- name: PlasFlow
  role: Upstream tool producing the plasmid-binned FASTA and predictions TSV that
    BLAST verification validates
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script that pre-filters the assembly, retaining only sequences longer
    than 1000 bp before classification and verification
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:a897d096a2b64f3392ab7cc13ab774043dc9ad500f717ba49f3b01e51bed85d1
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
  - BLAST
  - PlasFlow
  - SPAdes
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# blast-based-sequence-similarity-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verify computationally predicted plasmid sequences (e.g. the plasmid bin FASTA produced by PlasFlow at threshold 0.7) by BLAST-searching them against a set of manually curated known plasmids to confirm which predictions are sequence-similar to validated references.

## When to use

Use when a kmer-signature classifier (e.g. PlasFlow) has binned assembly contigs as plasmids and you need sequence-level confirmation against known, manually curated plasmid references.

## When NOT to use

- Input contigs have not been classified yet — run PlasFlow.py (--threshold 0.7) on length-filtered (>1000 bp) assembly first; BLAST verification is a downstream confirmation step, not a classifier.
- No curated plasmid reference sequences exist for the organism/sample, since database-similarity verification depends on matching known references (a limitation of BLAST-based approaches such as PLACNET).
- Contigs are shorter than 1000 bp — PlasFlow explicitly recommends filtering these out because kmer-based results for short sequences are unreliable.

## Inputs

- Plasmid bin FASTA (sequences classified as plasmids by PlasFlow at threshold 0.7)
- FASTA of known manually curated plasmid reference sequences (e.g. four Aeromonas sp. O23A plasmids, 4-60 kb)
- PlasFlow predictions TSV with contig classifications and assignment probabilities

## Outputs

- BLAST hit table mapping predicted plasmid contigs to known plasmid references
- Count of predictions confirmed by similarity (e.g. five of eight matching known plasmids, per Supplemental Table S7)

## How to apply

After running PlasFlow.py with --threshold 0.7 on the filtered (>1 kb) assembly, collect the sequences classified as plasmids into the plasmids bin FASTA. Run BLAST of these predicted plasmid sequences against the known manually curated plasmid sequences for the organism (e.g. the four known 4-60 kb Aeromonas sp. O23A plasmids) to confirm similarity. In the reference workflow, five of eight PlasFlow predictions matched the four known plasmids, reproducing Supplemental Table S7. BLAST here serves as independent validation of neural-network kmer-based classification, complementing PlasFlow's probability filtering (uncertain predictions fall below threshold and are unclassified).

## Related tools

- **BLAST** (Sequence similarity search used to confirm that predicted plasmid sequences match the known manually curated plasmids)
- **PlasFlow** (Upstream tool producing the plasmid-binned FASTA and predictions TSV that BLAST verification validates) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl script that pre-filters the assembly, retaining only sequences longer than 1000 bp before classification and verification) — https://github.com/smaegol/PlasFlow

## Evaluation signals

- The PlasFlow predictions TSV contains contig classifications with assignment probabilities filtered at the 0.7 threshold; below-threshold sequences are unclassified, not called plasmids.
- The number of BLAST-confirmed predictions is comparable to the reported outcome: eight predicted plasmid sequences with five matching the four known plasmids (Supplemental Table S7).
- The plasmid bin FASTA contains only sequences that PlasFlow labeled plasmid, and its content matches the plasmid rows of the predictions TSV.
- Input to the whole pipeline was length-filtered (filter_sequences_by_length.pl, threshold 1000 bp), e.g. 36 sequences retained from 240 SPAdes scaffolds.
- Each BLAST hit aligns a predicted contig to one of the known 4-60 kb plasmid references with biologically meaningful coverage/identity.

## Limitations

- BLAST similarity verification can only confirm predictions that resemble known plasmids; novel plasmids without curated references will lack matches (the approach depends on reference databases, as noted for BLAST-based methods like PLACNET).
- PlasFlow's kmer-based classification is unreliable for short sequences, so contigs shorter than 1000 bp must be filtered out before verification.
- Probability thresholding at 0.7 excludes uncertain predictions, so some true plasmids may be left unclassified rather than binned as plasmids.

## Evidence

- [intro] which uses BLAST to compare sequences against reference databases and then a network analysis to reconstruct plasmids (26).: "which uses BLAST to compare sequences against reference databases and then a network analysis to reconstruct plasmids (26)."
- [methods] PlasFlow produces fasta files containing input sequences binned to plasmids, chromosomes and unclassified.: "PlasFlow produces fasta files containing input sequences binned to plasmids, chromosomes and unclassified."
- [methods] filter sequences by length, leaving only those longer than 1000 bp: "filter sequences by length, leaving only those longer than 1000 bp"
- [readme] It relies on the neural network models trained on full genome and plasmid sequences and is able to differentiate between plasmids and chromosomes with accuracy reaching 96%.: "It relies on the neural network models trained on full genome and plasmid sequences and is able to differentiate between plasmids and chromosomes with accuracy reaching 96%."
- [discussion] We also introduced probability filtering that excluded uncertain predictions; thus, the number of false hits was significantly lower: "We also introduced probability filtering that excluded uncertain predictions; thus, the number of false hits was significantly lower"
