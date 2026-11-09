---
name: benchmark-error-profile-reproduction
description: Use when two sequence-classification tools must be compared on fragments
  with known chromosomal/plasmid origin, and you need per-tool error rates (FPR, FNR)
  plus a concordance fraction between their predictions.
when_to_use_negative:
- Fragments are shorter than 1000 bp — kmer-based methods like PlasFlow do not perform
  well on short sequences and results are unreliable
- No ground-truth origin labels exist for the sequences, so true/false positive and
  negative classes cannot be assigned
- The goal is recovering circular plasmids from an assembly graph with paired reads
  (use Recycler or PlasmidSpades instead of signature-based classification)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PlasFlow
  role: Reference classifier run with probability threshold 0.7 to produce plasmid/chromosome
    predictions on the validation fragments
  repo: https://github.com/smaegol/PlasFlow
- name: cBar
  role: Comparator classifier using genomic signatures (self-organizing maps) run
    on the same fragments to obtain its error profile
- name: PlasFlow_models
  role: Repository of trained neural network models needed to run PlasFlow predictions
  repo: https://github.com/smaegol/PlasFlow_models
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:004031359989c02c1c3fdd6bf40f06525c92c90035d8cba97343b7a9cd910cd3
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
  - cBar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# benchmark-error-profile-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Head-to-head benchmarking of two plasmid classifiers (cBar and PlasFlow) on the same RefSeq-derived validation fragments, computing false-positive/false-negative rates per tool and pairwise prediction concordance to reproduce the comparison reported in the PlasFlow paper.

## When to use

Use when two sequence-classification tools must be compared on fragments with known chromosomal/plasmid origin, and you need per-tool error rates (FPR, FNR) plus a concordance fraction between their predictions.

## When NOT to use

- Fragments are shorter than 1000 bp — kmer-based methods like PlasFlow do not perform well on short sequences and results are unreliable
- No ground-truth origin labels exist for the sequences, so true/false positive and negative classes cannot be assigned
- The goal is recovering circular plasmids from an assembly graph with paired reads (use Recycler or PlasmidSpades instead of signature-based classification)

## Inputs

- Validation fragment FASTA sets of varying length, derived from chromosome and plasmid sequences downloaded from RefSeq, with known origin labels
- cBar executable (genomic-signature-based plasmid predictor)
- PlasFlow.py with trained neural network models (github.com/smaegol/PlasFlow_models)

## Outputs

- Per-fragment chromosomal/plasmid predictions from cBar and PlasFlow (PlasFlow predictions in a tabular TSV file)
- False-positive rate and false-negative rate for each tool (target: cBar 6.46% / 15.14%, PlasFlow 2.21% / 3.00%)
- Concordance fraction between the two tools' predictions (target: ~79%)

## How to apply

Obtain validation fragment sets of varying lengths from chromosomes and plasmids downloaded from RefSeq, as used in the PlasFlow benchmark, with known fragment origin labels. Run cBar, which identifies plasmid sequences based on genomic signatures, on each fragment set to get chromosomal/plasmid calls. Run PlasFlow on the identical fragments with PlasFlow.py and the probability filtering threshold set to 0.7 (its default), so uncertain predictions land in the 'unclassified' bin rather than counting as plasmid or chromosome. Classify each cBar prediction as true/false positive or true/false negative against the known origin and compute overall false-positive and false-negative rates; do the same for PlasFlow. Quantify concordance as the fraction of fragments assigned the same class by both tools. Accept the reproduction if cBar yields FPR 6.46% and FNR 15.14%, PlasFlow yields FPR 2.21% and FNR 3.00%, and the tools agree on approximately 79% of predictions, matching the manuscript values.

## Related tools

- **PlasFlow** (Reference classifier run with probability threshold 0.7 to produce plasmid/chromosome predictions on the validation fragments) — https://github.com/smaegol/PlasFlow
- **cBar** (Comparator classifier using genomic signatures (self-organizing maps) run on the same fragments to obtain its error profile)
- **PlasFlow_models** (Repository of trained neural network models needed to run PlasFlow predictions) — https://github.com/smaegol/PlasFlow_models

## Examples

```
PlasFlow.py --input validation_fragments.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- cBar error rates on the validation fragments match FPR 6.46% and FNR 15.14%
- PlasFlow error rates (threshold 0.7) match FPR 2.21% and FNR 3.00%
- Concordance between cBar and PlasFlow predictions is approximately 79% of fragments
- PlasFlow output TSV contains all input fragments binned to plasmid, chromosome, or unclassified (uncertain predictions excluded by the 0.7 threshold)
- PlasFlow false-prediction rate stays low (close to 5%), consistent with the reported near-96% classification accuracy

## Limitations

- Prediction quality is unreliable for short sequences — length filtering to keep only sequences longer than 1000 bp is recommended because kmer coverage is insufficient for short fragments
- Models must be updated with newly published genomic data; the training set was fixed at development time (10 kb fragments, dataset divided by mean taxonomic origin/phylum)
- PlasFlow neural network models were trained with TensorFlow 0.10.0 on Python 3.5; the package is no longer maintained (version 1.1), so dependency resolution may be brittle

## Evidence

- [methods] PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7: [evidence span withheld — anchor status: not_found]
- [methods] filter sequences by length, leaving only those longer than 1000 bp: [evidence span withheld — anchor status: not_found]
- [discussion] comparison of both programs revealed that PlasFlow outperformed cBar in plasmid identification: [evidence span withheld — anchor status: not_found]
- [discussion] testing datasets comprising of different length fragments of chromosomes and plasmids downloaded from the RefSeq database: [evidence span withheld — anchor status: not_found]
- [readme] incorporates the thresholding which allows for exclusion of incertain predictions: [evidence span withheld — anchor status: not_found]
