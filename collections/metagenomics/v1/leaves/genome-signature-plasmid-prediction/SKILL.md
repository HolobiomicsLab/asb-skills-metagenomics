---
name: genome-signature-plasmid-prediction
description: Use when metagenomic or genome assembly contigs (ideally >1000 bp, FASTA)
  must be labelled plasmid vs chromosome without prior knowledge of sample taxonomic
  composition, e.g. plasmidome profiling of environmental assemblies.
when_to_use_negative:
- Input sequences are shorter than 1000 bp — kmer-based prediction does not perform
  well on short sequences and results are unreliable
- The goal is recovering circular plasmids from a de Bruijn assembly graph with paired
  reads (use PlasmidSpades or Recycler instead, which require the assembly graph)
- Plasmid replicon identification in complete bacterial genome assemblies via known
  replicon sequences (PlasmidFinder's web-based approach) is the target
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: PlasFlow
  role: 'Primary classifier: computes kmer signatures, runs averaged neural network
    models, applies the 0.7 probability threshold, and writes predictions plus binned
    FASTA files'
  repo: https://github.com/smaegol/PlasFlow
- name: PlasFlow_models
  role: Repository holding all neural network models tested and created during PlasFlow
    development, used as the ensemble for prediction
  repo: https://github.com/smaegol/PlasFlow_models
- name: cBar
  role: Alternative genomic-signature tool (self-organizing maps) used as the benchmark
    comparator; PlasFlow outperformed it (e.g. FP 2.21% vs 6.46%, FN 3.00% vs 15.14%)
- name: scikit-learn
  role: TF-IDF transformation and L2 normalization of the kmer frequency matrix before
    neural network scoring
- name: TensorFlow
  role: Deep neural network runtime (version 0.10.0 required) executing the trained
    plasmid/chromosome classification models
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:77cad68aa587579bf7a22888ae06429d2d9a0d135eeeb44e14f07eabfdd25230
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
  - SPAdes
  - TensorFlow
  - cBar
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# genome-signature-plasmid-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify metagenomic contigs as plasmid or chromosome using kmer-frequency genome signatures scored by an averaged ensemble of TensorFlow neural network models (PlasFlow), applying a probability threshold (default 0.7) to exclude uncertain predictions.

## When to use

Use when metagenomic or genome assembly contigs (ideally >1000 bp, FASTA) must be labelled plasmid vs chromosome without prior knowledge of sample taxonomic composition, e.g. plasmidome profiling of environmental assemblies.

## When NOT to use

- Input sequences are shorter than 1000 bp — kmer-based prediction does not perform well on short sequences and results are unreliable
- The goal is recovering circular plasmids from a de Bruijn assembly graph with paired reads (use PlasmidSpades or Recycler instead, which require the assembly graph)
- Plasmid replicon identification in complete bacterial genome assemblies via known replicon sequences (PlasmidFinder's web-based approach) is the target

## Inputs

- Metagenomic or genome assembly contigs in FASTA format (e.g. SPAdes scaffolds such as Citrobacter_freundii_strain_CAV1321_scaffolds.fasta)
- Length-filtered FASTA (contigs >1000 bp) produced by filter_sequences_by_length.pl

## Outputs

- Tabular predictions file (e.g. test.plasflow_predictions.tsv) with contig_id, label, and per-class probability columns
- Three FASTA files binning input sequences into plasmids, chromosomes, and unclassified
- Per-sample summary of plasmid-origin fraction of sequences and cumulative scaffold length

## How to apply

First filter the input assembly with filter_sequences_by_length.pl, retaining only contigs longer than 1000 bp, because kmer-based prediction is unreliable on short sequences and short contigs inflate RAM usage. Compute kmer frequency signatures for each retained contig, transform them with TF-IDF and L2 normalization (scikit-learn), and run the PlasFlow neural network models (TensorFlow 0.10.0) to get probabilities over 26 chromosome.*/plasmid.* phylum-level classes; PlasFlow averages probabilities over its three best models and assigns the class with the maximum averaged probability. Invoke PlasFlow.py with --threshold 0.7 so contigs whose maximum averaged probability falls below the cutoff are labelled 'unclassified.unclassified'; tune --batch_size downward if memory issues arise on large datasets (PlasFlow 1.1). Finally write the tabular predictions and bin sequences into plasmid, chromosome, and unclassified FASTA files.

## Related tools

- **PlasFlow** (Primary classifier: computes kmer signatures, runs averaged neural network models, applies the 0.7 probability threshold, and writes predictions plus binned FASTA files) — https://github.com/smaegol/PlasFlow
- **PlasFlow_models** (Repository holding all neural network models tested and created during PlasFlow development, used as the ensemble for prediction) — https://github.com/smaegol/PlasFlow_models
- **cBar** (Alternative genomic-signature tool (self-organizing maps) used as the benchmark comparator; PlasFlow outperformed it (e.g. FP 2.21% vs 6.46%, FN 3.00% vs 15.14%))
- **scikit-learn** (TF-IDF transformation and L2 normalization of the kmer frequency matrix before neural network scoring)
- **TensorFlow** (Deep neural network runtime (version 0.10.0 required) executing the trained plasmid/chromosome classification models)

## Examples

```
filter_sequences_by_length.pl -input assembly.fasta -output filtered.fasta -thresh 1000; PlasFlow.py --input filtered.fasta --output plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- plasflow_predictions.tsv exists in tabular format with contig_id, label, and per-class probability columns matching the reference test file (Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv) at --threshold 0.7
- On the RefSeq chromosome/plasmid fragment validation sets, expected error rates are FP 2.21% and FN 3.00% for PlasFlow versus cBar's 6.46% and 15.14%, with ~79% prediction concordance between the two tools
- For KOW/ZS mine metagenome assemblies (MG-RAST mgp80771), plasmid fractions should fall in the reported ranges: 25.02–27.05% of sequences and 14.94–20.66% of cumulative scaffold length
- Contigs whose maximum averaged class probability is below the 0.7 threshold appear as 'unclassified.unclassified' in the label column rather than being force-assigned
- No contigs shorter than 1000 bp are present in the classified input after filter_sequences_by_length.pl pre-filtering

## Limitations

- Prediction on short sequences (<1000 bp) is unreliable because proper kmer coverage cannot be obtained from them
- The trained model can become outdated as genomic databases grow, and can 'only be overcome by regularly updating the model with newly published genomic data'
-  PlasFlow project is NOT MAINTAINED (per README); requires a Python 3.5 environment with TensorFlow 0.10.0rc0 and specific scikit-learn 0.18.1, so dependency pinning is fragile
- Large datasets may cause memory problems; use smaller --batch_size values or the 1.1 release

## Evidence
