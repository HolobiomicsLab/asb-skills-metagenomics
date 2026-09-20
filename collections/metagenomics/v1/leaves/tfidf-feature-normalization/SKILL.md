---
name: tfidf-feature-normalization
description: Use when building genomic-signature features from kmer frequency counts
  of DNA sequences (e.g. plasmid vs. chromosome classification) and the downstream
  model expects TF-IDF-weighted, L2-normalized feature vectors rather than raw counts.
when_to_use_negative:
- Input sequences are shorter than 1000 bp — PlasFlow notes kmer-based prediction
  is unreliable on short contigs because proper kmer coverage cannot be obtained
- The downstream classifier expects raw kmer counts (e.g. some training pipelines
  in PlasFlow_preprocessing use raw kmern_split_raw_counts_tax_classes_numeric_annotated_filtered_tax.tsv
  files)
- Applying to non-signature-based data (e.g. assembly graphs or read alignments for
  tools like PlasmidSpades or Recycler, which PlasFlow explicitly contrasts itself
  with)
edam_operation: http://edamontology.org/operation_2936
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PlasFlow
  role: pipeline whose feature-preparation step transforms kmer frequency matrices
    with TF-IDF and L2 normalization before neural network classification
  repo: https://github.com/smaegol/PlasFlow
- name: scikit-learn
  role: provides TfidfTransformer and L2 Normalizer used to transform the kmer frequency
    matrix
- name: TensorFlow
  role: consumes the TF-IDF-normalized features in the neural network models (version
    0.10.0 required)
- name: filter_sequences_by_length.pl
  role: pre-filters contigs to >1000 bp so kmer counts used for TF-IDF are reliable
  repo: https://github.com/smaegol/PlasFlow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkx1321
    title: 'PlasFlow: predicting plasmid sequences in metagenomic data using genome
      signatures'
  source_content_hash: sha256:4cdba8ade858363fb17827ee0c4248969cfdf034b8c77871b3055890c304dcd0
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
  - TensorFlow
  - filter_sequences_by_length.pl
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2936
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tfidf-feature-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transform raw kmer frequency count matrices (genome signatures) with TF-IDF weighting followed by L2 normalization using scikit-learn, as done in PlasFlow before feeding features into its TensorFlow neural network classifiers.

## When to use

Use when building genomic-signature features from kmer frequency counts of DNA sequences (e.g. plasmid vs. chromosome classification) and the downstream model expects TF-IDF-weighted, L2-normalized feature vectors rather than raw counts.

## When NOT to use

- Input sequences are shorter than 1000 bp — PlasFlow notes kmer-based prediction is unreliable on short contigs because proper kmer coverage cannot be obtained
- The downstream classifier expects raw kmer counts (e.g. some training pipelines in PlasFlow_preprocessing use raw kmern_split_raw_counts_tax_classes_numeric_annotated_filtered_tax.tsv files)
- Applying to non-signature-based data (e.g. assembly graphs or read alignments for tools like PlasmidSpades or Recycler, which PlasFlow explicitly contrasts itself with)

## Inputs

- kmer frequency count matrix (rows = sequences/contigs, columns = kmer features) in TSV form
- FASTA file of metagenomic contigs filtered to >1000 bp

## Outputs

- TF-IDF-weighted, L2-normalized feature matrix suitable for neural network classification
- fitted scikit-learn transformer objects for reapplying the same transformation at prediction time

## How to apply

First compute kmer frequency counts (genome signatures) for each sequence, using sequences longer than 1000 bp since kmer coverage is unreliable on short contigs. Fit a scikit-learn TfidfTransformer on the kmer count matrix, then apply L2 normalization (Normalizer) so each sequence's feature vector has unit norm. The rationale in PlasFlow is to weight informative kmers and scale vectors so the neural network models (trained on 10 kb fragments from 9,565 bacterial chromosomes and plasmids, split by phylum) receive comparably scaled inputs. The transformed matrix is then passed to the TensorFlow models, whose averaged class probabilities are thresholded at 0.7 (default) before label assignment. Reuse the exact transform (fit on training data, apply to test data) to keep feature spaces consistent between training and prediction.

## Related tools

- **PlasFlow** (pipeline whose feature-preparation step transforms kmer frequency matrices with TF-IDF and L2 normalization before neural network classification) — https://github.com/smaegol/PlasFlow
- **scikit-learn** (provides TfidfTransformer and L2 Normalizer used to transform the kmer frequency matrix)
- **TensorFlow** (consumes the TF-IDF-normalized features in the neural network models (version 0.10.0 required))
- **filter_sequences_by_length.pl** (pre-filters contigs to >1000 bp so kmer counts used for TF-IDF are reliable) — https://github.com/smaegol/PlasFlow

## Examples

```
from sklearn.feature_extraction.text import TfidfTransformer, Normalizer; X = Normalizer(copy=False).fit_transform(TfidfTransformer().fit_transform(kmer_counts))
```

## Evaluation signals

- Each row of the transformed matrix should have L2 norm equal to 1 (unit-length feature vectors)
- Feature dimensionality must match the training kmer feature space (e.g. the kmern kmer-size feature columns) or prediction will fail
- End-to-end check: predictions file exists and its label/probability columns match the reference results (Citrobacter_freundii_strain_CAV1321_scaffolds.fasta.PlasFlow.tsv) with the 0.7 threshold applied
- Sequences whose maximum averaged class probability falls below 0.7 must be labelled 'unclassified.unclassified'
- Reported classification accuracy on test data should be in the range PlasFlow reports (up to 96%) when the transformation matches training

## Limitations

- kmer-based signatures perform poorly on short sequences; PlasFlow recommends filtering to sequences longer than 1000 bp
- Models must be regularly updated with newly published genomic data to avoid degradation as databases evolve
- PlasFlow 1.1's batch processing may require adjusting --batch_size for large datasets; the original project is no longer maintained and pins old dependencies (Python 3.5, TensorFlow 0.10.0, scikit-learn 0.18.1)

## Evidence

- [methods] Transform the kmer frequency matrix with TF-IDF and L2 normalization using scikit-learn.: [evidence span withheld — anchor status: not_found]
