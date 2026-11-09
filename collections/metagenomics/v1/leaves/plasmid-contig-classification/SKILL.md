---
name: plasmid-contig-classification
description: Use when you have metagenomic or bacterial genome assembly contigs (ideally
  >1000 bp) and need to label them as plasmid-derived versus chromosomal without prior
  knowledge of sample taxonomic composition.
when_to_use_negative:
- Input contains many contigs shorter than 1000 bp — results for short sequences are
  unreliable because k-mer coverage is insufficient; filter by length first
- The plasmids of interest require an assembly graph and paired reads (e.g., Recycler/PlasmidSpades-style
  recovery of circular contigs) rather than signature-based contig labeling
- You need replicon typing of known Enterobacteriaceae plasmids (PlasmidFinder use
  case) rather than sequence-origin classification
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: Neural-network classifier that assigns contigs to plasmid/chromosome/phylum
    classes and bins sequences by predicted origin
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Perl/Bioperl script that retains only contigs longer than a threshold (1000
    bp) prior to classification
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow_models
  role: Repository of trained neural network models tested and created during PlasFlow
    development
  repo: https://github.com/smaegol/PlasFlow_models
- name: TensorFlow
  role: Backend (version 0.10.0) executing the trained neural network for k-mer-based
    sequence classification
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:d62cac21d20d3567c1373b8306164e8a3f4ce611de24ee3bacf2927807048774
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
  - filter_sequences_by_length.pl
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# plasmid-contig-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify assembled metagenomic/isolate contigs as plasmid, chromosome, or unclassified using PlasFlow, a neural-network classifier trained on k-mer genome signatures of 9,565 RefSeq bacterial chromosomes and plasmids, with probability thresholding to exclude uncertain predictions.

## When to use

Use when you have metagenomic or bacterial genome assembly contigs (ideally >1000 bp) and need to label them as plasmid-derived versus chromosomal without prior knowledge of sample taxonomic composition.

## When NOT to use

- Input contains many contigs shorter than 1000 bp — results for short sequences are unreliable because k-mer coverage is insufficient; filter by length first
- The plasmids of interest require an assembly graph and paired reads (e.g., Recycler/PlasmidSpades-style recovery of circular contigs) rather than signature-based contig labeling
- You need replicon typing of known Enterobacteriaceae plasmids (PlasmidFinder use case) rather than sequence-origin classification

## Inputs

- Metagenomic or isolate assembly contigs in FASTA format (e.g., SPAdes 3.9.1 scaffolds, such as Citrobacter freundii strain CAV1321, GCA_001022155.1)
- Probability threshold value (default 0.7; 0.8 for higher-accuracy, higher-uncertainty runs)
- Optional: GenBank reference plasmids of the analyzed genomes for recall/precision benchmarking

## Outputs

- Tab-separated prediction file (contig_id, contig_name, contig_length, id, label, per-class probabilities)
- FASTA files binning input sequences into plasmids, chromosomes, and unclassified
- Initial phylum-level taxonomic classification of predicted sequences (26 chromosome/plasmid classes)

## How to apply

First filter the assembly to contigs longer than 1000 bp using PlasFlow's filter_sequences_by_length.pl, since k-mer-based methods perform poorly on short sequences and filtering prevents excessive RAM use. Run PlasFlow.py with --input <filtered_fasta> --output <predictions.tsv> and a probability threshold (default 0.7) so sequences whose class probability falls below the threshold are labeled unclassified; raising the threshold to 0.8 increased validation accuracy to 96.41% but left 25.03% of sequences unclassified. The classifier assigns contigs to 26 phylum-level chromosome/plasmid classes and bins sequences into plasmids, chromosomes, and unclassified FASTA files. Assess quality via a tab-separated prediction file (contig_id, contig_name, contig_length, label, per-class probabilities) and, when references exist, compute plasmid recall/precision — e.g., 85.98% recall / 72.17% precision on 42 plasmid-rich bacterial genomes, versus cBar (76.82% recall) and Recycler (12.0% recall).

## Related tools

- **PlasFlow** (Neural-network classifier that assigns contigs to plasmid/chromosome/phylum classes and bins sequences by predicted origin) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Perl/Bioperl script that retains only contigs longer than a threshold (1000 bp) prior to classification) — https://github.com/smaegol/PlasFlow
- **PlasFlow_models** (Repository of trained neural network models tested and created during PlasFlow development) — https://github.com/smaegol/PlasFlow_models
- **TensorFlow** (Backend (version 0.10.0) executing the trained neural network for k-mer-based sequence classification)

## Examples

```
filter_sequences_by_length.pl -input input_dataset.fasta -output filtered_output.fasta -thresh 1000 && PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- At threshold 0.7, validation-dataset accuracy should be ~94.79% (89.52% without thresholding); at threshold 0.8, ~96.41% with ~25.03% unclassified
- On the 42 plasmid-rich bacterial genomes, plasmid recall should approach 85.98% and precision 72.17%
- Every input contig appears exactly once in the predictions TSV with label = plasmid, chromosome, or unclassified, and the three FASTA bins partition the filtered input
- Class probabilities in the output are consistent with the chosen threshold: labels other than 'unclassified' must have class probability ≥ threshold

## Limitations

- K-mer-based classification performs poorly on sequences shorter than 1000 bp, so short contigs must be filtered out and their origin cannot be predicted
- Higher probability thresholds exclude uncertain predictions and lower the false-hit rate but leave a substantial unclassified fraction (~25% at threshold 0.8)
- The project is no longer maintained ('NOT MAINTAINED', version 1.1, 2018-05-25) and requires legacy dependencies (Python 3.5, TensorFlow 0.10.0, scikit-learn 0.18.1); the model should be updated with newly published genomic data to overcome training-set bias
- PlasFlow cannot recover plasmid sequences from assembly graphs with paired reads the way Recycler or PlasmidSpades do

## Evidence
