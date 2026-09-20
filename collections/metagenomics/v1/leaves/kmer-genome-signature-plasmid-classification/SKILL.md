---
name: kmer-genome-signature-plasmid-classification
description: Use when you have a metagenomic or isolate SPAdes assembly (contigs/scaffolds
  >1 kb) and need to bin sequences as plasmid vs chromosome, e.g. reproducing plasmid
  identification in a genome with known plasmids of 4-60 kb.
when_to_use_negative:
- Input sequences are shorter than 1000 bp — kmer-based results for short sequences
  are unreliable and should be filtered out first
- You need plasmid sequences recovered directly from the assembly graph with paired
  reads — use PlasmidSpades or Recycler instead, since PlasFlow works on assembled
  contigs
- You require plasmid replicon typing rather than sequence binning — PlasmidFinder
  is aimed at identifying plasmid replicons in bacterial genome assemblies
edam_operation: http://edamontology.org/operation_3461
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PlasFlow
  role: Neural-network kmer-signature classifier producing plasmid/chromosome/unclassified
    bins and a predictions TSV at a 0.7 probability threshold
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl script (Bioperl) that filters the assembly, retaining only sequences
    longer than the specified threshold (1000 bp)
  repo: https://github.com/smaegol/PlasFlow
- name: SPAdes
  role: Assembles sequencing reads into scaffolds that serve as the classification
    input
- name: PlasFlow_models
  role: Repository of trained neural network models used for prediction
  repo: https://github.com/smaegol/PlasFlow_models
- name: BLAST
  role: Confirms predicted plasmid sequences by similarity search against known manually
    curated plasmids
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:a53eeac270695140db23482e93f433db674562171b81e94609dcd381c9886e3d
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
  - PlasFlow_models
  - SPAdes
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_3461
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmer-genome-signature-plasmid-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify assembled bacterial contigs as plasmid or chromosome using PlasFlow's neural network trained on tetra/penta/hexa/heptamer kmer genome signatures from 9,565 RefSeq chromosomes and plasmids, reaching up to 96% accuracy without prior knowledge of taxonomic composition.

## When to use

Use when you have a metagenomic or isolate SPAdes assembly (contigs/scaffolds >1 kb) and need to bin sequences as plasmid vs chromosome, e.g. reproducing plasmid identification in a genome with known plasmids of 4-60 kb.

## When NOT to use

- Input sequences are shorter than 1000 bp — kmer-based results for short sequences are unreliable and should be filtered out first
- You need plasmid sequences recovered directly from the assembly graph with paired reads — use PlasmidSpades or Recycler instead, since PlasFlow works on assembled contigs
- You require plasmid replicon typing rather than sequence binning — PlasmidFinder is aimed at identifying plasmid replicons in bacterial genome assemblies

## Inputs

- SPAdes assembly scaffolds/contigs in FASTA format (e.g. Aeromonas sp. O23A assembly, 240 scaffolds)
- Length-filtered FASTA of sequences >1000 bp (e.g. 36 retained contigs)
- Trained PlasFlow neural network model (from github.com/smaegol/PlasFlow_models)

## Outputs

- PlasFlow predictions TSV with contig classifications and assignment probabilities (e.g. test.plasflow_predictions.tsv)
- FASTA files binning input sequences into plasmids, chromosomes and unclassified (e.g. eight plasmid sequences from 36 contigs)
- BLAST confirmation hits against known manually curated plasmids

## How to apply

First assemble reads into scaffolds with SPAdes, then filter sequences by length with filter_sequences_by_length.pl keeping only those longer than 1000 bp — PlasFlow does not perform well on short sequences because proper kmer coverage cannot be obtained, and filtering also prevents excessive RAM use. Run PlasFlow.py on the filtered FASTA with --input, --output and --threshold 0.7 (probability filtering that excludes uncertain predictions and lowers false hits). PlasFlow classifies each contig by kmer genome signatures using the trained deep neural network and produces a predictions TSV plus FASTA files binning sequences into plasmids, chromosomes and unclassified. Validate by checking that below-threshold sequences are labeled unclassified and by BLASTing predicted plasmid sequences against any known curated plasmids to confirm similarity.

## Related tools

- **PlasFlow** (Neural-network kmer-signature classifier producing plasmid/chromosome/unclassified bins and a predictions TSV at a 0.7 probability threshold) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl script (Bioperl) that filters the assembly, retaining only sequences longer than the specified threshold (1000 bp)) — https://github.com/smaegol/PlasFlow
- **SPAdes** (Assembles sequencing reads into scaffolds that serve as the classification input)
- **PlasFlow_models** (Repository of trained neural network models used for prediction) — https://github.com/smaegol/PlasFlow_models
- **BLAST** (Confirms predicted plasmid sequences by similarity search against known manually curated plasmids)

## Examples

```
PlasFlow.py --input filtered_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Predictions TSV contains classifications for every input contig with assignment probabilities; sequences below the 0.7 threshold are labeled unclassified
- Number of plasmid-binned FASTA entries matches expectation from similar data (e.g. eight plasmid predictions from 36 filtered Aeromonas sp. O23A contigs)
- BLAST of predicted plasmids against known curated plasmids confirms similarity for the expected subset (five of eight in the reported result, per Supplemental Table S7)
- All sequences in the input FASTA exceed the 1000 bp length threshold applied by filter_sequences_by_length.pl
- Overall classification accuracy on test datasets with known labels is in the reported range, up to 96%

## Limitations

- Does not perform well on short sequences (<1000 bp) because proper kmer coverage cannot be obtained from them, making results unreliable
- Models must be regularly updated with newly published genomic data to overcome training-set bias
- The PlasFlow project is no longer maintained (version 1.1, 2018); the author recommends newer packages instead
- Probability filtering at the threshold excludes uncertain predictions, so some true plasmids may be left unclassified (three of eight Aeromonas predictions did not match the four known plasmids)

## Evidence
