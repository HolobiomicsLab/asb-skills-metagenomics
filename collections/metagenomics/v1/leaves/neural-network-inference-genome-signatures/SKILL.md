---
name: neural-network-inference-genome-signatures
description: Use when you have metagenomic or genomic FASTA contigs longer than 1000
  bp that must be labeled plasmid vs chromosome by k-mer signature, and uncertain
  predictions should be excluded via a probability threshold.
when_to_use_negative:
- Input contigs are mostly shorter than 1000 bp, where k-mer-based methods perform
  poorly and results are unreliable
- The goal is recovering circular plasmid contigs from an assembly graph with paired
  reads (use PlasmidSpades or Recycler instead)
- The goal is identifying specific plasmid replicons in an isolate genome assembly
  (use PlasmidFinder instead)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3474
tools:
- name: PlasFlow
  role: 'Main classifier: computes k-mer frequency features and applies the trained
    neural network with probability thresholding and output binning'
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow_models
  role: Provides the pre-trained models tested and described in the manuscript
  repo: https://github.com/smaegol/PlasFlow_models
- name: TensorFlow
  role: Runtime for the deep artificial neural network classifier (version 0.10.0rc0
    required)
- name: scikit-learn
  role: Python dependency (0.18.1) used in the PlasFlow environment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:c2cd27a240bf3c6c33bab24e538ce93d35b7a1207de8e35cb46788a4754569f0
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
  - Python
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# neural-network-inference-genome-signatures

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies metagenomic contigs as plasmid or chromosome using a TensorFlow-trained deep neural network over k-mer genome signatures (PlasFlow), with probability-threshold filtering to trade unclassified fraction for accuracy.

## When to use

Use when you have metagenomic or genomic FASTA contigs longer than 1000 bp that must be labeled plasmid vs chromosome by k-mer signature, and uncertain predictions should be excluded via a probability threshold.

## When NOT to use

- Input contigs are mostly shorter than 1000 bp, where k-mer-based methods perform poorly and results are unreliable
- The goal is recovering circular plasmid contigs from an assembly graph with paired reads (use PlasmidSpades or Recycler instead)
- The goal is identifying specific plasmid replicons in an isolate genome assembly (use PlasmidFinder instead)

## Inputs

- FASTA file of assembly contigs or genomic fragments (>1000 bp recommended)
- Trained neural network model files from github.com/smaegol/PlasFlow_models

## Outputs

- Tabular predictions file (.tsv) with per-sequence plasmid/chromosome labels and class probabilities
- FASTA files binning input sequences into plasmids, chromosomes, and unclassified
- Validation accuracy report (e.g., validation_accuracy_report.csv) when benchmarking thresholds

## How to apply

First filter input sequences by length, leaving only those longer than 1000 bp, using the PlasFlow Perl script (short contigs give poor k-mer coverage and unreliable results). Run PlasFlow.py with a trained model from the PlasFlow_models repository under Python 3.5 / TensorFlow 0.10.0rc0, specifying --input FASTA, --output TSV, and a probability threshold (default 0.7); use --batch_size to control sequences per k-mer frequency batch when memory is limited. Sweep the threshold to tune the trade-off: on the 61,221-fragment RefSeq validation set, accuracy rose from ~89.52% (no threshold) to ~94.79% at threshold 0.7 and ~96.41% at threshold 0.8, with ~25.03% of sequences left unclassified at 0.8. Finally use PlasFlow's binning to split input sequences into plasmid, chromosome, and unclassified FASTA files, and optionally use its taxonomic labels for initial host-phylum classification.

## Related tools

- **PlasFlow** (Main classifier: computes k-mer frequency features and applies the trained neural network with probability thresholding and output binning) — https://github.com/smaegol/PlasFlow
- **PlasFlow_models** (Provides the pre-trained models tested and described in the manuscript) — https://github.com/smaegol/PlasFlow_models
- **TensorFlow** (Runtime for the deep artificial neural network classifier (version 0.10.0rc0 required))
- **scikit-learn** (Python dependency (0.18.1) used in the PlasFlow environment)

## Examples

```
PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Overall classification accuracy on the RefSeq validation fragment set (~61,221 fragments, 1-15.8 kb) should reach ~89.52% unfiltered, ~94.79% at threshold 0.7, and ~96.41% at threshold 0.8 within small numerical tolerance
- At threshold 0.8, roughly 25.03% of validation sequences should be reported unclassified
- The predictions TSV contains a probability column per sequence and no class probability falls below the specified --threshold among classified entries
- Three output FASTA files (plasmid, chromosome, unclassified) partition the filtered input sequences with no sequence duplicated or lost
- Short (<1000 bp) sequences are absent from classification inputs after the length-filtering step

## Limitations

- PlasFlow, like other k-mer-based methods, performs poorly on short sequences because proper k-mer coverage is hard to obtain, so results for contigs <1000 bp are unreliable
- Thresholding trades recall for precision: at threshold 0.8 about 25.03% of sequences remain unclassified
- The project is NOT MAINTAINED and pins old dependencies (Python 3.5, TensorFlow 0.10.0rc0, scikit-learn 0.18.1); large datasets may require reducing --batch_size

## Evidence
