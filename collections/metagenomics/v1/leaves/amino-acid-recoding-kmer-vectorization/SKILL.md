---
name: amino-acid-recoding-kmer-vectorization
description: Use when protein FASTA sequences must be vectorized as reduced-alphabet
  AAR kmers, or when supervised family-scoring semantics (F_kmer, F'_kmer, S_kmer
  with values in (-1, 1)) must be reproduced or unit-tested in Python.
when_to_use_negative:
- The goal is unsupervised clustering of sequences (use Snekmer's cluster mode with
  agglomerative-jaccard or density-jaccard) rather than supervised family scoring
- The input is already a feature/embedding table and no AAR kmer vectorization is
  needed
- The sequences are not proteins (e.g. nucleotide FASTA) and no amino-acid recoding
  is intended
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Snekmer
  role: Python package providing AAR kmer vectorization (cluster, model, search, learn,
    apply, motif modes) whose supervised scoring semantics (F_kmer, F'_kmer, S_kmer)
    this skill reproduces
  repo: http://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Implementation and unit-test language for the scoring formulas and hand-computed
    fixtures
- name: Snakemake
  role: Workflow orchestrator through which Snekmer pipeline stages are executed
- name: bsf-jaccard-py
  role: Blazing Signature Filter wrapper computing Jaccard similarity on kmer sets,
    used as a fast alternative for kmer-set comparisons
  repo: https://github.com/PNNL-CompBio/bsf-jaccard-py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:89e48529cb8b654b40b1310d89ea6c637ab15f620a9bc26a4b96564ca800c933
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Snakemake
  - Snekmer
  - bsf-jaccard-py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# amino-acid-recoding-kmer-vectorization

## Summary

Implements Snekmer's AAR-kmer feature construction and supervised kmer scoring: proteins are recoded into reduced-alphabet amino-acid-recoded (AAR) kmer vectors, and family scores F_kmer = n_kmer/N, modified scores F'_kmer (penalizing kmers shared with other families), and overall scores S_kmer in (-1, 1) are computed.

## When to use

Use when protein FASTA sequences must be vectorized as reduced-alphabet AAR kmers, or when supervised family-scoring semantics (F_kmer, F'_kmer, S_kmer with values in (-1, 1)) must be reproduced or unit-tested in Python.

## When NOT to use

- The goal is unsupervised clustering of sequences (use Snekmer's cluster mode with agglomerative-jaccard or density-jaccard) rather than supervised family scoring
- The input is already a feature/embedding table and no AAR kmer vectorization is needed
- The sequences are not proteins (e.g. nucleotide FASTA) and no amino-acid recoding is intended

## Inputs

- Protein sequences (FASTA) for training families and query proteins
- AAR kmer count/feature matrix derived from recoded sequences
- Annotation file, e.g. TIGRFAMs_annotation.ann mapping sequences to families (200 TIGRFAM demo families)
- config.yaml with search params (k, alphabet recoding, min_rep/max_rep, selection)

## Outputs

- Per-kmer family probability scores F_kmer = n_kmer/N
- Modified family probability scores F'_kmer penalizing cross-family kmers
- Overall kmer scores S_kmer strictly within (-1, 1)
- results/apply/snekmer_results.csv with columns Sequence, Prediction, Score, delta, Confidence

## How to apply

First recode protein sequences into AAR kmer vectors using Snekmer's encoding (configure k and alphabet recoding in config.yaml; optionally apply min_rep/max_rep to discard rare k-mers). Then compute the family probability F_kmer = n_kmer/N, the fraction of family members containing a given AAR kmer, from the kmer count matrix. Compute the modified family probability F'_kmer by subtracting from F_kmer the summed fractional probabilities of the same kmer in every other family, so kmers shared across families are penalized. Combine F'_kmer with the background prevalence P_kmer of the kmer across the dataset to obtain the overall score S_kmer, which must fall strictly within (-1, 1): 1 means the kmer occurs in all in-group and no out-group sequences, -1 the reverse, and 0 indicates in-group/out-group agnosticism. Validate each formula with hand-computed fixtures covering edge cases (kmers unique to one family, kmers present in all families), and smoke-test end-to-end with the demo TIGRFAMs_annotation.ann file.

## Related tools

- **Snekmer** (Python package providing AAR kmer vectorization (cluster, model, search, learn, apply, motif modes) whose supervised scoring semantics (F_kmer, F'_kmer, S_kmer) this skill reproduces) — http://github.com/PNNL-CompBio/Snekmer
- **Python** (Implementation and unit-test language for the scoring formulas and hand-computed fixtures)
- **Snakemake** (Workflow orchestrator through which Snekmer pipeline stages are executed)
- **bsf-jaccard-py** (Blazing Signature Filter wrapper computing Jaccard similarity on kmer sets, used as a fast alternative for kmer-set comparisons) — https://github.com/PNNL-CompBio/bsf-jaccard-py

## Examples

```
snekmer apply --ann resources/demo_sequences/learn_apply_inputs/annotations/TIGRFAMs_annotation.ann
```

## Evaluation signals

- Every computed S_kmer value lies strictly within the open interval (-1, 1)
- Boundary semantics hold: a kmer in all in-group and no out-group sequences scores near 1, the reverse near -1, and a family-agnostic kmer scores near 0
- Hand-computed fixture tests pass for edge cases: kmers unique to one family and kmers present in all families (where F'_kmer should be heavily penalized)
- End-to-end smoke test parses the demo annotation file TIGRFAMs_annotation.ann and a small kmer count matrix without error
- Apply output snekmer_results.csv contains the expected schema: Sequence, Prediction, Score, delta, Confidence

## Limitations

- Kmer set representation is an exact-match methodology that may be limited in sensitivity and coverage; AAR recoding mitigates but does not remove this
- Scores depend on background prevalence P_kmer, so rare or ubiquitous kmers need explicit edge-case testing
- Requires Python 3.11+ (per README) and a C/C++ toolchain may be needed for some dependencies

## Evidence

- [methods] family probability F_kmer = n_kmer/N is the fraction of family members containing a kmer: [evidence span withheld — anchor status: not_found]
- [methods] the overall score S_kmer combines F'_kmer with background kmer prevalence P_kmer, yielding values in (-1, 1): [evidence span withheld — anchor status: not_found]
- [abstract] Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models: "Here, we describe Snekmer, a software tool for recoding proteins into AAR kmer vectors and performing either (i) construction of supervised classification models"
- [readme] Snekmer is a software package designed to reduce the representation of protein sequences: "Snekmer is a software package designed to reduce the representation of protein sequences"
- [methods] ``min_rep``               ``int`` or ``None``    Discard k-mers with fewer than this many occurrences across the input set: "``min_rep``               ``int`` or ``None``    Discard k-mers with fewer than this many occurrences across the input set"
