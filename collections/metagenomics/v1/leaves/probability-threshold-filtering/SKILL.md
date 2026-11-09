---
name: probability-threshold-filtering
description: Use when classifying metagenomic contigs with PlasFlow and you need to
  control the accuracy/unclassified-fraction trade-off, or when reproducing the paper's
  reported accuracy at thresholds 0.7 (94.79%) and 0.8 (96.41%).
when_to_use_negative:
- Input contigs are shorter than 1000 bp, where k-mer-based probabilities are unreliable
  anyway (pre-filter them out instead)
- Complete recovery of every input sequence into a plasmid/chromosome bin is required
  and no unclassified output can be tolerated
- The task requires identifying plasmid replicons or using assembly graph/paired-read
  information (use PlasmidFinder, PlasmidSpades, or Recycler instead)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PlasFlow
  role: Runs the neural network ensemble and applies the --threshold probability filter,
    writing low-confidence contigs as unclassified
  repo: https://github.com/smaegol/PlasFlow
- name: TensorFlow
  role: Executes the three best neural network models whose averaged probabilities
    are thresholded
- name: scikit-learn
  role: Transforms k-mer frequency features with TF-IDF and L2 normalization before
    probability scoring
- name: filter_sequences_by_length.pl
  role: Pre-filters contigs to >1000 bp so thresholded probabilities are only computed
    on reliable-length sequences
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:5ffa3e49b79b0d327ea165f6ab53d26798c883a1812f873415f8ef3b551a9275
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
  - Python
  - TensorFlow
  - filter_sequences_by_length.pl
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# probability-threshold-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a class-probability cutoff (default 0.7, tunable to 0.8) to PlasFlow's ensemble neural network predictions so that contigs whose maximum averaged probability falls below the threshold are labelled 'unclassified.unclassified', trading coverage for accuracy.

## When to use

Use when classifying metagenomic contigs with PlasFlow and you need to control the accuracy/unclassified-fraction trade-off, or when reproducing the paper's reported accuracy at thresholds 0.7 (94.79%) and 0.8 (96.41%).

## When NOT to use

- Input contigs are shorter than 1000 bp, where k-mer-based probabilities are unreliable anyway (pre-filter them out instead)
- Complete recovery of every input sequence into a plasmid/chromosome bin is required and no unclassified output can be tolerated
- The task requires identifying plasmid replicons or using assembly graph/paired-read information (use PlasmidFinder, PlasmidSpades, or Recycler instead)

## Inputs

- FASTA of filtered metagenomic assembly contigs (>1000 bp)
- PlasFlow predictions TSV with per-label class probabilities
- Probability threshold value (default 0.7)

## Outputs

- Tabular predictions file (e.g. test.plasflow_predictions.tsv) with labels and probabilities, unclassified entries marked 'unclassified.unclassified'
- FASTA files binning input sequences into plasmids, chromosomes, and unclassified
- Accuracy and unclassified-fraction summary (e.g. validation_accuracy_report.csv)

## How to apply

Run PlasFlow's TensorFlow neural network ensemble on TF-IDF/L2-normalized k-mer frequency features of contigs >1000 bp (pre-filtered with filter_sequences_by_length.pl), obtain per-label class probabilities for the chromosome.* and plasmid.* classes, and assign each contig the label with the highest averaged probability. Apply the probability threshold with PlasFlow.py --threshold (default 0.7); sequences whose maximum average score falls below the threshold are written as 'unclassified.unclassified' and binned into a separate unclassified FASTA. The rationale is that 'excluding low-probability classifications is advantageous for PlasFlow and increases overall accuracy': raising the threshold from unfiltered (~89.52% accuracy) to 0.7 (~94.79%) and 0.8 (~96.41%) on the 61,221-fragment validation set removes uncertain predictions at the cost of leaving ~25.03% of sequences unclassified at threshold 0.8. Choose the threshold based on whether downstream analyses prioritize recall (lower/no threshold) or precision (0.7–0.8).

## Related tools

- **PlasFlow** (Runs the neural network ensemble and applies the --threshold probability filter, writing low-confidence contigs as unclassified) — https://github.com/smaegol/PlasFlow
- **TensorFlow** (Executes the three best neural network models whose averaged probabilities are thresholded)
- **scikit-learn** (Transforms k-mer frequency features with TF-IDF and L2 normalization before probability scoring)
- **filter_sequences_by_length.pl** (Pre-filters contigs to >1000 bp so thresholded probabilities are only computed on reliable-length sequences) — https://github.com/smaegol/PlasFlow

## Examples

```
PlasFlow.py --input Citrobacter_freundii_strain_CAV1321_scaffolds.fasta --output test.plasflow_predictions.tsv --threshold 0.7
```

## Evaluation signals

- Unfiltered PlasFlow accuracy on the 61,221-fragment validation set is ~89.52%
- Accuracy at --threshold 0.7 is ~94.79% and at 0.8 is ~96.41% (within small numerical tolerance)
- Fraction of sequences labelled unclassified at threshold 0.8 is ~25.03%
- Predictions file contains label and probability columns matching the reference Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv with the 0.7 threshold applied
- Three FASTA bins (plasmids, chromosomes, unclassified) are produced and the unclassified bin grows as the threshold increases

## Limitations

- Higher thresholds increase accuracy but leave a substantial fraction of sequences unclassified (~25.03% at threshold 0.8), reducing plasmid recovery
- PlasFlow does not perform well on short sequences (<1000 bp) because proper k-mer coverage is hard to obtain, so thresholded results on short contigs are unreliable
- The model depends on training data composition; performance degrades on phyla underrepresented in the RefSeq training set and the model requires regular updating with newly published genomic data

## Evidence
