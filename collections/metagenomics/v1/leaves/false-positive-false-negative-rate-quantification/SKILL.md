---
name: false-positive-false-negative-rate-quantification
description: Use when benchmarking plasmid/chromosome classification tools (e.g. cBar
  vs PlasFlow) on fragments of known origin and you need error rates plus percent
  agreement between tools.
when_to_use_negative:
- Fragments of unknown origin — true/false positive classification against ground
  truth is impossible without known fragment origin
- Sequences shorter than 1000 bp — PlasFlow, similarly to other kmer-based methods,
  does not perform well on short sequences, as it is hard to get proper kmer coverage
  from them
- Assembly-graph-based plasmid recovery tasks (e.g. PlasmidSpades or Recycler), which
  require an assembly graph and paired sequencing reads rather than fragment-level
  classification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3941
tools:
- name: cBar
  role: Genomic-signature-based predictor whose per-fragment plasmid/chromosome calls
    are scored as true/false positives and negatives against known fragment origin
- name: PlasFlow
  role: Neural-network-based reference predictor run with --threshold 0.7; its error
    rates and concordance with cBar are quantified
  repo: https://github.com/smaegol/PlasFlow
- name: filter_sequences_by_length.pl
  role: Pre-classification filtering of validation fragments to sequences longer than
    1000 bp prior to PlasFlow
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:386928b6d2baff5112104490b49a2d16a148374d24c56ad5f491b4f60d6180a1
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
  - cBar
  - filter_sequences_by_length.pl
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# false-positive-false-negative-rate-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies plasmid-prediction error profiles (false-positive and false-negative rates) and inter-tool concordance for cBar versus PlasFlow on the RefSeq-derived validation fragments used in the PlasFlow benchmark.

## When to use

Use when benchmarking plasmid/chromosome classification tools (e.g. cBar vs PlasFlow) on fragments of known origin and you need error rates plus percent agreement between tools.

## When NOT to use

- Fragments of unknown origin — true/false positive classification against ground truth is impossible without known fragment origin
- Sequences shorter than 1000 bp — PlasFlow, similarly to other kmer-based methods, does not perform well on short sequences, as it is hard to get proper kmer coverage from them
- Assembly-graph-based plasmid recovery tasks (e.g. PlasmidSpades or Recycler), which require an assembly graph and paired sequencing reads rather than fragment-level classification

## Inputs

- Validation datasets of chromosome and plasmid fragments of varying lengths downloaded from RefSeq and used in the PlasFlow benchmark
- Known origin labels (chromosomal vs plasmid) for each validation fragment
- cBar chromosomal/plasmid predictions per fragment
- PlasFlow predictions tabular file (e.g. test.plasflow_predictions.tsv) produced with --threshold 0.7

## Outputs

- cBar overall false-positive rate (expected 6.46%) and false-negative rate (expected 15.14%)
- PlasFlow false-positive rate (expected 2.21%) and false-negative rate (expected 3.00%) at threshold 0.7
- Inter-tool concordance as the fraction of fragments assigned the same class by both tools (~79%)

## How to apply

Run cBar, which identifies plasmid sequences based on genomic signatures, on each validation fragment set of chromosome and plasmid fragments of varying lengths downloaded from RefSeq. Run PlasFlow.py on the same fragments with the probability filtering threshold set to 0.7 (its default) to obtain reference predictions. Classify each cBar prediction as true/false positive or true/false negative against the known fragment origin, computing overall false-positive and false-negative rates. Quantify concordance between cBar and PlasFlow as the fraction of fragments assigned the same class by both tools. Accept the reproduction if cBar yields a 6.46% false-positive rate and 15.14% false-negative rate, PlasFlow yields 2.21% and 3.00%, and the tools agree on approximately 79% of predictions; with PlasFlow run without probability filtering, 79% of predictions were consistent between the two tools.

## Related tools

- **cBar** (Genomic-signature-based predictor whose per-fragment plasmid/chromosome calls are scored as true/false positives and negatives against known fragment origin)
- **PlasFlow** (Neural-network-based reference predictor run with --threshold 0.7; its error rates and concordance with cBar are quantified) — https://github.com/smaegol/PlasFlow
- **filter_sequences_by_length.pl** (Pre-classification filtering of validation fragments to sequences longer than 1000 bp prior to PlasFlow) — https://github.com/smaegol/PlasFlow

## Examples

```
PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- cBar false-positive rate reproduces the reported value of 6.46% and false-negative rate of 15.14%
- PlasFlow (threshold 0.7) false-positive rate reproduces 2.21% and false-negative rate 3.00%
- cBar and PlasFlow agree on approximately 79% of predictions (consistency measured when PlasFlow is run without probability filtering)
- Every fragment in each validation set is classified into exactly one of true positive, false positive, true negative, or false negative relative to its known origin
- All fragments scored have passed the >1000 bp length filter recommended before PlasFlow classification

## Limitations

- PlasFlow results for short sequences (<1000 bp) are unreliable because it is hard to get proper kmer coverage from them
- PlasFlow's probability filtering (threshold 0.7) excludes uncertain predictions, so thresholded and unthresholded runs yield different concordance with cBar
- cBar uses self-organizing maps (SOMs) whereas PlasFlow implements a deep artificial neural network; trained models depend on the underlying genomic data and can only be improved by regularly updating the model with newly published genomic data
- No version numbers were stated for PlasFlow or cBar in the comparison, complicating exact reproduction

## Evidence

- [methods] Comparison of validation error rates and concordance protocol: [evidence span withheld — anchor status: not_found]
