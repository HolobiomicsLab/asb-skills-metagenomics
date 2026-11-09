---
name: cbar-chromosomal-plasmid-classification
description: 'Use when benchmarking plasmid-identification tools: you have chromosome/plasmid
  validation fragments of known origin (e.g. RefSeq-derived) and need cBar''s error
  rates and its agreement with PlasFlow predictions.'
when_to_use_negative:
- When the goal is metagenomic contig classification rather than a benchmarked comparison
  on fragments of known origin — PlasFlow alone is the appropriate tool there
- 'When fragments are very short: cBar and other kmer/genomic-signature methods do
  not perform well on short sequences since kmer coverage is insufficient'
- When prior assumptions about taxonomic or functional composition of the sample cannot
  be verified against a known ground truth, making error-rate scoring impossible
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: cBar
  role: Classifies each validation fragment as chromosomal or plasmid using genomic
    signatures; its predictions are scored against ground truth and compared with
    PlasFlow
- name: PlasFlow
  role: Produces the reference chromosomal/plasmid predictions (with probability filtering
    at threshold 0.7) against which cBar concordance is computed
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:900d41e5cfa3ccca0a8c17d98ba041509413dbfaef61565fb53f5b214623cc7f
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cbar-chromosomal-plasmid-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run cBar, a genomic-signature-based method implemented with self-organizing maps (SOMs), to classify sequence fragments as chromosomal or plasmid, and compare its false-positive/false-negative profile against PlasFlow's neural-network predictions with a 0.7 probability threshold. This skill captures the cBar classification and cross-tool concordance analysis used in the PlasFlow benchmark.

## When to use

Use when benchmarking plasmid-identification tools: you have chromosome/plasmid validation fragments of known origin (e.g. RefSeq-derived) and need cBar's error rates and its agreement with PlasFlow predictions.

## When NOT to use

- When the goal is metagenomic contig classification rather than a benchmarked comparison on fragments of known origin — PlasFlow alone is the appropriate tool there
- When fragments are very short: cBar and other kmer/genomic-signature methods do not perform well on short sequences since kmer coverage is insufficient
- When prior assumptions about taxonomic or functional composition of the sample cannot be verified against a known ground truth, making error-rate scoring impossible

## Inputs

- Validation datasets of chromosome fragments of varying lengths (FASTA) downloaded from RefSeq
- Validation datasets of plasmid fragments of varying lengths (FASTA) downloaded from RefSeq
- Known ground-truth labels (chromosomal vs plasmid origin) for each fragment

## Outputs

- cBar chromosomal/plasmid predictions per validation fragment
- PlasFlow predictions (test.plasflow_predictions.tsv) at threshold 0.7 for the same fragments
- cBar overall false-positive rate (~6.46%) and false-negative rate (~15.14%)
- Concordance fraction between cBar and PlasFlow predictions (~79% unfiltered)

## How to apply

Obtain validation datasets of chromosome and plasmid fragments of varying lengths downloaded from RefSeq (the same sets used in the PlasFlow benchmark). Run cBar, which identifies plasmid sequences based on genomic signatures, on each fragment set to obtain chromosomal/plasmid predictions. Run PlasFlow.py on the same fragments with the probability threshold set to 0.7 (its default) to obtain reference predictions, having first filtered sequences to those longer than 1000 bp. Score each cBar call against the known fragment origin to compute the false-positive rate (expected 6.46%) and false-negative rate (expected 15.14%). Quantify concordance between cBar and PlasFlow as the fraction of fragments assigned the same class by both tools — approximately 79% of predictions were consistent when PlasFlow was run without probability filtering. Rationale: cBar uses self-organizing maps (SOMs) while PlasFlow implements a deep artificial neural network, so the comparison highlights how model architecture and probability filtering affect error rates.

## Related tools

- **cBar** (Classifies each validation fragment as chromosomal or plasmid using genomic signatures; its predictions are scored against ground truth and compared with PlasFlow)
- **PlasFlow** (Produces the reference chromosomal/plasmid predictions (with probability filtering at threshold 0.7) against which cBar concordance is computed) — https://github.com/smaegol/PlasFlow

## Examples

```
cbar -i validation_fragments.fasta -o cbar_predictions.txt; PlasFlow.py --input filtered_output.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- cBar yields a false-positive rate of 6.46% and a false-negative rate of 15.14% on the RefSeq validation fragments, matching the manuscript values
- PlasFlow at threshold 0.7 yields a false-positive rate of 2.21% and a false-negative rate of 3.00%
- Approximately 79% of predictions are consistent between cBar and PlasFlow when PlasFlow is run without probability filtering
- Every fragment receives exactly one class assignment (chromosome or plasmid) from each tool, allowing per-fragment TP/FP/TN/FN tabulation
- PlasFlow's error rates fall below cBar's, consistent with the reported finding that PlasFlow outperformed cBar in plasmid identification

## Limitations

- cBar uses self-organizing maps (SOMs), whereas PlasFlow implements a deep artificial neural network; the comparison revealed PlasFlow outperformed cBar in plasmid identification
- PlasFlow does not perform well on short sequences because proper kmer coverage is hard to obtain, and results for short sequences are unreliable — the same limitation applies to genomic-signature methods like cBar
- Genomic-signature models degrade as newly published genomic data accumulates, so model updating is required over time

## Evidence

- [intro] cBar method description: "A method for the identification of plasmid sequences based on genomic signatures was implemented in the cBar software (30)."
- [discussion] cBar vs PlasFlow algorithmic difference: "Unlike cBar that uses self-organizing maps (SOMs), PlasFlow implements a deep artificial neural network"
- [discussion] Comparison outcome: "comparison of both programs revealed that PlasFlow outperformed cBar in plasmid identification"
- [methods] PlasFlow run command with threshold: "PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7"
- [readme] Length filtering recommendation: "it is highly recommended to filter sequences by length, leaving only those longer than 1000 bp."
