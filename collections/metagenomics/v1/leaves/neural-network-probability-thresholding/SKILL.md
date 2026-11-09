---
name: neural-network-probability-thresholding
description: Use when you need to classify contigs/scaffolds from a metagenomic or
  bacterial genome assembly as plasmid vs. chromosome, and want to discard low-confidence
  neural network predictions via probability thresholding.
when_to_use_negative:
- Input sequences are shorter than 1000 bp — PlasFlow kmer-based classification is
  unreliable on short sequences and such contigs should be filtered out first
- The assembly contains only chromosomal replicons with no plasmid content expected
  and no need for confidence filtering
- You need a plasmid replicon/typing result rather than sequence-level classification
  — PlasmidFinder targets replicons and is a different approach
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3474
tools:
- name: PlasFlow
  role: Neural-network classifier that scores each contig's plasmid/chromosome assignment
    probability; thresholding excludes uncertain predictions
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script that filters the input assembly by length (>1000 bp) before
    classification
  repo: https://github.com/smaegol/PlasFlow
- name: BLAST
  role: Used to confirm predicted plasmid sequences against known manually curated
    plasmids
- name: SPAdes
  role: Assembles sequencing reads into scaffolds that form the input contigs for
    classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:9e2aa0a2417de6c7126c47629d48face5fced8a5a8cdc86c7616a41c2f3fd778
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# neural-network-probability-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies PlasFlow's deep neural network classifier to assembled contigs and filters its per-sequence assignment probabilities at a specified threshold (default 0.7) so that uncertain predictions are excluded and sequences are binned into plasmids, chromosomes, or unclassified.

## When to use

Use when you need to classify contigs/scaffolds from a metagenomic or bacterial genome assembly as plasmid vs. chromosome, and want to discard low-confidence neural network predictions via probability thresholding.

## When NOT to use

- Input sequences are shorter than 1000 bp — PlasFlow kmer-based classification is unreliable on short sequences and such contigs should be filtered out first
- The assembly contains only chromosomal replicons with no plasmid content expected and no need for confidence filtering
- You need a plasmid replicon/typing result rather than sequence-level classification — PlasmidFinder targets replicons and is a different approach

## Inputs

- Filtered FASTA of assembly contigs/scaffolds >1000 bp (e.g., SPAdes scaffolds filtered by filter_sequences_by_length.pl)
- PlasFlow trained neural network model files (github.com/smaegol/PlasFlow_models)
- Probability threshold parameter (default 0.7)

## Outputs

- Tabular predictions file (e.g., test.plasflow_predictions.tsv) with per-contig class assignments and assignment probabilities
- FASTA files binning input sequences into plasmids, chromosomes, and unclassified
- Set of high-confidence plasmid sequences (e.g., 8 plasmid sequences identified in Aeromonas sp. O23A)

## How to apply

First filter the assembly FASTA with filter_sequences_by_length.pl to retain only sequences longer than 1000 bp, since kmer-based methods like PlasFlow do not perform well on short sequences and filtering also reduces RAM usage. Run PlasFlow.py with --input (filtered FASTA), --output (predictions TSV) and --threshold 0.7; the neural network, trained on kmer genome signatures of 9,565 bacterial chromosomes and plasmids, assigns each contig a class and probability, and sequences below the threshold are reported as unclassified. The rationale for thresholding is that excluding low-probability classifications significantly lowers false hits and increases overall accuracy (reported accuracy up to 96%). Collect the plasmid-binned sequences from the PlasFlow plasmids FASTA output, and optionally confirm predictions (e.g., by BLAST against known plasmids) as done for the Aeromonas sp. O23A assembly where 8 predicted plasmids included 5 matching the four known plasmids.

## Related tools

- **PlasFlow** (Neural-network classifier that scores each contig's plasmid/chromosome assignment probability; thresholding excludes uncertain predictions) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl/Bioperl script that filters the input assembly by length (>1000 bp) before classification) — https://github.com/smaegol/PlasFlow
- **BLAST** (Used to confirm predicted plasmid sequences against known manually curated plasmids)
- **SPAdes** (Assembles sequencing reads into scaffolds that form the input contigs for classification)

## Examples

```
PlasFlow.py --input filtered_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Predictions TSV contains a class and assignment probability for each input contig, with contigs below the 0.7 threshold reported as unclassified
- No input contig longer than the 1000 bp filter appears without a classification entry in the TSV
- Predicted plasmid sequences BLAST against known plasmids: in the reference study 5 of 8 predictions matched the four known Aeromonas sp. O23A plasmids
- Reported counts consistent with the study outcome: 8 plasmid-binned sequences from 36 filtered assembly sequences
- Overall classification accuracy should be in the range reported by the authors (up to 96% on test data) when the threshold is applied

## Limitations

- PlasFlow does not perform well on short sequences (<1000 bp) because proper kmer coverage is hard to achieve; results for short sequences are unreliable
- The model depends on training data coverage and can only be improved by regularly updating the model with newly published genomic data
- Thresholding trades recall for precision: below-threshold sequences are unclassified, so genuinely plasmid-derived contigs with low probability will be missed
- PlasFlow 1.1 is explicitly NOT MAINTAINED and requires a pinned old stack (Python 3.5, TensorFlow 0.10.0rc0)

## Evidence
