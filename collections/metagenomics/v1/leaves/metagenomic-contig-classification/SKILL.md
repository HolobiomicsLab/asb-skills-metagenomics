---
name: metagenomic-contig-classification
description: Use when a metagenomic or plasmidome assembly (FASTA contigs/scaffolds)
  needs plasmid-vs-chromosome assignment with initial phylum-level taxonomy, and kmer-signature
  classification without prior knowledge of sample composition is appropriate.
when_to_use_negative:
- Input is raw reads rather than assembled contigs — PlasFlow operates on assembly
  FASTA, and results for short sequences (<1000 bp) are unreliable
- Sequences shorter than 1000 bp are the analysis target, since kmer-based signature
  methods do not perform well on short contigs
- The goal is recovering circular plasmids from a de Bruijn assembly graph with paired
  reads (use PlasmidSpades or Recycler instead)
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: PlasFlow
  role: 'Main classifier: computes kmer genome signatures, runs neural network ensemble,
    applies 0.7 probability threshold, outputs TSV predictions and FASTA bins'
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl (BioPerl) preprocessing script that removes contigs below a 1000 bp length
    threshold before classification
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: SOM-based comparator tool run on the same filtered assemblies to benchmark
    plasmid/chromosome calls against PlasFlow
- name: SPAdes
  role: Assembler used to produce the reference plasmidome and sample assemblies (contig
    FASTA inputs to classification)
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:92c4f27e003c18ecfd2cdcb95e7ed2e5338f4c51d1e3a8d51689952505cba429
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
  - TensorFlow
  - cBar
  - filter_sequences_by_length.pl
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# metagenomic-contig-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies assembled metagenomic contigs as plasmid- or chromosome-derived using kmer-frequency genome signatures and an ensemble of TensorFlow neural network models in PlasFlow, with probability thresholding to exclude uncertain calls.

## When to use

Use when a metagenomic or plasmidome assembly (FASTA contigs/scaffolds) needs plasmid-vs-chromosome assignment with initial phylum-level taxonomy, and kmer-signature classification without prior knowledge of sample composition is appropriate.

## When NOT to use

- Input is raw reads rather than assembled contigs — PlasFlow operates on assembly FASTA, and results for short sequences (<1000 bp) are unreliable
- Sequences shorter than 1000 bp are the analysis target, since kmer-based signature methods do not perform well on short contigs
- The goal is recovering circular plasmids from a de Bruijn assembly graph with paired reads (use PlasmidSpades or Recycler instead)

## Inputs

- Metagenomic or plasmidome assembly FASTA (contigs/scaffolds, e.g. SPAdes output)
- Length-filtered FASTA retaining only contigs > 1000 bp

## Outputs

- Tabular predictions file (e.g. test.plasflow_predictions.tsv) with contig_id, label, and per-class probability columns
- FASTA bins of input sequences classified as plasmids, chromosomes, and unclassified

## How to apply

First filter the assembly with filter_sequences_by_length.pl, keeping only contigs longer than 1000 bp, because kmer-based predictors are unreliable on short sequences. Run PlasFlow.py on the filtered FASTA with --threshold 0.7 (the default probability cutoff); the tool computes kmer-frequency signatures, transforms them with TF-IDF and L2 normalization (scikit-learn), and averages probabilities over the three best neural network models to assign each contig to one of 26 chromosome.*/plasmid.* phylum classes. Contigs whose maximum average probability falls below the threshold are labelled 'unclassified.unclassified'. Collect the tabular predictions (contig_id, label, per-class probabilities) and the three binned FASTA files (plasmids, chromosomes, unclassified). Validate against the shipped test reference (Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv) and, for plasmidome datasets, expect PlasFlow plasmid:chromosome ratios such as 15.73 (Brown Kav), 13.55 (Szczepanowski), 4.72 (Sentchilo), 2.83 (Zhang), versus cBar ratios of ~1.81, 1.72, 0.69, 1.40.

## Related tools

- **PlasFlow** (Main classifier: computes kmer genome signatures, runs neural network ensemble, applies 0.7 probability threshold, outputs TSV predictions and FASTA bins) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl (BioPerl) preprocessing script that removes contigs below a 1000 bp length threshold before classification) — https://github.com/smaegol/PlasFlow
- **cBar** (SOM-based comparator tool run on the same filtered assemblies to benchmark plasmid/chromosome calls against PlasFlow)
- **SPAdes** (Assembler used to produce the reference plasmidome and sample assemblies (contig FASTA inputs to classification))

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000; PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- plasflow_predictions.tsv exists in tabular format with contig_id, label, and per-class probability columns matching the test reference Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv with the 0.7 threshold applied
- Contigs with maximum average probability below 0.7 are labelled 'unclassified.unclassified'
- Three output FASTA files (plasmids, chromosomes, unclassified) contain only input contigs, partitioning them without loss
- On the four reference plasmidome assemblies, PlasFlow plasmid:chromosome ratios reproduce ~15.73, 4.72, 13.55, 2.83 versus cBar ~1.81, 1.72, 0.69, 1.40, with ~44% shared predictions of which ~85% are plasmid
- On KOW/ZS samples, plasmid-origin fractions fall in reported ranges of 25.02–27.05% of sequences and 14.94–20.66% of cumulative scaffold length

## Limitations

- Kmer-based prediction is unreliable on short sequences; contigs must be longer than 1000 bp before classification
- Models reflect the training data and can only be overcome by regularly updating the model with newly published genomic data
- PlasFlow is not maintained (version 1.1, Python 3.5, TensorFlow 0.10.0 required); large datasets may need reduced --batch_size to avoid memory issues

## Evidence

- [methods] PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7: [evidence span withheld — anchor status: not_found]
