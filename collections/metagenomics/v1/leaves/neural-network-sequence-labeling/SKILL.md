---
name: neural-network-sequence-labeling
description: Use when you need to label contigs from a metagenomic or bacterial genome
  assembly as plasmid vs. chromosome without prior knowledge of sample taxonomic or
  functional composition, and want uncertain predictions flagged.
when_to_use_negative:
- Input contains many short contigs (< 1000 bp) — kmer coverage is insufficient and
  results are unreliable; filter or exclude them first
- The task requires recovering circular plasmids from an assembly graph with paired
  reads — PlasmidSpades or Recycler are the appropriate tools, not this signature-based
  classifier
- You need plasmid replicon typing of known enterobacterial plasmids — PlasmidFinder's
  replicon-database approach is the targeted tool
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0085
tools:
- name: PlasFlow
  role: Runs the neural network ensemble over TF-IDF-normalized kmer genome signatures
    to assign plasmid/chromosome/taxonomic labels with 0.7 thresholding
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow_models
  role: Repository holding the trained neural network models (the three best models
    averaged by the ensemble)
  repo: https://github.com/smaegol/PlasFlow_models
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script that removes contigs at or below the length threshold
    (1000 bp) before classification
  repo: https://github.com/smaegol/PlasFlow
- name: scikit-learn
  role: TF-IDF transformation and L2 normalization of the kmer frequency matrix (v0.18.1)
- name: TensorFlow
  role: Executes the deep neural network classifiers (v0.10.0) to produce class probabilities
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:65a6a93a7a3160b32714c16ebcede8ee41f52308728b87e962609aaea7a57279
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
  - PlasFlow_models
  - TensorFlow
  - filter_sequences_by_length.pl
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# neural-network-sequence-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify assembled metagenomic contigs as plasmid or chromosome (with phylum-level taxonomy) using PlasFlow, which computes kmer-frequency genome signatures and feeds TF-IDF-transformed features into an ensemble of TensorFlow neural network models, applying a 0.7 probability threshold to exclude uncertain calls.

## When to use

Use when you need to label contigs from a metagenomic or bacterial genome assembly as plasmid vs. chromosome without prior knowledge of sample taxonomic or functional composition, and want uncertain predictions flagged.

## When NOT to use

- Input contains many short contigs (< 1000 bp) — kmer coverage is insufficient and results are unreliable; filter or exclude them first
- The task requires recovering circular plasmids from an assembly graph with paired reads — PlasmidSpades or Recycler are the appropriate tools, not this signature-based classifier
- You need plasmid replicon typing of known enterobacterial plasmids — PlasmidFinder's replicon-database approach is the targeted tool

## Inputs

- Metagenomic or bacterial genome assembly in FASTA format (e.g. SPAdes 3.9.1 scaffolds), filtered to contigs > 1000 bp

## Outputs

- Tab-separated predictions file (contig_id, contig_name, contig_length, id, label, per-class probabilities)
- FASTA files binning input sequences into plasmids, chromosomes, and unclassified

## How to apply

First filter the FASTA assembly with filter_sequences_by_length.pl keeping only sequences longer than 1000 bp, because kmer-based prediction is unreliable on short contigs. Compute kmer-frequency genome signatures for each filtered sequence, transform them with TF-IDF and L2 normalization via scikit-learn, then run the TensorFlow neural network ensemble (averaging probabilities over the three best models trained on 10 kb fragments from 9,565 bacterial chromosomes and plasmids, split by phylum). Invoke PlasFlow.py with --threshold 0.7: each contig is assigned the label with the highest averaged probability across the chromosome.* and plasmid.* classes, and contigs whose maximum average score falls below 0.7 are labeled 'unclassified.unclassified'. Results are written as a tab-separated predictions file (contig_id, contig_name, contig_length, id, label, per-class probabilities) plus FASTA bins for plasmids, chromosomes, and unclassified sequences.

## Related tools

- **PlasFlow** (Runs the neural network ensemble over TF-IDF-normalized kmer genome signatures to assign plasmid/chromosome/taxonomic labels with 0.7 thresholding) — https://github.com/smaegol/PlasFlow
- **PlasFlow_models** (Repository holding the trained neural network models (the three best models averaged by the ensemble)) — https://github.com/smaegol/PlasFlow_models
- **filter_sequences_by_length.pl** (Perl/Bioperl script that removes contigs at or below the length threshold (1000 bp) before classification) — https://github.com/smaegol/PlasFlow
- **scikit-learn** (TF-IDF transformation and L2 normalization of the kmer frequency matrix (v0.18.1))
- **TensorFlow** (Executes the deep neural network classifiers (v0.10.0) to produce class probabilities)

## Examples

```
filter_sequences_by_length.pl -input assembly.fasta -output filtered.fasta -thresh 1000 && PlasFlow.py --input filtered.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Predictions TSV label/probability columns match the reference test result shipped in the repo test folder (Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv) with the 0.7 threshold applied
- On the 42 plasmid-rich bacterial genomes from Arredondo-Alonso et al., reproduced plasmid recall reaches ~85.98% and precision ~72.17% (vs. cBar 76.82%/60.51% and Recycler 12.0%/30.00%)
- Every contig label is either a chromosome.* or plasmid.* class or 'unclassified.unclassified'; no free-form labels appear
- FASTA bins (plasmids, chromosomes, unclassified) partition the filtered input contigs with no sequences lost or duplicated
- Classification accuracy on held-out test fragments approaches the reported upper bound of 96%

## Limitations

- Does not perform well on short sequences (< 1000 bp) because proper kmer coverage cannot be obtained from them
- Model accuracy degrades on taxa not represented in the 9,565 training chromosomes/plasmids; the authors note it can only be overcome by regularly updating the model with newly published genomic data
- Probability filtering (default 0.7) intentionally discards uncertain predictions — sensitivity is traded for a low false-prediction rate (~5%)
- Project is explicitly NOT MAINTAINED; requires dated dependencies (Python 3.5, TensorFlow 0.10.0rc0, scikit-learn 0.18.1, R Biostrings)

## Evidence
