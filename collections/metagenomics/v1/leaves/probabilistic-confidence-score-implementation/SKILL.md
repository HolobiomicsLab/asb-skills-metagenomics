---
name: probabilistic-confidence-score-implementation
description: Use when implementing or reproducing supervised kmer-based family scoring/confidence
  scores that must penalize kmers shared between families and bound scores to (-1,
  1), as in Snekmer's model/search scoring.
when_to_use_negative:
- Inputs are raw protein sequences not yet recoded into AAR kmer vectors — the scoring
  module requires kmer count vectors, not FASTA input.
- The goal is unsupervised clustering (Snekmer's cluster mode) — this scoring is defined
  for supervised family classification only.
- A different scoring scheme is required, e.g., cosine-similarity-to-training-profile
  scores used in Snekmer's apply-mode snekmer_results.csv (Sequence, Prediction, Score,
  delta, Confidence).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Snekmer
  role: Source implementation providing the AAR kmer encoding, F_kmer/F'_kmer/S_kmer
    scoring definitions, and demo data (TIGRFAMs_annotation.ann); reimplement/reuse
    its scoring module in Python
  repo: github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Language in which the scoring formulas are implemented and unit-tested against
    hand-computed fixtures (Python 3.11+ required by Snekmer)
  repo: github.com/PNNL-CompBio/Snekmer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:76a1b306d9d0d6406b76d8f810554a42edc6dd2d21b1c2e7e19cb1dd9682894f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Snekmer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# probabilistic-confidence-score-implementation

## Summary

Implements Snekmer's supervised kmer scoring formulas: the family probability F_kmer = n_kmer/N, the modified family probability F'_kmer that penalizes kmers shared across families, and the overall score S_kmer combining F'_kmer with background kmer prevalence to produce confidence values in the open interval (-1, 1).

## When to use

Use when implementing or reproducing supervised kmer-based family scoring/confidence scores that must penalize kmers shared between families and bound scores to (-1, 1), as in Snekmer's model/search scoring.

## When NOT to use

- Inputs are raw protein sequences not yet recoded into AAR kmer vectors — the scoring module requires kmer count vectors, not FASTA input.
- The goal is unsupervised clustering (Snekmer's cluster mode) — this scoring is defined for supervised family classification only.
- A different scoring scheme is required, e.g., cosine-similarity-to-training-profile scores used in Snekmer's apply-mode snekmer_results.csv (Sequence, Prediction, Score, delta, Confidence).

## Inputs

- AAR-reduced-alphabet kmer count matrix (family x kmer)
- Family annotation file (e.g., TIGRFAMs_annotation.ann) mapping sequences to families
- Number of families G and per-family sequence counts N

## Outputs

- Per-kmer family probability F_kmer = n_kmer/N
- Per-kmer modified family probability F'_kmer
- Per-kmer overall score S_kmer bounded in (-1, 1)

## How to apply

First compute the family probability F_kmer as n_kmer/N, the fraction of sequences within a family that contain a given AAR-kmer, using reduced-alphabet kmer vectors consistent with Snekmer's encoding. Next compute the modified family probability F'_kmer by subtracting from F_kmer the summed fractional probabilities of the same kmer in every other family (G families total), so that kmers shared across families are penalized. Then form the overall score S_kmer by combining F'_kmer with the background kmer prevalence P_kmer across the dataset; scores fall in (-1, 1), where 1 means the kmer occurs in all in-group and no out-group sequences, -1 the reverse, and 0 denotes in-group/out-group agnosticism. Unit-test each formula against hand-computed fixtures covering edge cases (kmers unique to one family, kmers present in all families) and smoke-test end-to-end with the demo annotation file TIGRFAMs_annotation.ann and a small kmer count matrix.

## Related tools

- **Snekmer** (Source implementation providing the AAR kmer encoding, F_kmer/F'_kmer/S_kmer scoring definitions, and demo data (TIGRFAMs_annotation.ann); reimplement/reuse its scoring module in Python) — github.com/PNNL-CompBio/Snekmer
- **Python** (Language in which the scoring formulas are implemented and unit-tested against hand-computed fixtures (Python 3.11+ required by Snekmer)) — github.com/PNNL-CompBio/Snekmer

## Examples

```
snekmer apply --ann resources/demo_sequences/learn_apply_inputs/annotations/TIGRFAMs_annotation.ann
```

## Evaluation signals

- Every computed S_kmer value lies strictly within the open interval (-1, 1) across all test fixtures.
- Boundary semantics match documentation: S_kmer = 1 when a kmer occurs in all in-group and no out-group sequences; S_kmer = -1 for the reverse; S_kmer = 0 for in-group/out-group agnosticism.
- Kmers present in all families yield F'_kmer <= 0 after subtracting the summed fractional probabilities of the other families (shared kmers are penalized).
- Kmers unique to one family score maximally for that family, matching hand-computed fixture values for F_kmer, F'_kmer, and S_kmer.
- End-to-end smoke test with TIGRFAMs_annotation.ann parses family labels and a small kmer count matrix without input-schema errors.

## Limitations

- Snekmer's kmer-set representation uses an exact-match methodology that may be limited in sensitivity and coverage; amino acid recoding partially mitigates but does not eliminate this.
- Score = 0 indicates the kmer is not informative for distinguishing in-group from out-group sequences, so scores near zero should not be interpreted as confident either way.
- No changelog or discussion-section content is available in the source document, so version drift in the scoring implementation cannot be traced from the article alone.

## Evidence

- [methods] family probability F_kmer = n_kmer/N is the fraction of family members containing a kmer; a modified score F'_kmer subtracts the summed fractional probabilities of the other G families: [evidence span withheld — anchor status: not_found]
- [methods] the overall score S_kmer combines F'_kmer with background kmer prevalence P_kmer, yielding values in (-1, 1), where 1 means the kmer occurs in all in-group and no out-group sequences: [evidence span withheld — anchor status: not_found]
- [results] The main output is a CSV file at ``results/apply/snekmer_results.csv``: "The main output is a CSV file at ``results/apply/snekmer_results.csv``"
- [methods] A family-specific threshold used for prediction filtering: None, 'Median', 'Mean', '90th Percentile', etc.: "``threshold``    A family-specific threshold used for prediction filtering: None, 'Median', 'Mean', '90th Percentile', etc."
- [readme] Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach: "Snekmer is a software package designed to reduce the representation of protein sequences by combining amino acid reduction (AAR) with the kmer approach"
- [methods] --ann    resources/demo_sequences/learn_apply_inputs/annotations/TIGRFAMs_annotation.ann: "--ann    resources/demo_sequences/learn_apply_inputs/annotations/TIGRFAMs_annotation.ann"
